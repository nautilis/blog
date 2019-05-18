---
layout: post
title: java8
date: 2019-05-18 12:31:24
categories: 工具&技巧 
tags: java
---

### List to Map
```java
    @Data
    @AllArgsConstructor
    @ToString
    class People {
        private String name;
        private int age;
        private String lastName;
    }

    @Test
    public void testStream() {
        List<People> list = new ArrayList<>();
        list.add(new People("nautilis", 25, "zheng"));
        list.add(new People("nautilis", 25, "zheng"));
        list.add(new People("bevan", 25, "zheng"));
        list.add(new People("ning", 24, "li"));
        //重复key只保留一个， 并用linkedHashMap保证顺序
        Map<String, People> map = list.stream().sorted(Comparator.comparing(People::getAge).reversed())
                .collect(Collectors.toMap(People::getName, Function.identity(),
                        (oldv, newV) -> newV, LinkedHashMap::new ));
        System.out.printf("重复key只保留一个， 并用linkedHashMap保证顺序--->");
        map.forEach((k, v)  -> System.out.println(k + ":" + v));

        //姓氏#=>人列表
        Map<String,List<People>>  lastPeopleMap = list.stream()
                .collect(Collectors.groupingBy(People::getLastName));
        System.out.println("姓氏#=>人列表 --->");
        lastPeopleMap.forEach((k, v) -> System.out.println(k + " : " + v));

        //姓氏#=> 人名列表
        Map<String, List<String>> lastnameNamelistMap = list.stream()
                .collect(Collectors.groupingBy(People::getLastName, Collectors.mapping(People::getName, Collectors.toList())));
        System.out.println("姓氏#=> 人名列表--->");
        lastnameNamelistMap.forEach((k, v) -> System.out.println(k + " : " + v));

        // 姓氏#=>人名集合
        Map<String, Set<String>> lastnameNameSet = list.stream()
                .collect(Collectors.groupingBy(People::getLastName, Collectors.mapping(People::getName, Collectors.toSet())));
        System.out.println("姓氏#=>人名集合--->");
        lastnameNameSet.forEach((k, v) -> System.out.println(k + " : " + v));
    }
```
