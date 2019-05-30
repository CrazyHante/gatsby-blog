---
title: RestTemplete超时设置
subTitle: springBoot
category: java
cover: java.png
---

​	

```java
@Configuration
public class RestTempleteConfig{
    @Bean
    public RestTemplate customRestTemplate(){
        HttpComponentsClientHttpRequestFactory httpRequestFactory 
            = new HttpComponentsClientHttpRequestFactory();
        
        httpRequestFactory.setConnectionRequestTimeout(3000);
        httpRequestFactory.setConnectTimeout(3000);
        httpRequestFactory.setReadTimeout(3000);

        return new RestTemplate(httpRequestFactory);
    }
}

```

connectionRequestTimout：指从连接池获取连接的超时时间,一般指可用连接不够的等待时间

connetionTimeout：指客户端和服务器建立连接超时

ReadTimeout：指客户端从服务器读取数据超时



***HttpComponentsClientHttpRequestFactory 和 SimpleClientHttpRequestFactory 区别:***

​	HttpComponentsClientHttpRequestFactory 可用于连接池配置,而SimpleClientHttpRequestFactory 没有

