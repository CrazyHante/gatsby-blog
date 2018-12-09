---
title: SpirngBoot利用AOP为接口加密加签
subTitle: springBoot
category: java
cover: java.png
---

​	最近工作与某头条，某金融机构做三方开发，接口需要做加密（RSA）加签（MD5）的操作，三方约定采用统一的加密加签方案，但是由于对于中间方来说，接口来源有可能是两方，所以需要判断来源选择不同的公钥加密。在思考后选择了AOP实现，代码如下。

Aop实现代码：

```java
package com.yunxin.aspects;

import com.alibaba.fastjson.JSON;
import com.yunxin.Enum.Constants;
import com.yunxin.config.AppConfig;
import com.yunxin.exception.DecryptException;
import com.yunxin.exception.ValidateException;
import com.yunxin.model.TouTiaoRequest;
import com.yunxin.model.CommonResponse;
import com.yunxin.model.YqbRequest;
import com.yunxin.utils.LoanHelper;
import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import org.springframework.util.StringUtils;

/**
 * @author yht
 * @Description:
 * @date 2018/11/27
 */
@Aspect
@Component
@Slf4j
public class EncryptAndDecryptAspect {
    @Autowired
    AppConfig appConfig;

    @Pointcut(value = "execution(* com.yunxin.controller..*.*(..)) && @annotation(com.yunxin.annotations.InDecrypt)")
    public void controllerCut() {}

    @Around("controllerCut()")
    public Object controllerAround(ProceedingJoinPoint point) throws Throwable {
        long start = System.currentTimeMillis();
        Object[] args = point.getArgs();
        String publicKey = null;
        String transSeqNo = null;
        for (int h = 0; h < args.length; h++) {
            Object obj = args[h];
            if (obj instanceof TouTiaoRequest) {
                TouTiaoRequest request = (TouTiaoRequest) obj;
                validateRequest(request,appConfig.getTtMerchantId());
                transSeqNo = request.getTransSeqNo();
                boolean upload = false;
                if (point.getSignature().getName().equals(Constants.UPLOAD_INTERFACE)){
                    upload = true;
                }
                log.info("{}请求验签,请求参数为{},流水号为:{}"
                        , point.getSignature().getName()
                        , upload?"内容过长,不打印":JSON.toJSON(request)
                        , transSeqNo);
                boolean verify = LoanHelper.verify(request, request.getSign(), 			  appConfig.getSalt());
                log.info(verify ? "{}请求验签成功" : "{}请求验签失败", transSeqNo, transSeqNo);
                if (verify) {
                    try {
                        String content = LoanHelper.decryptByRSA(request.getBizContent(), appConfig.getPrivateKey());
                        log.info("{}请求解密完成,明文内容为{},流水号为:{}"
                                , point.getSignature().getName()
                                , upload?String.format("内容过长,不打印,长度为:%d",content.length()):content
                                , transSeqNo);
                        request.setBizContent(content);
                    } catch (Exception e) {
                        log.info("{}请求解密异常,异常为:{},流水号为:{}", point.getSignature().getName()
                                ,JSON.toJSON(e), transSeqNo);
                        throw new DecryptException("解密失败");
                    }
                } else {
                    throw new DecryptException("验签失败");
                }
                if(request.getMerchantId().equals(appConfig.getTtMerchantId())){
                    publicKey = appConfig.getTtPublicKey();
                }else {
                    publicKey = appConfig.getYqbPublicKey();
                }
            }
            if (obj instanceof YqbRequest) {
                YqbRequest request = (YqbRequest) obj;
                validateRequest(request,appConfig.getYqbCompanyId());
                transSeqNo = request.getTransSeqNo();

                log.info("{}请求验签,请求参数为{},流水号为:{}"
                        , point.getSignature().getName()
                        , JSON.toJSON(request)
                        , transSeqNo);
                boolean verify = LoanHelper.verify(request, request.getSign(), appConfig.getSalt());
                log.info(verify ? "{}请求验签成功" : "{}请求验签失败", transSeqNo, transSeqNo);
                if (verify) {
                    try {
                        String content = LoanHelper.decryptByRSA(request.getBizContent(), appConfig.getPrivateKey());
                        log.info("{}请求解密完成,明文内容为{},流水号为:{}"
                                , point.getSignature().getName()
                                , content
                                , transSeqNo);
                        request.setBizContent(content);
                    } catch (Exception e) {
                        log.info("{}请求解密异常,异常为:{},流水号为:{}", point.getSignature().getName()
                                ,JSON.toJSON(e), transSeqNo);
                        throw new DecryptException("解密失败");
                    }
                } else {
                    throw new DecryptException("验签失败");
                }
                if(!request.getCompanyId().equals(appConfig.getYqbCompanyId())){
                    publicKey = appConfig.getTtPublicKey();
                }else {
                    publicKey = appConfig.getYqbPublicKey();
                }
            }
        }
        Object result = point.proceed();

        CommonResponse response = (CommonResponse) result;
        String encryptContent = null;
        log.info("{}响应加密,加密前内容为{},流水号为:{}", point.getSignature().getName()
                ,JSON.toJSONString(response), transSeqNo);
        try {
            encryptContent = LoanHelper.encryptByRSA(response.getBizContent()==null?"":response.getBizContent(), publicKey);
        } catch (Exception e) {
            throw new DecryptException("加密失败");
        }
        response.setBizContent(encryptContent);
        String sign = null;
        try {
            sign = LoanHelper.sign(response, appConfig.getSalt());
        } catch (Exception e) {
            throw new DecryptException("加签失败");
        }
        response.setSign(sign);
        log.info("{}响应加密,加密后内容为{},流水号为:{},执行{}ms",point.getSignature().getName()
                ,JSON.toJSONString(response), transSeqNo,System.currentTimeMillis()-start);
        return response;
    }


    @Pointcut(value = "execution(* com.yunxin.service..*.*(..)) && @annotation(com.yunxin.annotations.OutDecrypt)")
    public void serviceCut() {}

    @Around("serviceCut()")
    public Object servcieAround(ProceedingJoinPoint point) throws Throwable {
        long start = System.currentTimeMillis();
        Object[] args = point.getArgs();
        for (int h = 0; h < args.length; h++) {
            Object obj = args[h];
            if (obj instanceof TouTiaoRequest) {
                TouTiaoRequest request = (TouTiaoRequest) obj;
                String publicKey = null;
                if(request.getMerchantId().equals(appConfig.getTtMerchantId())){
                    publicKey = appConfig.getTtPublicKey();
                }else {
                    publicKey = appConfig.getYqbPublicKey();
                }
                log.info("{}转发请求加密,加密前内容为{},流水号为:{}",point.getSignature().getName()
                        ,JSON.toJSONString(request), request.getTransSeqNo());
                String encryptContent = null;
                try {
                    encryptContent = LoanHelper.encryptByRSA(request.getBizContent()==null?"":request.getBizContent(), publicKey);
                } catch (Exception e) {
                    throw new DecryptException("加密失败");
                }
                request.setBizContent(encryptContent);
                String sign = null;
                try {
                    sign = LoanHelper.sign(request, appConfig.getSalt());
                } catch (Exception e) {
                    throw new DecryptException("加签失败");
                }
                request.setSign(sign);
                log.info("{}转发请求加密,加密后内容为{},流水号为为:{}",point.getSignature().getName()
                        ,JSON.toJSONString(request), request.getTransSeqNo());

            }
            if (obj instanceof YqbRequest) {
                YqbRequest request = (YqbRequest) obj;
                String publicKey = null;
                if(request.getCompanyId().equals(appConfig.getTtCompanyId())){
                    publicKey = appConfig.getTtPublicKey();
                }else {
                    publicKey = appConfig.getYqbPublicKey();
                }
                log.info("{}转发请求加密,加密前内容为{},流水号为:{}",point.getSignature().getName()
                        ,JSON.toJSONString(request), request.getTransSeqNo());
                String encryptContent = null;
                try {
                    encryptContent = LoanHelper.encryptByRSA(request.getBizContent(), publicKey);
                } catch (Exception e) {
                    throw new DecryptException("加密失败");
                }
                request.setBizContent(encryptContent);
                String sign = null;
                try {
                    sign = LoanHelper.sign(request, appConfig.getSalt());
                } catch (Exception e) {
                    throw new DecryptException("加签失败");
                }
                request.setSign(sign);
                log.info("{}转发请求加密,加密后内容为{},流水号为为:{}",point.getSignature().getName()
                        ,JSON.toJSONString(request), request.getTransSeqNo());

            }
        }
        Object result = point.proceed();

        CommonResponse response = (CommonResponse) result;
        log.info("{}响应验签,响应参数为{},流水号为:{}", point.getSignature().getName()
                ,JSON.toJSONString(response), response.getTransSeqNo());
        boolean verify = LoanHelper.verify(response, response.getSign(), appConfig.getSalt());
        log.info(verify ? "{}响应验签成功" : "{}响应验签失败", response.getTransSeqNo(), response.getTransSeqNo());
        if (verify) {
            try {
                String content = LoanHelper.decryptByRSA(response.getBizContent(), appConfig.getPrivateKey());
                log.info("{}响应解密完成,明文内容为{},流水号为:{}", point.getSignature().getName()
                        ,content, response.getTransSeqNo());
                response.setBizContent(content);
            } catch (Exception e) {
                log.info("{}响应解密异常,异常为:{},流水号为:{}", point.getSignature().getName()
                        ,JSON.toJSON(e), response.getTransSeqNo());
                throw new DecryptException("解密失败");
            }
        } else {
            throw new DecryptException("验签失败");
        }
        log.info("请求{},{}转发调用执行{}ms",response.getTransSeqNo(),point.getSignature().getName(),System.currentTimeMillis()-start);
        return response;
    }


    private void validateRequest(TouTiaoRequest request, String merId){
        if(StringUtils.isEmpty(request.getMerchantId())){
            throw new ValidateException("merchantId缺失");
        }else if(!merId.equals(request.getMerchantId())){
            throw new ValidateException("merchantId错误");
        }else if(StringUtils.isEmpty(request.getTransSeqNo())){
            throw new ValidateException("transSeqNo缺失");
        }else if(StringUtils.isEmpty(request.getTransReqTime())){
            throw new ValidateException("transReqTime缺失");
        }else if(StringUtils.isEmpty(request.getVersion())){
            throw new ValidateException("version缺失");
        }else if (StringUtils.isEmpty(request.getBizContent())){
            throw new ValidateException("bizContent缺失");
        }else if(StringUtils.isEmpty(request.getSign())){
            throw new ValidateException("sign缺失");
        }
    }
    
    private void validateRequest(YqbRequest request, String companyId){
        if(StringUtils.isEmpty(request.getCompanyId())){
            throw new ValidateException("companyId缺失");
        }else if(!companyId.equals(request.getCompanyId())){
            throw new ValidateException("companyId错误");
        }else if(StringUtils.isEmpty(request.getTransSeqNo())){
            throw new ValidateException("transSeqNo缺失");
        }else if(StringUtils.isEmpty(request.getTransReqTime())){
            throw new ValidateException("transReqTime缺失");
        }else if(StringUtils.isEmpty(request.getVersion())){
            throw new ValidateException("version缺失");
        }else if (StringUtils.isEmpty(request.getBizContent())){
            throw new ValidateException("bizContent缺失");
        }else if(StringUtils.isEmpty(request.getSign())){
            throw new ValidateException("sign缺失");
        }
    }

}

```



