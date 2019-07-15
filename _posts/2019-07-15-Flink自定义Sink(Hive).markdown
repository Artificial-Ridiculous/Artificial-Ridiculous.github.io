---
layout: post
title:  "Flink自定义Sink(Hive)"
date:   2019-07-15 17:16:00 +0800
category: linux
---
***所有操作默认非root用户，本文中用 `lz` 用户。***

本文中我们需要使用到流计算引擎flink，以kafka作为source，以自定义的 Hive 作为sink，完成将kafka种实时产生的数据写入 Hive 数据库保存的任务。

假设在 Hive 的数据库test种有一张transactions表：

```text
transaction_id          int
card_number             int
terminal_id             int
transaction_time        timestamp
transaction_type        int
amount                  float
```

打开idea新建maven项目，在`pom.xml`中添加如下依赖：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>top.cocobolo</groupId>
    <artifactId>hive</artifactId>
    <version>1.0-SNAPSHOT</version>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>8</source>
                    <target>8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
    <dependencies>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.47</version>
        </dependency>

        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-java</artifactId>
            <version>1.8.0</version>
        </dependency>

        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-core</artifactId>
            <version>1.8.0</version>
            <!--            <scope>provided</scope>-->
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-java</artifactId>
            <version>1.8.0</version>
            <!--            <scope>provided</scope>-->
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-streaming-java_2.11</artifactId>
            <version>1.8.0</version>
            <!--            <scope>provided</scope>-->
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-clients_2.11</artifactId>
            <version>1.8.0</version>
            <!--            <scope>provided</scope>-->
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.apache.flink/flink-connector-kafka -->
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-connector-kafka_2.12</artifactId>
            <version>1.8.0</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.apache.kafka/kafka -->
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka_2.11</artifactId>
            <version>2.2.0</version>
        </dependency>

        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-streaming-java_2.11</artifactId>
            <version>1.8.0</version>
        </dependency>

        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.7</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.7.7</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.16</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.apache.bahir/flink-connector-redis -->
        <dependency>
            <groupId>org.apache.bahir</groupId>
            <artifactId>flink-connector-redis_2.11</artifactId>
            <version>1.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-common</artifactId>
            <version>2.6.5</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hive</groupId>
            <artifactId>hive-jdbc</artifactId>
            <version>1.2.2</version>
        </dependency>
    </dependencies>
</project>
```

我们需要定义一个Transaction类：

```java
package top.cocobolo;

import java.sql.Timestamp;

public class Transaction {
    private Integer transaction_id;
    private Integer card_number;
    private Integer terminal_id;
    private Timestamp transaction_time;
    private Integer transaction_type;
    private Float amount;

    public Transaction(Integer transaction_id, Integer card_number, Integer terminal_id, Timestamp transaction_time, Integer transaction_type, Float amount) {
        this.transaction_id = transaction_id;
        this.card_number = card_number;
        this.terminal_id = terminal_id;
        this.transaction_time = transaction_time;
        this.transaction_type = transaction_type;
        this.amount = amount;
    }

    @Override
    public String toString() {
        return "Transaction{" +
                "transaction_id=" + transaction_id +
                ", card_number=" + card_number +
                ", terminal_id=" + terminal_id +
                ", transaction_time=" + transaction_time +
                ", transaction_type=" + transaction_type +
                ", amount=" + amount +
                '}';
    }

    public Integer getTransaction_id() {
        return transaction_id;
    }

    public void setTransaction_id(Integer transaction_id) {
        this.transaction_id = transaction_id;
    }

    public Integer getCard_number() {
        return card_number;
    }

    public void setCard_number(Integer card_number) {
        this.card_number = card_number;
    }

    public Integer getTerminal_id() {
        return terminal_id;
    }

    public void setTerminal_id(Integer terminal_id) {
        this.terminal_id = terminal_id;
    }

    public Timestamp getTransaction_time() {
        return transaction_time;
    }

    public void setTransaction_time(Timestamp transaction_time) {
        this.transaction_time = transaction_time;
    }

    public Integer getTransaction_type() {
        return transaction_type;
    }

    public void setTransaction_type(Integer transaction_type) {
        this.transaction_type = transaction_type;
    }

    public Float getAmount() {
        return amount;
    }

    public void setAmount(Float amount) {
        this.amount = amount;
    }
}
```

我们还需要一个`KafkaProduce.java`来生向kafka的transaction topic里实时写入数据：

```java
package top.cocobolo;

