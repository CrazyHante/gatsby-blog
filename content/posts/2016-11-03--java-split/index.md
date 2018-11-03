---
title： split方法
subTitle: java
category: Focus
cover: 2.png
---

- 如果用“.”作为分隔的话,必须是如下写法,String.split("\\."),这样才能正确的分隔开,不能用String.split(".");

- 如果用“|”作为分隔的话,必须是如下写法,String.split("\\|"),这样才能正确的分隔开,不能用String.split("|");

​         **“.”和“|”都是转义字符,必须得加"\\";**

- 如果在一个字符串中有多个分隔符,可以用“|”作为连字符,比如,“acount=? and uu =? or n=?”,把三个都分隔出来,可以用String.split("and|or");

使用String.split方法分隔字符串时,分隔符如果用到一些特殊字符,可能会得不到我们预期的结果。 



```java
	String[] aa = "aaa|bbb|ccc".split("|");
    //String[] aa = "aaa|bbb|ccc".split("\\|"); 这样才能得到正确的结果
    for (int i = 0 ; i <aa.length ; i++ ) {
      System.out.println("--"+aa[i]); 
    }
```

