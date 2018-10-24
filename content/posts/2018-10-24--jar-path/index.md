---
title: springboot获取部署jar包绝对路径
category: "工具类"
cover: 1.png
---

1. 

   ```java
   ClassUtils.getDefaultClassLoader().getResource("").getPath();
   ```


2. 

   ```java
   StoreClient.class.getProtectionDomain().getCodeSource().getLocation().getPath();
   ```


