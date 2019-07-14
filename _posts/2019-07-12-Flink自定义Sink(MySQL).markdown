---
layout: post
title:  "Flink自定义Sink(MySQL)"
date:   2019-07-12 14:09:00 +0800
category: linux
---
***所有操作默认非root用户，本文中用 `lz` 用户。***

本文中我们需要使用到流计算引擎flink，以kafka作为source，以自定义的MySQL作为sink，完成将kafka种实时产生的数据写入MySQL数据库保存的任务。

假设在MySQL的数据库test种有一张transactions表：

```text
+-----------------+------------+------+-----+---------+-------+
| Field           | Type       | Null | Key | Default | Extra |
+-----------------+------------+------+-----+---------+-------+
| TransactionID   | bigint(20) | NO   | PRI | NULL    |       |
| CardNumber      | bigint(20) | YES  |     | NULL    |       |
| TerminalID      | int(11)    | YES  |     | NULL    |       |
| TransactionDate | date       | YES  |     | NULL    |       |
| TransactionTime | time       | YES  |     | NULL    |       |
| TransactionType | tinyint(4) | YES  |     | NULL    |       |
| Amount          | float      | YES  |     | NULL    |       |
+-----------------+------------+------+-----+---------+-------+
```

打开idea新建maven项目，在`pom.xml`中添加如下依赖：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>top.cocobolo</groupId>
    <artifactId>MysqlSink</artifactId>
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

    </dependencies>
</project>
```

我们需要定义一个Transaction类：

```java
package top.cocobolo;

import java.sql.Date;
import java.sql.Time;

public class Transaction {
    private long TransactionID;
    private long CardNumber;
    private long TerminalID;
    private Date TransactionDate;
    private Time TransactionTime;
    private int TransactionType;
    private Double Amount;

    public Transaction(long transactionID, long cardNumber, long terminalID, Date transactionDate, Time transactionTime, int transactionType, Double amount) {
        TransactionID = transactionID;
        CardNumber = cardNumber;
        TerminalID = terminalID;
        TransactionDate = transactionDate;
        TransactionTime = transactionTime;
        TransactionType = transactionType;
        Amount = amount;
    }

    @Override
    public String toString() {
        return "Transaction{" +
                "TransactionID=" + TransactionID +
                ", CardNumber=" + CardNumber +
                ", TerminalID=" + TerminalID +
                ", TransactionDate=" + TransactionDate +
                ", TransactionTime=" + TransactionTime +
                ", TransactionType=" + TransactionType +
                ", Amount=" + Amount +
                '}';
    }

    public long getTransactionID() {
        return TransactionID;
    }


    public long getCardNumber() {
        return CardNumber;
    }

    public long getTerminalID() {
        return TerminalID;
    }

    public Date getTransactionDate() {
        return TransactionDate;
    }

    public Time getTransactionTime() {
        return TransactionTime;
    }

    public int getTransactionType() {
        return TransactionType;
    }

    public Double getAmount() {
        return Amount;
    }

    public void setTransactionID(long transactionID) {
        TransactionID = transactionID;
    }

    public void setCardNumber(long cardNumber) {
        CardNumber = cardNumber;
    }

    public void setTerminalID(long terminalID) {
        TerminalID = terminalID;
    }

    public void setTransactionDate(Date transactionDate) {
        TransactionDate = transactionDate;
    }

    public void setTransactionTime(Time transactionTime) {
        TransactionTime = transactionTime;
    }

    public void setTransactionType(int transactionType) {
        TransactionType = transactionType;
    }

    public void setAmount(Double amount) {
        Amount = amount;
    }

}

```

我们还需要一个`KafkaProduce.java`来生向kafka的transaction topic里实时写入数据：

```java
package top.cocobolo;

import com.alibaba.fastjson.JSON;
import org.apache.kafka.clients.producer.ProducerRecord;
import java.sql.Date;
import java.sql.Time;
import java.util.Properties;
import java.util.Random;

import java.sql.Date;
import java.sql.Time;
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
        org.apache.kafka.clients.producer.KafkaProducer producer = new org.apache.kafka.clients.producer.KafkaProducer<String, String>(props);

        while(true) {
            int rand8 = (int)(1+Math.random()*(100000000-1+1));
            int rand7 = (int)(1+Math.random()*(100000000-1+1));
            int rand6 = (int)(1+Math.random()*(100000000-1+1));
            Date currentDate = new Date(System.currentTimeMillis());
            System.out.println("currentDate = "+ currentDate);
            Time currentTime = new Time(System.currentTimeMillis());
            System.out.println("currentTime = "+ currentTime);
            int rand1 = (int)(1+Math.random()*(10-1+1));
            double generatorDouble = new Random().nextDouble()*1000;

            Transaction transaction = new Transaction(rand8,rand7,rand6,currentDate,currentTime,rand1,generatorDouble);

            ProducerRecord record = new ProducerRecord<String, String>(topic, null, null, JSON.toJSONString(transaction));
            producer.send(record);
            String a = JSON.toJSONString(transaction);
            System.out.println("发送数据: " + a);
            Thread.sleep(5000);
            producer.flush();
        }
