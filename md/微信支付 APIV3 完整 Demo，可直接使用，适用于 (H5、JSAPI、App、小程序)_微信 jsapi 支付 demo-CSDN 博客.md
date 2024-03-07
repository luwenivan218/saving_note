> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/zhuocailing3390/article/details/125708191)

**😊 @ 作者： 一恍过去** **💖 @ 主页： [https://blog.csdn.net/zhuocailing3390](https://blog.csdn.net/zhuocailing3390)** **🎊 @ 社区： [Java 技术栈交流](https://bbs.csdn.net/forums/lige)** **🎉 @ 主题： 微信支付 APIV3 完整 Demo，可直接使用，适用于 (H5、JSAPI、App、小程序)** **⏱️ @ 创作时间： 2022 年 07 月 14 日**

#### 目录

*   [前言](#_14)
*   [1、引入 POM](#1POM_17)
*   [2、配置 Yaml](#2Yaml_26)
*   [3、配置密钥文件](#3_48)
*   [4、配置 PayConfig](#4PayConfig_52)
*   [5、定义统一枚举](#5_211)
*   [6、封装统一请求处理](#6_289)
*   [7、回调校验器](#7_368)
*   [8、回调 Body 内容处理](#8Body_523)
*   [9、封装统一代码](#9_559)
*   *   [9.1、统一下单处理](#91_563)
    *   [9.2 、其他接口处理 (退款、查询、取消订单等)](#92__753)
*   [10、支付 / 退款回调通知](#10_1085)

![](https://img-blog.csdnimg.cn/61cfd6093dea40419cde0fa6f46fa1d7.jpeg#pic_center)

前言
--

对[微信支付](https://so.csdn.net/so/search?q=%E5%BE%AE%E4%BF%A1%E6%94%AF%E4%BB%98&spm=1001.2101.3001.7020)的 H5、JSAPI、H5、App、小程序支付方式进行统一，此封装接口适用于普通商户模式支付，如果要进行服务商模式支付可以结合服务商官方 API 进行参数修改（未验证可行性）。

1、引入 POM
--------

```
        <!--微信支付SDK-->
        <dependency>
            <groupId>com.github.wechatpay-apiv3</groupId>
            <artifactId>wechatpay-apache-httpclient</artifactId>
            <version>0.4.7</version>
        </dependency>

```

2、配置 Yaml
---------

```
wxpay:
  #应用编号
  appId: xxxx
  #商户号
  mchId: xxx
  # APIv2密钥
  apiKey: xxxx
  # APIv3密钥
  apiV3Key: xxx
  # 微信支付V3-url前缀
  baseUrl: https://api.mch.weixin.qq.com/v3
  # 支付通知回调, pjm6m9.natappfree.cc 为内网穿透地址
  notifyUrl: http://pjm6m9.natappfree.cc/pay/payNotify
  # 退款通知回调, pjm6m9.natappfree.cc 为内网穿透地址
  refundNotifyUrl: http://pjm6m9.natappfree.cc/pay/refundNotify
  # 密钥路径,resources根目录下
  keyPemPath: apiclient_key.pem
  #商户证书序列号
  serialNo: xxxxx

```

3、配置密钥文件
--------

在商户 / 服务商平台的”账户中心 " => “API 安全” 进行 API 证书、密钥的设置，API 证书主要用于获取 “商户证书序列号” 以及“p12”、“key.pem”、”cert.pem“证书文件，j 将获取的`apiclient_key.pem`文件放在项目的`resources`目录下。  
![](https://img-blog.csdnimg.cn/4db37afcc13e4acaa2045de2be430729.png)

4、配置 PayConfig
--------------

**WechatPayConfig：**

```
import com.wechat.pay.contrib.apache.httpclient.WechatPayHttpClientBuilder;
import com.wechat.pay.contrib.apache.httpclient.auth.PrivateKeySigner;
import com.wechat.pay.contrib.apache.httpclient.auth.Verifier;
import com.wechat.pay.contrib.apache.httpclient.auth.WechatPay2Credentials;
import com.wechat.pay.contrib.apache.httpclient.auth.WechatPay2Validator;
import com.wechat.pay.contrib.apache.httpclient.cert.CertificatesManager;
import com.wechat.pay.contrib.apache.httpclient.exception.HttpCodeException;
import com.wechat.pay.contrib.apache.httpclient.exception.NotFoundException;
import com.wechat.pay.contrib.apache.httpclient.util.PemUtil;
import lombok.Data;
import lombok.extern.slf4j.Slf4j;
import org.apache.http.impl.client.CloseableHttpClient;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.stereotype.Component;

import java.io.IOException;
import java.io.InputStream;
import java.nio.charset.StandardCharsets;
import java.security.GeneralSecurityException;
import java.security.PrivateKey;

/**
 * @Author:
 * @Description:
 **/
@Component
@Data
@Slf4j
@ConfigurationProperties(prefix = "wxpay")
public class WechatPayConfig {
    /**
     * 应用编号
     */
    private String appId;
    /**
     * 商户号
     */
    private String mchId;
    /**
     * 服务商商户号
     */
    private String slMchId;
    /**
     * APIv2密钥
     */
    private String apiKey;
    /**
     * APIv3密钥
     */
    private String apiV3Key;
    /**
     * 支付通知回调地址
     */
    private String notifyUrl;
    /**
     * 退款回调地址
     */
    private String refundNotifyUrl;

    /**
     * API 证书中的 key.pem
     */
    private String keyPemPath;

    /**
     * 商户序列号
     */
    private String serialNo;

    /**
     * 微信支付V3-url前缀
     */
    private String baseUrl;

    /**
     * 获取商户的私钥文件
     * @param keyPemPath
     * @return
     */
    public PrivateKey getPrivateKey(String keyPemPath){

            InputStream inputStream = this.getClass().getClassLoader().getResourceAsStream(keyPemPath);
            if(inputStream==null){
                throw new RuntimeException("私钥文件不存在");
            }
            return PemUtil.loadPrivateKey(inputStream);
    }

    /**
     * 获取证书管理器实例
     * @return
     */
    @Bean
    public  Verifier getVerifier() throws GeneralSecurityException, IOException, HttpCodeException, NotFoundException {

        log.info("获取证书管理器实例");

        //获取商户私钥
        PrivateKey privateKey = getPrivateKey(keyPemPath);

        //私钥签名对象
        PrivateKeySigner privateKeySigner = new PrivateKeySigner(serialNo, privateKey);

        //身份认证对象
        WechatPay2Credentials wechatPay2Credentials = new WechatPay2Credentials(mchId, privateKeySigner);

        // 使用定时更新的签名验证器，不需要传入证书
        CertificatesManager certificatesManager = CertificatesManager.getInstance();
        certificatesManager.putMerchant(mchId,wechatPay2Credentials,apiV3Key.getBytes(StandardCharsets.UTF_8));

        return certificatesManager.getVerifier(mchId);
    }


    /**
     * 获取支付http请求对象
     * @param verifier
     * @return
     */
    @Bean(name = "wxPayClient")
    public CloseableHttpClient getWxPayClient(Verifier verifier)  {

        //获取商户私钥
        PrivateKey privateKey = getPrivateKey(keyPemPath);

        WechatPayHttpClientBuilder builder = WechatPayHttpClientBuilder.create()
                .withMerchant(mchId, serialNo, privateKey)
                .withValidator(new WechatPay2Validator(verifier));

        // 通过WechatPayHttpClientBuilder构造的HttpClient，会自动的处理签名和验签，并进行证书自动更新
        return builder.build();
    }

    /**
     * 获取HttpClient，无需进行应答签名验证，跳过验签的流程
     */
    @Bean(name = "wxPayNoSignClient")
    public CloseableHttpClient getWxPayNoSignClient(){

        //获取商户私钥
        PrivateKey privateKey = getPrivateKey(keyPemPath);

        //用于构造HttpClient
        WechatPayHttpClientBuilder builder = WechatPayHttpClientBuilder.create()
                //设置商户信息
                .withMerchant(mchId, serialNo, privateKey)
                //无需进行签名验证、通过withValidator((response) -> true)实现
                .withValidator((response) -> true);

        // 通过WechatPayHttpClientBuilder构造的HttpClient，会自动的处理签名和验签，并进行证书自动更新
        return builder.build();
    }
}

```

5、定义统一枚举
--------

**WechatPayUrlEnum：**

```
@AllArgsConstructor
@Getter
public enum WechatPayUrlEnum {


	/**
	 * native
	 */
	NATIVE("native"),
	/**
	 * app
	 */
	APP("app"),
	/**
	 * h5
	 */
	H5("h5"),
	/**
	 *  jsapi
	 */
	JSAPI("jsapi"),

	/**
	 *  小程序jsapi
	 */
	SUB_JSAPI("sub_jsapi"),

	/**
	 * APIV3下单
	 */
	PAY_TRANSACTIONS("/pay/transactions/"),

	/**
	 * APIV2下单
	 */
	NATIVE_PAY_V2("/pay/unifiedorder"),

	/**
	 * 查询订单
	 */
	ORDER_QUERY_BY_NO("/pay/transactions/out-trade-no/"),

	/**
	 * 关闭订单
	 */
	CLOSE_ORDER_BY_NO("/pay/transactions/out-trade-no/%s/close"),

	/**
	 * 申请退款
	 */
	DOMESTIC_REFUNDS("/refund/domestic/refunds"),

	/**
	 * 查询单笔退款
	 */
	DOMESTIC_REFUNDS_QUERY("/refund/domestic/refunds/"),

	/**
	 * 申请交易账单
	 */
	TRADE_BILLS("/bill/tradebill"),

	/**
	 * 申请资金账单
	 */
	FUND_FLOW_BILLS("/bill/fundflowbill");

	/**
	 * 类型
	 */
	private final String type;
}

```

![](https://img-blog.csdnimg.cn/778aafa8b1e947e09f5c33c7e1f40d89.jpeg#pic_center)

6、封装统一请求处理
----------

> 封装公共的请求 API，用于在业务请求时的统一处理。

**WechatPayRequest：**

```
import lombok.extern.slf4j.Slf4j;
import org.apache.http.HttpEntity;
import org.apache.http.HttpStatus;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.entity.StringEntity;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.util.EntityUtils;
import org.springframework.stereotype.Component;

import javax.annotation.Resource;
import java.io.IOException;

/**
 * @Author: 
 * @Description:
 **/
@Component
@Slf4j
public class WechatPayRequest {
    @Resource
    private CloseableHttpClient wxPayClient;
    public  String wechatHttpGet(String url) {
        try {
            // 拼接请求参数
            HttpGet httpGet = new HttpGet(url);
            httpGet.setHeader("Accept", "application/json");

            //完成签名并执行请求
            CloseableHttpResponse response = wxPayClient.execute(httpGet);

            return getResponseBody(response);
        }catch (Exception e){
            throw new RuntimeException(e.getMessage());
        }
    }

    public  String wechatHttpPost(String url,String paramsStr) {
        try {
            HttpPost httpPost = new HttpPost(url);
            StringEntity entity = new StringEntity(paramsStr, "utf-8");
            entity.setContentType("application/json");
            httpPost.setEntity(entity);
            httpPost.setHeader("Accept", "application/json");

            CloseableHttpResponse response = wxPayClient.execute(httpPost);
            return getResponseBody(response);
        }catch (Exception e){
            throw new RuntimeException(e.getMessage());
        }
    }

   private String getResponseBody(CloseableHttpResponse response) throws IOException {

            //响应体
           HttpEntity entity = response.getEntity();
           String body = entity==null?"":EntityUtils.toString(entity);
            //响应状态码
            int statusCode = response.getStatusLine().getStatusCode();

            //处理成功,204是，关闭订单时微信返回的正常状态码
            if (statusCode == HttpStatus.SC_OK || statusCode == HttpStatus.SC_NO_CONTENT) {
                log.info("成功, 返回结果 = " + body);
            } else {
                String msg = "微信支付请求失败,响应码 = " + statusCode + ",返回结果 = " + body;
                log.error(msg);
                throw new RuntimeException(msg);
            }
            return body;
    }
}

```

7、回调校验器
-------

> 用于对微信支付成功后的回调数据进行签名验证，保证数据的安全性与真实性。

**WechatPayValidator：**

```
import com.alibaba.fastjson.JSONObject;
import com.alibaba.fastjson.TypeReference;
import com.wechat.pay.contrib.apache.httpclient.auth.Verifier;
import com.wechat.pay.contrib.apache.httpclient.util.AesUtil;
import lombok.extern.slf4j.Slf4j;
import org.apache.http.HttpEntity;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.util.EntityUtils;

import javax.servlet.http.HttpServletRequest;
import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.time.DateTimeException;
import java.time.Duration;
import java.time.Instant;
import java.util.Map;

import static com.wechat.pay.contrib.apache.httpclient.constant.WechatPayHttpHeaders.*;

/**
 * @Author: 
 * @Description:
 **/
@Slf4j
public class WechatPayValidator {
    /**
     * 应答超时时间，单位为分钟
     */
    private static final long RESPONSE_EXPIRED_MINUTES = 5;
    private final Verifier verifier;
    private final String requestId;
    private final String body;


    public WechatPayValidator(Verifier verifier, String requestId, String body) {
        this.verifier = verifier;
        this.requestId = requestId;
        this.body = body;
    }

    protected static IllegalArgumentException parameterError(String message, Object... args) {
        message = String.format(message, args);
        return new IllegalArgumentException("parameter error: " + message);
    }

    protected static IllegalArgumentException verifyFail(String message, Object... args) {
        message = String.format(message, args);
        return new IllegalArgumentException("signature verify fail: " + message);
    }

    public final boolean validate(HttpServletRequest request)  {
        try {
            //处理请求参数
            validateParameters(request);

            //构造验签名串
            String message = buildMessage(request);

            String serial = request.getHeader(WECHAT_PAY_SERIAL);
            String signature = request.getHeader(WECHAT_PAY_SIGNATURE);

            //验签
            if (!verifier.verify(serial, message.getBytes(StandardCharsets.UTF_8), signature)) {
                throw verifyFail("serial=[%s] message=[%s] sign=[%s], request-id=[%s]",
                        serial, message, signature, requestId);
            }
        } catch (IllegalArgumentException e) {
            log.warn(e.getMessage());
            return false;
        }

        return true;
    }

    private  void validateParameters(HttpServletRequest request) {

        // NOTE: ensure HEADER_WECHAT_PAY_TIMESTAMP at last
        String[] headers = {WECHAT_PAY_SERIAL, WECHAT_PAY_SIGNATURE, WECHAT_PAY_NONCE, WECHAT_PAY_TIMESTAMP};

        String header = null;
        for (String headerName : headers) {
            header = request.getHeader(headerName);
            if (header == null) {
                throw parameterError("empty [%s], request-id=[%s]", headerName, requestId);
            }
        }

        //判断请求是否过期
        String timestampStr = header;
        try {
            Instant responseTime = Instant.ofEpochSecond(Long.parseLong(timestampStr));
            // 拒绝过期请求
            if (Duration.between(responseTime, Instant.now()).abs().toMinutes() >= RESPONSE_EXPIRED_MINUTES) {
                throw parameterError("timestamp=[%s] expires, request-id=[%s]", timestampStr, requestId);
            }
        } catch (DateTimeException | NumberFormatException e) {
            throw parameterError("invalid timestamp=[%s], request-id=[%s]", timestampStr, requestId);
        }
    }

    private  String buildMessage(HttpServletRequest request)  {
        String timestamp = request.getHeader(WECHAT_PAY_TIMESTAMP);
        String nonce = request.getHeader(WECHAT_PAY_NONCE);
        return timestamp + "\n"
                + nonce + "\n"
                + body + "\n";
    }

    private  String getResponseBody(CloseableHttpResponse response) throws IOException {
        HttpEntity entity = response.getEntity();
        return (entity != null && entity.isRepeatable()) ? EntityUtils.toString(entity) : "";
    }

    /**
     * 对称解密，异步通知的加密数据
     * @param resource 加密数据
     * @param apiV3Key apiV3密钥
     * @param type 1-支付，2-退款
     * @return
     */
    public static  Map<String, Object> decryptFromResource(String resource,String apiV3Key,Integer type) {

        String msg = type==1?"支付成功":"退款成功";
        log.info(msg+"，回调通知，密文解密");
        try {
        //通知数据
        Map<String, String> resourceMap = JSONObject.parseObject(resource, new TypeReference<Map<String, Object>>() {
        });
        //数据密文
        String ciphertext = resourceMap.get("ciphertext");
        //随机串
        String nonce = resourceMap.get("nonce");
        //附加数据
        String associatedData = resourceMap.get("associated_data");

        log.info("密文： {}", ciphertext);
        AesUtil aesUtil = new AesUtil(apiV3Key.getBytes(StandardCharsets.UTF_8));
        String resourceStr = aesUtil.decryptToString(associatedData.getBytes(StandardCharsets.UTF_8),
                nonce.getBytes(StandardCharsets.UTF_8),
                ciphertext);

        log.info(msg+"，回调通知，解密结果 ： {}", resourceStr);
        return JSONObject.parseObject(resourceStr, new TypeReference<Map<String, Object>>(){});
    }catch (Exception e){
        throw new RuntimeException("回调参数，解密失败！");
        }
    }
}

```

8、回调 Body 内容处理
--------------

**HttpUtils：**

```
public class HttpUtils {

    /**
     * 将通知参数转化为字符串
     * @param request
     * @return
     */
    public static String readData(HttpServletRequest request) {
        BufferedReader br = null;
        try {
            StringBuilder result = new StringBuilder();
            br = request.getReader();
            for (String line; (line = br.readLine()) != null; ) {
                if (result.length() > 0) {
                    result.append("\n");
                }
                result.append(line);
            }
            return result.toString();
        } catch (IOException e) {
            throw new RuntimeException(e);
        } finally {
            if (br != null) {
                try {
                    br.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

```

9、封装统一代码
--------

> 对支付接口进行统一的封装，通过传入不同类型 (H5、JSAPI、App) 的支付方式，封装公共的请求参数，对不同类型特定参数进行单独的处理。

> 商户系统先调用该接口在微信支付服务后台生成预支付交易单，返回正确的预支付交易会话标识后再按 Native、JSAPI、APP、H5 等不同场景生成交易串调起支付。

### 9.1、统一下单处理

**PayController：**

```
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;
import com.alibaba.fastjson.TypeReference;
import com.github.xiaoymin.knife4j.annotations.ApiOperationSupport;
import com.lhz.demo.pay.WechatPayConfig;
import com.lhz.demo.model.enums.WechatPayUrlEnum;
import com.lhz.demo.pay.WechatPayRequest;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import lombok.extern.slf4j.Slf4j;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.util.EntityUtils;
import org.springframework.web.bind.annotation.*;

import javax.annotation.Resource;
import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
import java.io.*;
import java.nio.charset.StandardCharsets;
import java.util.*;


/**
 * @Author: 
 * @Description:
 **/
@Api(tags = "支付接口(API3)")
@RestController
@RequestMapping("/test")
@Slf4j
public class PayController {

    @Resource
    private WechatPayConfig wechatPayConfig;

    @Resource
    private WechatPayRequest wechatPayRequest;

    /**
     * 无需应答签名
     */
    @Resource
    private CloseableHttpClient wxPayNoSignClient;

    /**
     * type：h5、jsapi、app、native、sub_jsapi
     * @param type
     * @return
     */
    @ApiOperation(value = "统一下单-统一接口", notes = "统一下单-统一接口")
    @ApiOperationSupport(order = 10)
    @GetMapping("/transactions")
    public Map<String,Object> transactions(String type) {
        log.info("统一下单API，支付方式：{}",type);

        // 统一参数封装
        Map<String, Object> params = new HashMap<>(8);
        params.put("appid", wechatPayConfig.getAppId());
        params.put("mchid", wechatPayConfig.getMchId());
        params.put("description", "测试商品");
        int outTradeNo = new Random().nextInt(999999999);
        params.put("out_trade_no", outTradeNo + "");
        params.put("notify_url", wechatPayConfig.getNotifyUrl());

        Map<String, Object> amountMap = new HashMap<>(4);
        // 金额单位为分
        amountMap.put("total", 1);
        amountMap.put("currency", "CNY");
        params.put("amount", amountMap);

        // 场景信息
        Map<String, Object> sceneInfoMap = new HashMap<>(4);
        // 客户端IP
        sceneInfoMap.put("payer_client_ip", "127.0.0.1");
        // 商户端设备号（门店号或收银设备ID）
        sceneInfoMap.put("device_id", "127.0.0.1");

        // 除H5与JSAPI有特殊参数外，其他的支付方式都一样
        if (type.equals(WechatPayUrlEnum.H5.getType())) {

            Map<String, Object> h5InfoMap = new HashMap<>(4);
            // 场景类型:iOS, Android, Wap
            h5InfoMap.put("type", "IOS");
            sceneInfoMap.put("h5_info", h5InfoMap);
        } else if (type.equals(WechatPayUrlEnum.JSAPI.getType()) || type.equals(WechatPayUrlEnum.SUB_JSAPI.getType())) {
            Map<String, Object> payerMap = new HashMap<>(4);
            payerMap.put("openid", "123123123");
            params.put("payer", payerMap);
        }

        params.put("scene_info", sceneInfoMap);

        String paramsStr = JSON.toJSONString(params);
        log.info("请求参数 ===> {}" + paramsStr);

        // 重写type值，因为小程序会多一个下划线(sub_type)
        String[] split = type.split("_");
        String  newType = split[split.length - 1];

        String resStr = wechatPayRequest.wechatHttpPost(wechatPayConfig.getBaseUrl().concat(WechatPayUrlEnum.PAY_TRANSACTIONS.getType().concat(newType)), paramsStr);
        Map<String, Object> resMap = JSONObject.parseObject(resStr, new TypeReference<Map<String, Object>>(){});

        Map<String, Object> signMap = paySignMsg(resMap, type);
        resMap.put("type",type);
        resMap.put("signMap",signMap);
        return resMap;
    }


    private  Map<String, Object>  paySignMsg(Map<String, Object> map,String type){
        // 设置签名信息,Native与H5不需要
        if(type.equals(WechatPayUrlEnum.H5.getType()) || type.equals(WechatPayUrlEnum.NATIVE.getType()) ){
            return null;
        }

        long timeMillis = System.currentTimeMillis();
        String appId = wechatPayConfig.getAppId();
        String timeStamp = timeMillis/1000+"";
        String nonceStr = timeMillis+"";
        String prepayId = map.get("prepay_id").toString();
        String packageStr = "prepay_id="+prepayId;

        // 公共参数
        Map<String, Object> resMap = new HashMap<>();
        resMap.put("nonceStr",nonceStr);
        resMap.put("timeStamp",timeStamp);

        // JSAPI、SUB_JSAPI(小程序)
        if(type.equals(WechatPayUrlEnum.JSAPI.getType()) || type.equals(WechatPayUrlEnum.SUB_JSAPI.getType()) ) {
            resMap.put("appId",appId);
            resMap.put("package", packageStr);
            // 使用字段appId、timeStamp、nonceStr、package进行签名
            String paySign = createSign(resMap);
            resMap.put("paySign", paySign);
            resMap.put("signType", "HMAC-SHA256");
        }
        // APP
        if(type.equals(WechatPayUrlEnum.APP.getType())) {
            resMap.put("appid",appId);
            resMap.put("prepayid", prepayId);
            // 使用字段appId、timeStamp、nonceStr、prepayId进行签名
            String sign = createSign(resMap);
            resMap.put("package", "Sign=WXPay");
            resMap.put("partnerid", wechatPayConfig.getMchId());
            resMap.put("sign", sign);
            resMap.put("signType", "HMAC-SHA256");
        }
        return resMap;
    }

    /**
     * 获取加密数据
     */
    private  String createSign(Map<String, Object> params){
        try {
            Map<String, Object> treeMap = new TreeMap<>(params);
            List<String> signList = new ArrayList<>(5);
            for (Map.Entry<String, Object> entry : treeMap.entrySet())
            {
                signList.add(entry.getKey() + "=" + entry.getValue());
            }
            String signStr = String.join("&", signList);

            signStr = signStr+"&key="+wechatPayConfig.getApiV3Key();
            System.out.println(signStr);

            Mac sha = Mac.getInstance("HmacSHA256");
            SecretKeySpec secretKey = new SecretKeySpec(wechatPayConfig.getApiV3Key().getBytes(StandardCharsets.UTF_8), "HmacSHA256");
            sha.init(secretKey);
            byte[] array = sha.doFinal(signStr.getBytes(StandardCharsets.UTF_8));
            StringBuilder sb = new StringBuilder();
            for (byte item : array) {
                sb.append(Integer.toHexString((item & 0xFF) | 0x100), 1, 3);
            }
            signStr = sb.toString().toUpperCase();
            System.out.println(signStr);

            return signStr;
        }catch (Exception e){
            throw new RuntimeException("加密失败！");
        }
    }
}

```

### 9.2 、其他接口处理 (退款、查询、取消订单等)

**PayController：**

```
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;
import com.alibaba.fastjson.TypeReference;
import com.github.xiaoymin.knife4j.annotations.ApiOperationSupport;
import com.lhz.demo.pay.WechatPayConfig;
import com.lhz.demo.model.enums.WechatPayUrlEnum;
import com.lhz.demo.pay.WechatPayRequest;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import lombok.extern.slf4j.Slf4j;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.util.EntityUtils;
import org.springframework.web.bind.annotation.*;

import javax.annotation.Resource;
import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
import java.io.*;
import java.nio.charset.StandardCharsets;
import java.util.*;


/**
 * @Author: LiHuaZhi
 * @Description:
 **/
@Api(tags = "支付接口(API3)")
@RestController
@RequestMapping("/test")
@Slf4j
public class PayController {

    @Resource
    private WechatPayConfig wechatPayConfig;

    @Resource
    private WechatPayRequest wechatPayRequest;

    /**
     * 无需应答签名
     */
    @Resource
    private CloseableHttpClient wxPayNoSignClient;


    @ApiOperation(value = "根据订单号查询订单-统一接口", notes = "根据订单号查询订单-统一接口")
    @ApiOperationSupport(order = 15)
    @GetMapping("/transactions/{orderNo}")
    public  Map<String, Object> transactionsByOrderNo(@PathVariable("orderNo") String orderNo) {

        // TODO 如果是扫码支付时，该接口就很有必要，应该前端通过轮询的方式请求该接口查询订单是否支付成功

        log.info("根据订单号查询订单，订单号： {}", orderNo);

        String url = wechatPayConfig.getBaseUrl().concat(WechatPayUrlEnum.ORDER_QUERY_BY_NO.getType().concat(orderNo))
                .concat("?mchid=").concat(wechatPayConfig.getMchId());

        String res = wechatPayRequest.wechatHttpGet(url);
        log.info("查询订单结果：{}",res);

        Map<String, Object> resMap = JSONObject.parseObject(res, new TypeReference<Map<String, Object>>(){});

        String outTradeNo = resMap.get("out_trade_no").toString();
        String appId = resMap.get("appid").toString();
        String mchId = resMap.get("mchid").toString();
        /**
         *         交易状态，枚举值：
         *         SUCCESS：支付成功
         *         REFUND：转入退款
         *         NOTPAY：未支付
         *         CLOSED：已关闭
         *         REVOKED：已撤销（仅付款码支付会返回）
         *         USERPAYING：用户支付中（仅付款码支付会返回）
         *         PAYERROR：支付失败（仅付款码支付会返回）
         */
        String tradeState = resMap.get("trade_state").toString();

        log.info("outTradeNo："+outTradeNo);
        log.info("appId："+appId);
        log.info("mchId："+mchId);
        log.info("tradeState："+tradeState);

        return resMap;
    }

    /**
     * 关闭(取消)订单
     * @param orderNo
     * @return
     */
    @ApiOperation(value = "关闭(取消)订单-统一接口", notes = "关闭(取消)订单-统一接口")
    @ApiOperationSupport(order = 20)
    @PostMapping("/closeOrder/{orderNo}")
    public void closeOrder(@PathVariable("orderNo") String orderNo) {

        // TODO 用于在客户下单后，不进行支付，取消订单的场景
        log.info("根据订单号取消订单，订单号： {}", orderNo);

        String url = String.format(WechatPayUrlEnum.CLOSE_ORDER_BY_NO.getType(), orderNo);
        url = wechatPayConfig.getBaseUrl().concat(url);

        // 设置参数
        Map<String, String> params = new HashMap<>(2);
        params.put("mchid", wechatPayConfig.getMchId());

        String paramsStr = JSON.toJSONString(params);
        log.info("请求参数 ===> {}" + paramsStr);

        String res = wechatPayRequest.wechatHttpPost(url,paramsStr);
    }

    /**
     * 申请退款
     * @param orderNo
     */
    @ApiOperation(value = "申请退款-统一接口", notes = "申请退款-统一接口")
    @ApiOperationSupport(order = 25)
    @PostMapping("/refundOrder/{orderNo}")
    public void refundOrder(@PathVariable("orderNo") String orderNo) {

        log.info("根据订单号申请退款，订单号： {}", orderNo);

        String url = wechatPayConfig.getBaseUrl().concat(WechatPayUrlEnum.DOMESTIC_REFUNDS.getType());

        // 设置参数
        Map<String, Object> params = new HashMap<>(2);
        // 订单编号
        params.put("out_trade_no", orderNo);
        // 退款单编号 - 自定义
        int outRefundNo = new Random().nextInt(999999999);
        log.info("退款申请号：{}",outRefundNo);
        params.put("out_refund_no",outRefundNo+"");
        // 退款原因
        params.put("reason","申请退款");
        // 退款通知回调地址
        params.put("notify_url", wechatPayConfig.getRefundNotifyUrl());

        Map<String, Object>  amountMap =new HashMap<>();
        //退款金额，单位：分
        amountMap.put("refund", 1);
        //原订单金额，单位：分
        amountMap.put("total", 1);
        //退款币种
        amountMap.put("currency", "CNY");
        params.put("amount", amountMap);

        String paramsStr = JSON.toJSONString(params);
        log.info("请求参数 ===> {}" + paramsStr);


        String res = wechatPayRequest.wechatHttpPost(url,paramsStr);

        log.info("退款结果：{}",res);
    }

    /**
     * 查询单笔退款信息
     * @param refundNo
     * @return
     */
    @ApiOperation(value = "查询单笔退款信息-统一接口", notes = "查询单笔退款信息-统一接口")
    @ApiOperationSupport(order = 30)
    @GetMapping("/queryRefundOrder/{refundNo}")
    public  Map<String, Object> queryRefundOrder(@PathVariable("refundNo") String refundNo) {

        log.info("根据订单号查询退款订单，订单号： {}", refundNo);

        String url = wechatPayConfig.getBaseUrl().concat(WechatPayUrlEnum.DOMESTIC_REFUNDS_QUERY.getType().concat(refundNo));

        String res = wechatPayRequest.wechatHttpGet(url);
        log.info("查询退款订单结果：{}",res);

        Map<String, Object> resMap = JSONObject.parseObject(res, new TypeReference<Map<String, Object>>(){});

        String successTime = resMap.get("success_time").toString();
        String refundId = resMap.get("refund_id").toString();
        /**
         * 款到银行发现用户的卡作废或者冻结了，导致原路退款银行卡失败，可前往商户平台-交易中心，手动处理此笔退款。
         * 枚举值：
         * SUCCESS：退款成功
         * CLOSED：退款关闭
         * PROCESSING：退款处理中
         * ABNORMAL：退款异常
         */
        String status = resMap.get("status").toString();

        /**
         * 枚举值：
         * ORIGINAL：原路退款
         * BALANCE：退回到余额
         * OTHER_BALANCE：原账户异常退到其他余额账户
         * OTHER_BANKCARD：原银行卡异常退到其他银行卡
         */
        String channel = resMap.get("channel").toString();

        log.info("successTime："+successTime);
        log.info("channel："+channel);
        log.info("refundId："+refundId);
        log.info("status："+status);

        // TODO 在查询单笔退款信息时，可以再去查询一次订单的状态，保证该订单已经退款完毕了

        return resMap;
    }

    /**
     * 申请交易账单
     * @param billDate 格式yyyy-MM-dd 仅支持三个月内的账单下载申请 ，如果传入日期未为当天则会出错
     * @param billType 分为：ALL、SUCCESS、REFUND
     * ALL：返回当日所有订单信息（不含充值退款订单）
     * SUCCESS：返回当日成功支付的订单（不含充值退款订单）
     * REFUND：返回当日退款订单（不含充值退款订单）
     * @return
     */
    @ApiOperation(value = "申请交易账单-统一接口", notes = "申请交易账单-统一接口")
    @ApiOperationSupport(order = 35)
    @GetMapping("/tradeBill")
    public  String tradeBill(@RequestParam("billDate") String billDate, @RequestParam("billType")  String billType) {

        log.info("申请交易账单，billDate：{}，billType：{}", billDate,billType);

        String url = wechatPayConfig.getBaseUrl().concat(WechatPayUrlEnum.TRADE_BILLS.getType())
                .concat("?bill_date=").concat(billDate).concat("&bill_type=").concat(billType);

        String res = wechatPayRequest.wechatHttpGet(url);
        log.info("查询退款订单结果：{}",res);

        Map<String, Object> resMap = JSONObject.parseObject(res, new TypeReference<Map<String, Object>>(){});

        String downloadUrl = resMap.get("download_url").toString();

        return downloadUrl;
    }

    /**
     *
     * @param billDate 格式yyyy-MM-dd 仅支持三个月内的账单下载申请，如果传入日期未为当天则会出错
     * @param accountType 分为：BASIC、OPERATION、FEES
     * BASIC：基本账户
     * OPERATION：运营账户
     * FEES：手续费账户
     * @return
     */
    @ApiOperation(value = "申请资金账单-统一接口", notes = "申请资金账单-统一接口")
    @ApiOperationSupport(order = 40)
    @GetMapping("/fundFlowBill")
    public String fundFlowBill(@RequestParam("billDate") String billDate, @RequestParam("accountType")  String accountType) {

        log.info("申请交易账单，billDate：{}，accountType：{}", billDate,accountType);

        String url = wechatPayConfig.getBaseUrl().concat(WechatPayUrlEnum.FUND_FLOW_BILLS.getType())
                .concat("?bill_date=").concat(billDate).concat("&account_type=").concat(accountType);

        String res = wechatPayRequest.wechatHttpGet(url);
        log.info("查询退款订单结果：{}",res);

        Map<String, Object> resMap = JSONObject.parseObject(res, new TypeReference<Map<String, Object>>(){});

        String downloadUrl = resMap.get("download_url").toString();

        return downloadUrl;
    }


    @ApiOperation(value = "下载账单-统一接口", notes = "下载账单-统一接口")
    @ApiOperationSupport(order = 45)
    @GetMapping("/downloadBill")
    public void downloadBill(String downloadUrl) {

        log.info("下载账单，下载地址：{}",downloadUrl);

        HttpGet httpGet = new HttpGet(downloadUrl);
        httpGet.addHeader("Accept", "application/json");

        CloseableHttpResponse response =null;
        try {
            //使用wxPayClient发送请求得到响应
            response =  wxPayNoSignClient.execute(httpGet);

            String body = EntityUtils.toString(response.getEntity());

            int statusCode = response.getStatusLine().getStatusCode();
            if (statusCode == 200 || statusCode == 204) {
                log.info("下载账单，返回结果 = " + body);
            } else {
                throw new RuntimeException("下载账单异常, 响应码 = " + statusCode+ ", 下载账单返回结果 = " + body);
            }
            // TODO 将body内容转为excel存入本地或者输出到浏览器，演示存入本地
            writeStringToFile(body);

        }catch (Exception e){
            throw new RuntimeException(e.getMessage());
        }
        finally {
            if(response!=null) {
                try {
                    response.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    private void writeStringToFile(String body) {
        FileWriter fw = null;
        try {
            String filePath = "C:\\Users\\lhz12\\Desktop\\wxPay.txt";
             fw = new FileWriter(filePath, true);
            BufferedWriter  bw = new BufferedWriter(fw);
             bw.write(body);
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            try {
                if(fw!=null) {
                    fw.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}

```

10、支付 / 退款回调通知
--------------

> 支付 / 退款回调通知：在进行支付或退款操作后，支付平台向商户发送的异步通知，用于告知支付或退款的结果和相关信息。这种回调通知是实现[支付系统](https://so.csdn.net/so/search?q=%E6%94%AF%E4%BB%98%E7%B3%BB%E7%BB%9F&spm=1001.2101.3001.7020)与商户系统之间数据同步和交互的重要方式。

> 支付 / 退款回调通知包含一些关键信息，如订单号、交易金额、支付 / 退款状态、支付平台的交易流水号等。

> 在回调后，通过解析回调数据，一定需要验证签名的合法性。

**NotifyController：**

```
import com.alibaba.fastjson.JSONObject;
import com.alibaba.fastjson.TypeReference;
import com.github.xiaoymin.knife4j.annotations.ApiOperationSupport;
import com.lhz.demo.pay.WechatPayConfig;
import com.lhz.demo.pay.WechatPayValidator;
import com.lhz.demo.utils.HttpUtils;
import com.wechat.pay.contrib.apache.httpclient.auth.Verifier;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.locks.ReentrantLock;


/**
 * @Author: 
 * @Description:
 **/
@Api(tags = "回调接口(API3)")
@RestController
@Slf4j
public class NotifyController {

    @Resource
    private WechatPayConfig wechatPayConfig;

    @Resource
    private Verifier verifier;

    private final ReentrantLock lock = new ReentrantLock();

    @ApiOperation(value = "支付回调", notes = "支付回调")
    @ApiOperationSupport(order = 5)
    @PostMapping("/payNotify")
    public Map<String, String> payNotify(HttpServletRequest request, HttpServletResponse response) {
        log.info("支付回调");

        // 处理通知参数
        Map<String,Object> bodyMap = getNotifyBody(request);
        if(bodyMap==null){
            return falseMsg(response);
        }

        log.warn("=========== 在对业务数据进行状态检查和处理之前，要采用数据锁进行并发控制，以避免函数重入造成的数据混乱 ===========");
        if(lock.tryLock()) {
            try {
                // 解密resource中的通知数据
                String resource = bodyMap.get("resource").toString();
                Map<String, Object> resourceMap = WechatPayValidator.decryptFromResource(resource, wechatPayConfig.getApiV3Key(),1);
                String orderNo = resourceMap.get("out_trade_no").toString();
                String transactionId = resourceMap.get("transaction_id").toString();

                // TODO 根据订单号，做幂等处理，并且在对业务数据进行状态检查和处理之前，要采用数据锁进行并发控制，以避免函数重入造成的数据混乱
                log.warn("=========== 根据订单号，做幂等处理 ===========");
            } finally {
                //要主动释放锁
                lock.unlock();
            }
        }

        //成功应答
        return trueMsg(response);
    }


    @ApiOperation(value = "退款回调", notes = "退款回调")
    @ApiOperationSupport(order = 5)
    @PostMapping("/refundNotify")
    public Map<String, String> refundNotify(HttpServletRequest request, HttpServletResponse response) {
        log.info("退款回调");

        // 处理通知参数
        Map<String,Object> bodyMap = getNotifyBody(request);
        if(bodyMap==null){
            return falseMsg(response);
        }

        log.warn("=========== 在对业务数据进行状态检查和处理之前，要采用数据锁进行并发控制，以避免函数重入造成的数据混乱 ===========");
        if(lock.tryLock()) {
            try {
                // 解密resource中的通知数据
                String resource = bodyMap.get("resource").toString();
                Map<String, Object> resourceMap = WechatPayValidator.decryptFromResource(resource, wechatPayConfig.getApiV3Key(),2);
                String orderNo = resourceMap.get("out_trade_no").toString();
                String transactionId = resourceMap.get("transaction_id").toString();

                // TODO 根据订单号，做幂等处理，并且在对业务数据进行状态检查和处理之前，要采用数据锁进行并发控制，以避免函数重入造成的数据混乱

                log.warn("=========== 根据订单号，做幂等处理 ===========");
            } finally {
                //要主动释放锁
                lock.unlock();
            }
        }

        //成功应答
        return trueMsg(response);
    }

    private Map<String,Object> getNotifyBody(HttpServletRequest request){
        //处理通知参数
        String body = HttpUtils.readData(request);
        log.info("退款回调参数：{}",body);

        // 转换为Map
        Map<String, Object> bodyMap = JSONObject.parseObject(body, new TypeReference<Map<String, Object>>(){});
        // 微信的通知ID（通知的唯一ID）
        String notifyId = bodyMap.get("id").toString();

        // 验证签名信息
        WechatPayValidator wechatPayValidator
                = new WechatPayValidator(verifier, notifyId, body);
        if(!wechatPayValidator.validate(request)){

            log.error("通知验签失败");
            return null;
        }
        log.info("通知验签成功");
        return bodyMap;
    }

    private Map<String, String> falseMsg(HttpServletResponse response){
        Map<String, String> resMap = new HashMap<>(8);
        //失败应答
        response.setStatus(500);
        resMap.put("code", "ERROR");
        resMap.put("message", "通知验签失败");
        return resMap;
    }

    private Map<String, String> trueMsg(HttpServletResponse response){
        Map<String, String> resMap = new HashMap<>(8);
        //成功应答
        response.setStatus(200);
        resMap.put("code", "SUCCESS");
        resMap.put("message", "成功");
        return resMap;
    }
}

```

![](https://img-blog.csdnimg.cn/3ea3f17b84b6483a84f1f4b462c40930.jpeg#pic_center)