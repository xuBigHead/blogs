---
layout: post
title: JDK LocalDateTime.md
categories: [Java]
description: Java
keywords: Java
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---
# Instant

## 概述

## 源码解析

### 方法定义



## 实践应用



```java
@Test
public void Instant(){
    // 获取当前时间戳
    Instant timestamp = Instant.now();
}
```



# Clock

## 概述

> @since 8



Clock类提供了访问当前日期和时间的方法，Clock类是时区敏感的，可以用来取代`System.currentTimeMillis()`来获取当前的微秒数。



## 源码解析

### 方法定义



## 实践应用

```java
@Test
public void testClock() {
    // 根据系统时钟区域返回时间
    Clock clock = Clock.systemDefaultZone();
    System.out.println(clock.millis()); // 1741933682593
    System.out.println(System.currentTimeMillis()); // 1741933682593
}
```





# LocalDate

## 概述

LocalDate表示了一个确切的日期，比如2014-03-11，该对象值是不可变的。提供了获取当前日期和不同日期之间比较等方法。



## 源码解析

### 方法定义



## 实践应用

```java
@Test
public void localDate(){
    // 获取当前日期
    LocalDate now = LocalDate.now();

    // 获取当前年月日
    int year = now.getYear();
    int month = now.getMonthValue();
    int day = now.getDayOfMonth();

    // 创建指定日期
    LocalDate today = LocalDate.of(2022, 10, 20);

    // 判断日期是否相等
    boolean isSameDay = today.equals(now);

    // 获取月日信息，如生日、法定假日等
    MonthDay monthDay = MonthDay.of(now.getMonth(), now.getDayOfMonth());
    MonthDay anotherMonthDay = MonthDay.from(today);
    boolean isSameMonthDay = monthDay.equals(anotherMonthDay);

    // 获取下一周
    LocalDate nextWeek = now.plus(1, ChronoUnit.WEEKS);

    // 获取上一年
    LocalDate previousYear = now.minus(1, ChronoUnit.YEARS);

    // 比较日期先后，有 isBefore 和 isAfter 方法
    boolean beforeTargetDay = now.isBefore(today);

    // 判断是否是闰年
    boolean isLeapYear = now.isLeapYear();

    LocalDate formattedDate = LocalDate.parse("20221020", DateTimeFormatter.BASIC_ISO_DATE);
}
```



日期格式转字符串

```java
@Test
public void testFormatToString() {
    LocalDate date = LocalDate.now();
    System.out.println(String.format("date format : %s", date));

    //format HH:mm:ss
    LocalTime time = LocalTime.now().withNano(0);
    System.out.println(String.format("time format : %s", time));

    //format yyyy-MM-dd HH:mm:ss
    LocalDateTime dateTime = LocalDateTime.now();
    DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
    String dateTimeStr = dateTime.format(dateTimeFormatter);
    System.out.println(String.format("dateTime format : %s", dateTimeStr));
}
```



字符串转换日期格式

```java
@Test
public void testParseFromString() {
    LocalDate date = LocalDate.of(2021, 1, 26);
    LocalDate.parse("2021-01-26");

    LocalDateTime dateTime = LocalDateTime.of(2021, 1, 26, 12, 12, 22);
    LocalDateTime.parse("2021-01-26 12:12:22");

    LocalTime time = LocalTime.of(12, 12, 22);
    LocalTime.parse("12:12:22");
}
```



下面仅以一周后日期为例，其他单位（年、月、日、1/2 日、时等等）大同小异。

```java
@Test
public void testCalculateDateByLocalDate() {
    //一周后的日期
    LocalDate localDate = LocalDate.now();
    //方法1
    LocalDate after = localDate.plus(1, ChronoUnit.WEEKS);
    //方法2
    LocalDate after2 = localDate.plusWeeks(1);
    System.out.println("一周后日期：" + after);

    //算两个日期间隔多少天，计算间隔多少年，多少月
    LocalDate date1 = LocalDate.parse("2021-02-26");
    LocalDate date2 = LocalDate.parse("2021-12-23");
    Period period = Period.between(date1, date2);
    System.out.println("date1 到 date2 相隔：" + period.getYears() + "年" + period.getMonths() + "月" + period.getDays() + "天");
    //打印结果是 “date1 到 date2 相隔：0年9月27天”，这里period.getDays()得到的天是抛去年月以外的天数，并不是总天数
    //如果要获取纯粹的总天数应该用下面的方法
    long day = date2.toEpochDay() - date1.toEpochDay();
    System.out.println(date1 + "和" + date2 + "相差" + day + "天"); // 2021-02-26和2021-12-23相差300天
}
```



获取特定一个日期，比如获取本月最后一天，第一天。

