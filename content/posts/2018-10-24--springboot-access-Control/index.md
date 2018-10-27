---
title: springboot跨域设置
subTitle: springboot
category: java
cover: 1.png
---

#### 跨域设置

```java
@Component
public class CharacterFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
    }
    
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        //转换
        HttpServletRequest request = (HttpServletRequest) servletRequest;
        HttpServletResponse response = (HttpServletResponse) servletResponse;
        //设置编码
        request.setCharacterEncoding("UTF-8");
        response.setCharacterEncoding("UTF-8");
        //设置允许访问的域名
        response.setHeader("Access-Control-Allow-Origin", "*");
        //设置允许访问的方法
        response.setHeader("Access-Control-Allow-Methods", "POST, PUT, GET, OPTIONS, DELETE");
        //返回结果可以用于缓存的最长时间 秒
        response.setHeader("Access-Control-Max-Age", "3600");
        //将会在正式请求的字段中出现的首部信息
        response.setHeader("Access-Control-Allow-Headers", "Content-Type, Access-Control-Allow-Headers, Authorization, X-Requested-With");
        //浏览器可读取的字段
        response.setHeader("Access-Control-Expose-Headers","x-filename");
        response.setHeader("Cache-Control", "no-cache, no-store, must-revalidate");
        response.setHeader("Pragma", "no-cache");
        //使得getInputStream可以多次调用
        MultiReadHttpServletRequest multiReadRequest = new MultiReadHttpServletRequest((HttpServletRequest) request);
        filterChain.doFilter(multiReadRequest, response);
    }

    @Override
    public void destroy() {
    }
}
```



**注意**

1. Access-Control-Allow-Origin:如果设置 "Access-Control-Allow-Origin":"*"，则允许所有域名的脚本访问该资源。 "Access-Control-Allow-Origin":"<http://www.phpddt.com.com>",允许特定的域名访问。
2. Access-Control-Max-Age:在Firefox中，上限是24小时 （即86400秒），而在Chromium 中则是10分钟（即600秒）。Chromium 同时规定了一个默认值 5 秒。 如果值为 -1，则表示禁用缓存，每一次请求都需要提供预检请求，即用OPTIONS请求进行检测（即preflight请求-options）。 Access-Control-Max-Age的设置针对完全一样的url，如果url加上路径参数，其中一个url的Access-Control-Max-Age设置对另一个url没有效果



**遇到的问题**

在进行跨域调用的时候发现在Controller层写的Response.setHeader("x-filename",URLEncoder.encode(fileName,"UTF-8"));无法被跨域的浏览器取得,查阅后得知,在跨域请求中,会过滤掉请求头,在Access-Control-Expose-Headers中设置对应的header字段可使浏览器获得在前端获取该字段的权限



--------------------------------------

> 参考:http://www.ruanyifeng.com/blog/2012/09/xmlhttprequest_level_2.html