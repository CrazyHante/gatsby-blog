---
title: LocalDate,Date,String互转
subTitle: java8新时间类
category: "工具类"
cover: 1.jpg
---

```java
1.Date转LocalDate
    //Date日期
    Date date = new Date();
    //设置时区
    Instant instant = date.toInstant();
    ZoneId zoneId = ZoneId.systemDefault();
    // atZone()方法返回在指定时区从此Instant生成的ZonedDateTime。
    LocalDate localDate = instant.atZone(zoneId).toLocalDate();
    
    System.out.println("Date = " + date);
    System.out.println("LocalDate = " + localDate);
    
2.LocalDate转Date
    //LocalDate日期
    LocalDate localDate = LocalDate.now();
    
    ZoneId zoneId = ZoneId.systemDefault();
    ZonedDateTime zdt = localDate.atStartOfDay(zoneId);
    Date date = Date.from(zdt.toInstant());

    System.out.println("LocalDate = " + localDate);
    System.out.println("Date = " + date);
    
3.String转LocalDate
    //日期
    String dateStr1 = "2018-01-01";
    String dateStr2 = "2018/01/02";
    String dateStr3 = "2017-09-30 10:40:06"
    DateTimeFormatter df1 = DateTimeFormatter.ofPattern("yyyy-MM-dd");
    DateTimeFormatter df2 = DateTimeFormatter.ofPattern("yyyy/MM/dd");
    DateTimeFormatter df3 = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");

    LocalDateTime ldt = LocalDateTime.parse(dateStr1,df1);
    
4.LocalDate转String
    方法一:
    DateTimeFormatter df = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
    LocalDateTime time = LocalDateTime.now();
    String localTime = df.format(time);
    System.out.println("localTime = " + localTime);
    方法二:
    LocalDateTime time = LocalDateTime.now();
    String localTime = time.format(DateTimeFormatter.ofPattern("yyyy/MM/dd HH:mm:ss"));
    System.out.println("localTime = " + localTime);
    
5.常用方法
    // 取本月第1天：
    LocalDate firstDayOfThisMonth = today.with(TemporalAdjusters.firstDayOfMonth()); // 2017-03-01
    // 取本月第2天：
    LocalDate secondDayOfThisMonth = today.withDayOfMonth(2); // 2017-03-02
    // 取本月最后一天，再也不用计算是28，29，30还是31：
    LocalDate lastDayOfThisMonth = today.with(TemporalAdjusters.lastDayOfMonth()); // 2017-12-31
    // 取下一天：
    LocalDate firstDayOf2015 = lastDayOfThisMonth.plusDays(1); // 变成了2018-01-01
    // 取2017年1月第一个周一
    LocalDate firstMondayOf2015 = LocalDate.parse("2017-01-01").with(TemporalAdjusters.firstInMonth(DayOfWeek.MONDAY)); // 2017-01-02
```