import com.alibaba.fastjson.JSON;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;
import java.sql.Timestamp;
import java.util.Properties;
import java.util.Random;

/**
 * 向kafka中写数据
 * 可使用此main函数进行测试一下
 */

public class KafkaProduce {
    public static final String broker_list = "192.168.229.129:9092";
    public static final String topic = "transaction";  //kafka topic 需要和 flink 程序用同一个 topic

    public static void writeToKafka() throws InterruptedException {
        Properties props = new Properties();
        props.put("bootstrap.servers", broker_list);
        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        KafkaProducer producer = new KafkaProducer<String, String>(props);

        while(true) {
            Integer transaction_id = (int)(1+Math.random()*(1000000000-1+1));
            Integer card_number = (int)(1+Math.random()*(100000000-1+1));
            Integer terminal_id = (int)(1+Math.random()*(10000000-1+1));
            Timestamp transaction_time = new Timestamp(System.nanoTime());
            Integer transaction_type = (int)(1+Math.random()*(8-1+1));
            Float amount = new Random().nextFloat()*10000;

            Transaction transaction = new Transaction(transaction_id,card_number,terminal_id,transaction_time,transaction_type,amount);

            ProducerRecord record = new ProducerRecord<String, String>(topic, null, null, JSON.toJSONString(transaction));
            producer.send(record);
            String a = JSON.toJSONString(transaction);
            System.out.println("发送数据: " + a);
            Thread.sleep(5000);
            producer.flush();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        writeToKafka();
    }
}
```

接下来就是自定义`HiveSink.java`了：

```java
package top.cocobolo;

import java.sql.*;

import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.functions.sink.RichSinkFunction;
import java.sql.PreparedStatement;


public class HiveSink extends RichSinkFunction<Transaction> {
    private PreparedStatement state ;
    private Connection conn ;
    private String querySQL = "";
//    private String sql;

    @Override
    public void open(Configuration parameters) throws Exception {
        super.open(parameters);
        conn = getConnection();

        querySQL = "insert into transaction(transaction_id,card_number,terminal_id,transaction_time,transaction_type,amount) values(? ,?, ?, ?, ?, ?)";
        state = conn.prepareStatement(querySQL);

    }

    @Override
    public void close() throws Exception {
        super.close();
        if (state != null) {
            state.close();
        }
        if (conn != null) {
            conn.close();
        }

    }

    @Override
    public void invoke(Transaction value, Context context) throws Exception {
        state.setLong(1,value.getTransaction_id());
        state.setLong(2,value.getCard_number());
        state.setLong(3,value.getTerminal_id());
        state.setString(4,value.getTransaction_time().toString());
        state.setInt(5,value.getTransaction_type());
        state.setDouble(6,value.getAmount());

        state.executeUpdate();
    }

    private static Connection getConnection() {
        Connection conn = null;
        try {
            String jdbc = "org.apache.hive.jdbc.HiveDriver";
            String url = "jdbc:hive2://192.168.229.129:10000/test";
            String user = "lz";  // 重要！此处必须填入具有HDFS写入权限的用户名，比如hive文件夹的owner
            String password = "";
            Class.forName(jdbc);
            conn = DriverManager.getConnection(url, user, password);

        } catch (Exception e) {
            System.out.println(e.getMessage());
        }
        return conn;
    }


    public static void main(String[] args) {
        getConnection();
    }

}
```

最后写一个`Main.java`实现整个逻辑：

```java
package top.cocobolo;

import com.alibaba.fastjson.JSON;
import org.apache.flink.api.common.serialization.SimpleStringSchema;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer;
import java.util.Properties;

public class Main {
    public static void main(String[] args) throws Exception {
        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        Properties props = new Properties();
        props.put("bootstrap.servers", "192.168.229.129:9092");
        props.put("zookeeper.connect", "192.168.229.129:2181");
        props.put("group.id", "metric-group");
        props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        props.put("auto.offset.reset", "latest");

        env.addSource(
            new FlinkKafkaConsumer<>(
                "transaction"
                ,new SimpleStringSchema()
                , props
            )
                .setStartFromLatest()
        )
            .setParallelism(1)
            .map(string -> JSON.parseObject(string, Transaction.class))
            .addSink(new HiveSink()); //数据 sink 到 mysql

        env.execute("Flink add sink");
    }

}
```

先运行`KafkaProduce`的`main()`方法，再运行`Main.java`的`main()`方法，实时刷新 Hive 数据库，可以看到每过5s就有一条新的交易记录被写入。