//        producer.flush();
    }

    public static void main(String[] args) throws InterruptedException {
        writeToKafka();
    }
}
```

接下来就是自定义`MysqlSink.java`了：

```java
package top.cocobolo;

import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.functions.sink.RichSinkFunction;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;


public class MysqlSink extends RichSinkFunction<Transaction>{
    private PreparedStatement state ;
    private Connection conn ;

    @Override
    public void open(Configuration parameters) throws Exception {
        super.open(parameters);
        conn = getConnection();
        //System.out.println("conn=" + conn);
        String sql = "insert into transactions(transactionID,cardNumber,terminalID,transactionDate,transactionTime,transactionType,amount) values(?, ?, ?, ?, ?, ?, ?);";
        state = this.conn.prepareStatement(sql);
        //System.out.println("state=" + state);
    }

    @Override
    public void close() throws Exception {
        super.close();
        if (conn != null) {
            conn.close();
        }
        if (state != null) {
            state.close();
        }
    }

    @Override
    public void invoke(Transaction value, Context context) throws Exception {
        state.setLong(1,value.getTransactionID());
        state.setLong(2,value.getCardNumber());
        state.setLong(3,value.getTerminalID());
        state.setDate(4,value.getTransactionDate());
        state.setTime(5,value.getTransactionTime());
        state.setInt(6,value.getTransactionType());
        state.setDouble(7,value.getAmount());
        state.executeUpdate();
    }

    private static Connection getConnection() {
        Connection conn = null;
        try {
            String jdbc = "com.mysql.jdbc.Driver";
            Class.forName(jdbc);
            String url = "jdbc:mysql://router.cocobolo.top:33306/test";
            String user = "root";
            String password = "root";
            conn = DriverManager.getConnection(url +
                "?useUnicode=true" +
                "&useSSL=false" +
                "&characterEncoding=UTF-8" +
                "&serverTimezone=UTC",
                user,
                password);
        } catch (Exception e) {
            System.out.println(e.getMessage());
        }
        return conn;
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


public class MysqlSink extends RichSinkFunction<Transaction>{
    private PreparedStatement state ;
    private Connection conn ;

    @Override
    public void open(Configuration parameters) throws Exception {
        super.open(parameters);
        conn = getConnection();
        //System.out.println("conn=" + conn);
        String sql = "insert into transactions(transactionID,cardNumber,terminalID,transactionDate,transactionTime,transactionType,amount) values(?, ?, ?, ?, ?, ?, ?);";
        state = this.conn.prepareStatement(sql);
        //System.out.println("state=" + state);
    }

    @Override
    public void close() throws Exception {
        super.close();
        if (conn != null) {
            conn.close();
        }
        if (state != null) {
            state.close();
        }
    }

    @Override
    public void invoke(Transaction value, Context context) throws Exception {
        state.setLong(1,value.getTransactionID());
        state.setLong(2,value.getCardNumber());
        state.setLong(3,value.getTerminalID());
        state.setDate(4,value.getTransactionDate());
        state.setTime(5,value.getTransactionTime());
        state.setInt(6,value.getTransactionType());
        state.setDouble(7,value.getAmount());
        state.executeUpdate();
    }

    private static Connection getConnection() {
        Connection conn = null;
        try {
            String jdbc = "com.mysql.jdbc.Driver";
            Class.forName(jdbc);
            String url = "jdbc:mysql://{mysql server地址}:3306/test";
            String user = "root";
            String password = "root";
            conn = DriverManager.getConnection(url +
                "?useUnicode=true" +
                "&useSSL=false" +
                "&characterEncoding=UTF-8" +
                "&serverTimezone=UTC",
                user,
                password);
        } catch (Exception e) {
            System.out.println(e.getMessage());
        }
        return conn;
    }

}

```

先运行`KafkaProduce`的`main()`方法，再运行`Main.java`的`main()`方法，实时刷新MySQL数据库，可以看到每过5s就有一条新的交易记录被写入。
