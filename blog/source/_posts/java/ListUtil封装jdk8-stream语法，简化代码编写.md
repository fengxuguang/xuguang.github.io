---
title: ListUtil封装jdk8-stream语法，简化代码编写
tags:
  - jdk8
  - stream
abbrlink: 508b40e4
date: 2024-03-20 16:08:40
---

## 背景

​	由于在日常开发中，会经常使用到`JDK8`中的`Stream`语法糖，然而在使用过程中，发现存在很多冗余的方法，便想着封装个工具类，简化开发过程中的使用。

## 工具类 ListUtil

```java
import org.apache.commons.collections.CollectionUtils;
import org.apache.commons.lang3.StringUtils;

import java.util.ArrayList;
import java.util.Collections;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Optional;
import java.util.function.BinaryOperator;
import java.util.function.Function;
import java.util.function.Predicate;
import java.util.stream.Collectors;

/**
 * JDK8-Stream 工具类封装
 * Created by fengxuguang on 2024/3/20 14:29
 */
public class ListUtil {

    /**
     * 过滤出有效 属性, 并去重
     * @param list 集合
     * @param mapper 处理属性
     * @return List<String> 返回结果
     * @param <T> 形参集合内实体类型
     */
    public static <T>List<String> filterValidValue(List<T> list, Function<? super T, String> mapper) {
        if (CollectionUtils.isEmpty(list)) {
            return Collections.emptyList();
        }

        return list.stream()
                .map(mapper)
                .filter(StringUtils::isNotBlank)
                .distinct()
                .collect(Collectors.toList());
    }

    /**
     * 过滤多字段有效值, 并去重
     * @param list 集合
     * @param mappers 属性, 支持多属性
     * @return List<String> 返回结果
     * @param <T> 形参集合内实体类型
     */
    public static <T> List<String> filterValidValueForFields(List<T> list, Function<? super T, String> ...mappers) {
        if (CollectionUtils.isEmpty(list)) {
            return Collections.emptyList();
        }

        List<String> result = new ArrayList<>();
        for (Function<? super T, String> mapper : mappers) {
            result.addAll(list.stream()
                    .map(mapper)
                    .filter(StringUtils::isNotBlank)
                    .distinct()
                    .collect(Collectors.toList()));
        }
        return result.stream()
                .distinct()
                .collect(Collectors.toList());
    }

    /**
     * 将 List 转为 Map, 如果有相同元素, 去第一个
     * @param list list集合
     * @param keyMapper 作为属性列
     * @return Map<String, T> map 集合
     * @param <T> 形参集合内实体类型
     */
    public static <T> Map<String, T> toMap(List<T> list, Function<? super T, String> keyMapper) {
        BinaryOperator<T> miss = (k1, k2) -> k1;
        return list.stream()
                .collect(Collectors.toMap(keyMapper, Function.identity(), miss));
    }

    /**
     * 过滤数据
     * @param list list 集合
     * @param predicate 断言
     * @return List<T> 集合
     * @param <T> 形参集合内实体类型
     */
    public static <T> List<T> filter(List<T> list, Predicate<? super T> predicate) {
        if (CollectionUtils.isEmpty(list)) {
            return Collections.emptyList();
        }

        return list.stream()
                .filter(predicate)
                .collect(Collectors.toList());
    }

    /**
     * 获取 List 集合内实体中某属性的值
     * @param list List集合
     * @param mapper 属性
     * @return List<T> 集合
     * @param <R> 返回集合内实体的类型
     * @param <T> 形参集合内实体类型
     */
    public static <R, T> List<R> map(List<T> list, Function<? super T, R> mapper) {
        if (CollectionUtils.isEmpty(list)) {
            return Collections.emptyList();
        }

        return list.stream()
                .map(mapper)
                .collect(Collectors.toList());
    }

    /**
     * 将 List 集合按照集合内实体某属性进行分组
     * @param list List集合
     * @param mapper 属性
     * @return Map<R, List<T> map 集合
     * @param <R> 分组属性类型
     * @param <T> 形参集合内实体类型
     */
    public static <R, T> Map<R, List<T>> groupBy(List<T> list, Function<? super T, R> mapper) {
        if (CollectionUtils.isEmpty(list)) {
            return Collections.emptyMap();
        }

        return list.stream()
                .collect(Collectors.groupingBy(mapper));
    }

    /**
     * 将 List 集合按照实体属性 keyMapper 作为 key, valueMapper 作为 value 封装成 map 集合
     * @param list List集合
     * @param keyMapper 实体属性, 作为 map 集合的 key
     * @param valueMapper 实体属性, 作为 map 集合的 value
     * @return Map<K, List<R>> map 集合
     * @param <K> 返回 map 集合的 key 类型
     * @param <T> 形参 List 集合类型
     * @param <R> 返回 map 集合的 value 类型
     */
    public static <K, T, R> Map<K, List<R>> groupBy(List<T> list,
                                                    Function<? super T, K> keyMapper,
                                                    Function<? super T, R> valueMapper) {
        if (CollectionUtils.isEmpty(list)) {
            return Collections.emptyMap();
        }

        Map<K, List<R>> result = new HashMap<>();
        groupBy(list, keyMapper).forEach((k, v) -> result.put(k, map(v, valueMapper)));

        return result;
    }

    /**
     * 获取 List 集合过滤之后的一个数据
     * @param list List 集合
     * @param predicate 断言, 过滤条件
     * @return Optional<T>
     * @param <T> 形参集合内实体类型
     */
    public static <T> Optional<T> findOne(List<T> list, Predicate<? super T> predicate) {
        return list.stream()
                .filter(predicate)
                .findFirst();
    }

    /**
     * 获取 List 集合过滤后的数量
     * @param list List 集合
     * @param predicate 断言, 过滤条件
     * @return long 数量
     * @param <T> 形参集合内实体类型
     */
    public static <T> long count(List<T> list, Predicate<? super T> predicate) {
        if (CollectionUtils.isEmpty(list)) {
            return 0L;
        }

        return list.stream()
                .filter(predicate)
                .count();
    }

}
```