在业务流程中同时也调用了自己公司的某些接口，所以定义了两个自定义注解，标注那些接口需要加密，哪些不需要加密。

自定义注解如下：

```java
package com.yunxin.annotations;

import java.lang.annotation.*;

@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface InDecrypt {
}

```

```java
package com.yunxin.annotations;

import java.lang.annotation.*;

@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface OutDecrypt {
}

```

加解密处理失败，抛出自定义异常，为了使返回报文有统一的格式，设置了统一异常处理类。代码如下：

```java
package com.yunxin.exception;

/**
 * @author yht
 * @Description: 加解密异常类
 * @date 2018/11/20
 */
public class DecryptException extends RuntimeException{
    private String msg;

    public DecryptException(String msg) {
        this.msg = msg;
    }

    public DecryptException(String message, String msg) {
        super(message);
        this.msg = msg;
    }

    public DecryptException(String message, Throwable cause, String msg) {
        super(message, cause);
        this.msg = msg;
    }

    public DecryptException(Throwable cause, String msg) {
        super(cause);
        this.msg = msg;
    }

    public DecryptException(String message, Throwable cause, boolean enableSuppression, boolean writableStackTrace, String msg) {
        super(message, cause, enableSuppression, writableStackTrace);
        this.msg = msg;
    }

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }
}

```

