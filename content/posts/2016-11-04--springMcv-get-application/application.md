---
title: SpringMvc从Context中获得Bean对象
subTitle: springMvc
category: java
cover: java.png
---

1. 实现ApplicationContextAware接口

   ```java
   public class SpringContextUtil implements ApplicationContextAware {  
       // Spring应用上下文环境  
       private static ApplicationContext applicationContext;  
       /** 
        * 实现ApplicationContextAware接口的回调方法，设置上下文环境 
        *  
        * @param applicationContext 
        */  
       public void setApplicationContext(ApplicationContext applicationContext) {  
           SpringContextUtil.applicationContext = applicationContext;  
       }  
       /** 
        * @return ApplicationContext 
        */  
       public static ApplicationContext getApplicationContext() {  
           return applicationContext;  
       }  
       /** 
        * 获取对象 
        *  
        * @param name 
        * @return Object
        * @throws BeansException 
        */  
       public static Object getBean(String name) throws BeansException {  
           return applicationContext.getBean(name);  
       }  
   }
   ```


2. 通过Spring提供的ContextLoader

   ```java
   WebApplicationContext wac = ContextLoader.getCurrentWebApplicationContext();
   wac.getBean(beanID);
   ```

   ```
   注意:不依赖于servlet,不需要注入的方式。但是需要注意一点，在服务器启动时，Spring容器初始化时，不能通过以下方法获取Spring 容器
   ```