```java
@Test
public void testGetTargetDayByLocalDate() {
    LocalDate today = LocalDate.now();
    // 获取当前月第一天：
    LocalDate firstDayOfThisMonth = today.with(TemporalAdjusters.firstDayOfMonth());
    // 取本月最后一天
    LocalDate lastDayOfThisMonth = today.with(TemporalAdjusters.lastDayOfMonth());
    // 取下一天：
    LocalDate nextDay = lastDayOfThisMonth.plusDays(1);
    // 当年最后一天
    LocalDate lastDay = today.with(TemporalAdjusters.lastDayOfYear());
    // 2021年最后一个周日，如果用Calendar是不得烦死。
    LocalDate lastMondayOf2021 = LocalDate.parse("2021-12-31").with(TemporalAdjusters.lastInMonth(DayOfWeek.SUNDAY));
}
```



# LocalTime

## 概述

LocalTime定义了一个没有时区信息的时间，例如晚上10点或者17:30:15。提供了获取当前时间和不同时间之间比较等方法。



## 源码解析

### 方法定义



## 实践应用

```java
@Test
public void localTime(){
    // 获取当前时间
    LocalTime time = LocalTime.now();

    // 增加一个小时
    LocalTime oneHourPlus = time.plusHours(1);
}
```



# LocalDateTime

## 概述

LocalDateTime同时表示了时间和日期，相当于LocalTime和LocalDate合并到一个对象上了。LocalDateTime和LocalTime还有LocalDate一样，都是不可变的。LocalDateTime提供了一些能访问具体字段的方法。



## 源码解析

### 方法定义



## 实践应用



# DateTimeFormatter

## 概述

> @since 8



Java 8新增了日期格式化类DateTimeFormatter来代替SimpleDateFormat。DateTimeFormatter是不可变的，所以是线程安全的。



## 源码解析

### 方法定义



## 实践应用

# YearMonth

## 概述

## 源码解析

### 方法定义



## 实践应用

```java
@Test
public void YearMonth(){
    // 表示固定日期，如信用卡到期日等
    YearMonth yearMonth = YearMonth.now();
    YearMonth octoberMonth = YearMonth.of(2022, Month.OCTOBER);
}
```





# Period

## 概述

## 源码解析

### 方法定义



## 实践应用

```java
@Test
public void period(){
    // 获取时间范围间隔
    LocalDate today = LocalDate.now();
    LocalDate birthday = LocalDate.of(1994, 9, 1);
    Period periodFromBirthday = Period.between(today, birthday);
}
```





时区代表了地球上某个区域内普遍使用的标准时间。每个时区都有一个代号，格式通常由区域/城市构成（Asia/Tokyo），在加上与格林威治或 UTC的时差。例如：东京的时差是+09:00。



# ZonedDateTime

## 概述

ZonedDateTime是带时区的时间，可以理解为LocalDatetime + ZoneId。



## 源码解析

### 方法定义



## 实践应用

```java
@Test
public void testZoneDateTime() {
    //当前时区时间
    ZonedDateTime zonedDateTime = ZonedDateTime.now();
    System.out.println("当前时区时间: " + zonedDateTime);

    //东京时间
    ZoneId zoneId = ZoneId.of(ZoneId.SHORT_IDS.get("JST"));
    ZonedDateTime tokyoTime = zonedDateTime.withZoneSameInstant(zoneId);
    System.out.println("东京时间: " + tokyoTime);

    // ZonedDateTime 转 LocalDateTime
    LocalDateTime localDateTime = tokyoTime.toLocalDateTime();
    System.out.println("东京时间转当地时间: " + localDateTime);

    //LocalDateTime 转 ZonedDateTime
    ZonedDateTime localZoned = localDateTime.atZone(ZoneId.systemDefault());
    System.out.println("本地时区时间: " + localZoned);
}
```



# ZoneOffset

## 概述

## 源码解析

### 方法定义



## 实践应用



# ZoneId

## 概述

> @since 8

时区使用ZoneId来表示，可以很方便的使用静态方法of来获取。抽象类`ZoneId`表示一个区域标识符，它有一个名为`getAvailableZoneIds`的静态方法，返回所有区域标识符。



## 源码解析

### 方法定义



## 实践应用

```java
@Test
public void test() {
    System.out.println(ZoneId.getAvailableZoneIds());

    ZoneId zone1 = ZoneId.of("Europe/Berlin");
    ZoneId zone2 = ZoneId.of("Brazil/East");
    System.out.println(zone1.getRules());// ZoneRules[currentStandardOffset=+01:00]
    System.out.println(zone2.getRules());// ZoneRules[currentStandardOffset=-03:00]
}
```
