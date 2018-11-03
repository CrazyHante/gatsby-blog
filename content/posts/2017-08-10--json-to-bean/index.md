---
title: Json转bean 首字母大写异常错误解决
subTitle: Json
category: java
cover: java.png
---

```java
public static Object toBean(String json,Class beanClass) {
    JSONObject jsonObject = JSONObject.fromObject(json);
    JsonConfig config = new JsonConfig();
    config.setJavaIdentifierTransformer(new JavaIdentifierTransformer() {

        public String transformToJavaIdentifier(String str) {
            char[] chars = str.toCharArray();
            if(Character.isUpperCase(chars[0]) && Character.isUpperCase(chars[1])){
                return str;
            }
            chars[0] = Character.toLowerCase(chars[0]);
            return new String(chars);
        }

    });
    config.setRootClass(beanClass);
    return JSONObject.toBean(jsonObject,config);
}
```

>#### json首字母为大写的时候,json转bean的时候会报错
>
>#### json全部为大写的时候 比如IDC 也会报错
>
>#### json首两位为大小也会报错 比如IDType