```java
package com.yunxin.interceptor;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;
import com.yunxin.Enum.TouTiaoCodeEnum;
import com.yunxin.config.AppConfig;
import com.yunxin.exception.DecryptException;
import com.yunxin.exception.ValidateException;
import com.yunxin.model.CommonResponse;
import com.yunxin.utils.LoanHelper;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.io.IOUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import javax.servlet.http.HttpServletRequest;
import java.io.IOException;
import java.nio.charset.StandardCharsets;

/**
 * @author yht
 * @Description:
 * @date 2018/11/20
 */
@Slf4j
//@ControllerAdvice
@RestControllerAdvice
public class MyExceptionHandler {
    @Autowired
    AppConfig appConfig;

    @ResponseBody
    @ExceptionHandler(value = DecryptException.class)
    public CommonResponse decryptExceptionHandler(HttpServletRequest request, DecryptException ex) {
        log.error("catch DecryptException:{}",ex);
        CommonResponse response = new CommonResponse();
        try {
            String requestContent= IOUtils.toString(request.getInputStream(), StandardCharsets.UTF_8);
            JSONObject obj = JSON.parseObject(requestContent);
            response.setTransSeqNo(obj.getString("transSeqNo"));
            response.setTransReqTime(obj.getString("transReqTime"));
            response.setCode(TouTiaoCodeEnum.CODE_2.getKey());
            response.setMsg(TouTiaoCodeEnum.CODE_2.getValue()+":"+ex.getMsg());

            String sign = LoanHelper.sign(response, appConfig.getSalt());
            response.setSign(sign);

        } catch (IOException e) {
            e.printStackTrace();
        }

        return response;
    }

    @ResponseBody
    @ExceptionHandler(value = ValidateException.class)
    public CommonResponse validateExceptionHandler(HttpServletRequest request, ValidateException ex) {
        log.error("catch ValidateException:{}",ex);
        CommonResponse response = new CommonResponse();
        try {
            String requestContent= IOUtils.toString(request.getInputStream(), StandardCharsets.UTF_8);
            JSONObject obj = JSON.parseObject(requestContent);
            response.setTransSeqNo(obj.getString("transSeqNo"));
            response.setTransReqTime(obj.getString("transReqTime"));
            response.setCode(TouTiaoCodeEnum.CODE_1.getKey());
            response.setMsg(TouTiaoCodeEnum.CODE_1.getValue()+":"+ex.getMsg());

            String sign = LoanHelper.sign(response, appConfig.getSalt());
            response.setSign(sign);

        } catch (IOException e) {
            e.printStackTrace();
        }

        return response;
    }

    @ResponseBody
    @ExceptionHandler(value = Exception.class)
    public CommonResponse otherExceptionHandler(HttpServletRequest request, Exception ex) {
        log.error("catch Exception:{}",ex);
        CommonResponse response = new CommonResponse();
        try {
            String requestContent= IOUtils.toString(request.getInputStream(), StandardCharsets.UTF_8);
            JSONObject obj = JSON.parseObject(requestContent);
            response.setTransSeqNo(obj.getString("transSeqNo"));
            response.setTransReqTime(obj.getString("transReqTime"));
            response.setCode(TouTiaoCodeEnum.CODE_4.getKey());
            response.setMsg(TouTiaoCodeEnum.CODE_4.getValue()+":"+ex.getMessage());

            String sign = LoanHelper.sign(response, appConfig.getSalt());
            response.setSign(sign);
        } catch (IOException e) {
            e.printStackTrace();
        }

        return response;
    }

}

```

