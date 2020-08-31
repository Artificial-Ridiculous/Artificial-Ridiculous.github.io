---
layout: post
title:  "FastJSON vs GSON"
date:   2020-08-31 14:48:00 +0800
category: java
---

本文主要对比对于相同的操作，FastJSON和GSON的语法的区别。

`​msg.getKafkaMessage().toJson()`会得到上游Kafka发来的信息并转换成Json字符串格式。

调用示例：

```java
// FastJSON
JSONObject json = JSONObject.parseObject(msg.getKafkaMessage().toJson()).getJSONObject("fields");
String datekey = json.getString("datekey");
int is_high_star = json.getIntValue("datekey");  // 解析失败返回默认值，此处为0
int is_high_star = json.getInteger("datekey");  // 解析失败返回null
json.put("hotel_star_parent_rank",hotel_star_parent_rank);
String data = json.toJSONString();

//Gson
JsonObject json = new Gson().fromJson(msg.getKafkaMessage().toJson(), JsonObject.class).getAsJsonObject("fields");
String datekey = json.get("datekey").getAsString();
int is_high_star = json.get("is_high_star").getAsInt();  //返回int，相当于第4行getIntValue()
json.addProperty("hotel_star_parent_rank",hotel_star_parent_rank);
String data = json.toString();  // JsonElement的方法：Returns a String representation of this element.
```

分析：

FastJSON的`JSONObject`类有静态方法`parseObject(String text)`，该方法接受一个Json字符串，并解析成JsonObject：

![FastJSON_vs_GSON-1.png](/PNG/FastJSON_vs_GSON-1.png)

得到JsonObject后，再调用方法​`getJSONObject(String key)`方法得到其中键值为key的value的JsonObject：

![FastJSON_vs_GSON-2.png](/PNG/FastJSON_vs_GSON-2.png)

---

GSON的对象可以调用成员方法​`fromJson(String json, Class<T> classOfT)`，从Json字符串json中解析出一个JsonObject对象：

![FastJSON_vs_GSON-3.png](/PNG/FastJSON_vs_GSON-3.png)

同样地，得到JsonObject对象后，再调用方法`​getAsJsonObject(String menberName)`得到其中键值为memberName的value的JsonObject：

![FastJSON_vs_GSON-4.png](/PNG/FastJSON_vs_GSON-4.png)