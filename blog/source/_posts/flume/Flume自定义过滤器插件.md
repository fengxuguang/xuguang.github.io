---
title: Flume自定义过滤器插件
date: 2024-06-18 15:08:39
tags:
  - flume
categories:
  - flume
---

Flume 是一种可扩展的、可靠的分布式日志收集和聚合系统。它使用了拦截器来对数据流进行处理和过滤，以满足不同的需求。Flume 提供了很多内置的拦截器，但我们也可以通过自定义拦截器来实现自定义的数据过滤和处理。



## 一、引入依赖

新建一个工程，引入如下依赖：

```java
<dependency>
    <groupId>org.apache.flume</groupId>
    <artifactId>flume-ng-core</artifactId>
    <version>1.11.0</version>
</dependency>
```

## 二、自定义拦截器类

```java
package com.bda.dcp.flume.interceptor;

import com.google.common.base.Splitter;
import com.google.gson.Gson;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.compress.utils.Lists;
import org.apache.commons.lang.StringUtils;
import org.apache.flume.Context;
import org.apache.flume.Event;
import org.apache.flume.interceptor.Interceptor;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 *
 * Created by fengxuguang on 2024/6/14 15:32
 */
@Slf4j
public class JsonInterceptor implements Interceptor {

    private String jsonTopic;

    public JsonInterceptor(String jsonTopic) {
        this.jsonTopic = jsonTopic;
        System.out.println("======> keyword = " + jsonTopic);
    }

    @Override
    public void initialize() {

    }

    @Override
    public Event intercept(Event event) {
        try {
            String body = new String(event.getBody());

            Gson gson = new Gson();
            Map<String, String> bodyJson = gson.fromJson(body, Map.class);
            log.info("=====> body json: {}", bodyJson);

            // 根据逗号分割配置的字符串，转换成 List<String> 集合
            List<String> columns = new ArrayList<>();
            if (StringUtils.isNotBlank(jsonTopic)) {
                Iterable<String> split = Splitter.on(",")
                        .omitEmptyStrings()
                        .trimResults()
                        .split(jsonTopic);
                split.forEach(s -> columns.add(s));
            }
            Map<String, String> result = new HashMap<>();
            for (String key : bodyJson.keySet()) {
                // 过滤出事件 body 内的数据在配置项中指定的字段
                if (columns.contains(key)) {
                    result.put(key, bodyJson.get(key));
                }
            }

            event.setBody(gson.toJson(result).getBytes());
            return event;
        } catch (Exception e) {
            log.error(e.getMessage(), e);
            return null;
        }
    }

    @Override
    public List<Event> intercept(List<Event> list) {
        List<Event> resultList = Lists.newArrayList();
        for (Event event : list) {
            Event result = intercept(event);
            if (result != null) {
                resultList.add(result);
            }
        }
        return resultList;
    }

    @Override
    public void close() {

    }

    public static class Builder implements Interceptor.Builder {

        private String jsonTopic;

        @Override
        public Interceptor build() {
            return new JsonInterceptor(jsonTopic);
        }

        @Override
        public void configure(Context context) {
            jsonTopic = context.getString("jsonTopic");
        }
    }

}
```

## 三、打包

mvn package 打包成 jar 包后放入 flume/lib/ 目录

## 四、配置

```properties
a1.sources = r1
a1.sinks = k1
a1.channels = c1

a1.sources.r1.type = org.apache.flume.source.kafka.KafkaSource
a1.sources.r1.channels = c1
a1.sources.r1.batchSize = 3
a1.sources.r1.batchDurationMillis = 2000
a1.sources.r1.kafka.bootstrap.servers = 192.162.11.25:9092
a1.sources.r1.kafka.topics = flume-source-kafka
a1.sources.r1.interceptors = i1 i2
a1.sources.r1.interceptors.i1.type = static
a1.sources.r1.interceptors.i1.key = topic
a1.sources.r1.interceptors.i1.preserveExisting = false
a1.sources.r1.interceptors.i1.value = flume-collect-kafka
# 配置自定义拦截器，类型必须是: 类全名$内部类名
a1.sources.r1.interceptors.i2.type = com.bda.dcp.flume.interceptor.JsonInterceptor$Builder
# 配置要过滤的字段
a1.sources.r1.interceptors.i2.jsonTopic=name,age

a1.sinks.k1.channel = c1
a1.sinks.k1.type = org.apache.flume.sink.kafka.KafkaSink
a1.sinks.k1.kafka.flumeBatchSize = 3
a1.sinks.k1.kafka.bootstrap.servers = 192.162.11.191:9092
a1.sinks.k1.kafka.topic = flume-collect-kafka
a1.sinks.k1.kafka.producer.acks = 1
a1.sinks.k1.kafka.producer.linger.ms = 1
a1.sinks.k1.kafka.producer.compression.type = snappy

a1.channels.c1.type = org.apache.flume.channel.kafka.KafkaChannel
a1.channels.c1.kafka.bootstrap.servers = 192.162.11.25:9092
a1.channels.c1.kafka.topic = flume-channel
a1.channels.c1.kafka.consumer.auto.offset.reset = latest
```

