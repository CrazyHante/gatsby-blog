---
title: Nexus3 jar包上传
subTitle: maven
category: 教程
cover: 4.png
---

1. 修改权限

   修改maven安装目录下/conf/setting.xml

   增加权限

   ```xml
   <servers>
       <server>
           <id>maven-releases</id>
           <username>admin</username>
           <password>admin123</password>
       </server>
   </servers>
   ```


2. 上传

   ```shell
   mvn deploy:deploy-file  -DgroupId=com.yunxin  -DartifactId=xdoc  -Dversion=1.0.0  -Dpackaging=jar  -Dfile=XDocService.jar -Durl=http://10.10.2.44/repository/maven-releases/ -DrepositoryId=maven-releases
   ```

   -Dfile jar包所在路径

   -Durl   maven仓库url

   -DrepositoryId  maven仓库名称 与权限service配置Id一致 

