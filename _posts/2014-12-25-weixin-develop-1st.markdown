---
layout: post
title:  微信公众平台开发（1）-接入指南
description: 微信公众平台开发（1）-接入指南
date:   2014-11-25 16:49:39
categories: blog
tags: 微信
---
##接入指南
###第一步：申请消息接口
登录[https://mp.weixin.qq.com/](https://mp.weixin.qq.com/) 后，在公众平台后台管理页面 – 开发者中心页，点击“修改配置”按钮，填写URL、Token和EncodingAESKey，  
其中URL是开发者用来接收微信服务器数据的接口URL。Token可由开发者可以任意填写，用作生成签名（该Token会和接口URL中包含的Token进行比对，从而验证安全性）。  
EncodingAESKey由开发者手动填写或随机生成，将用作消息体加解密密钥。同时，开发者可选择消息加解密方式：明文模式、兼容模式和安全模式。  
模式的选择与服务器配置在提交后都会立即生效，请开发者谨慎填写及选择。加解密方式的默认状态为明文模式，  
选择兼容模式和安全模式需要提前配置好相关加解密代码，详情请参考消息体签名及加解密部分的文档。   

###第二步：验证URL有效性
开发者提交信息后，微信服务器将发送GET请求到填写的URL上，GET请求携带四个参数：  
参数 描述  
signature  微信加密签名，signature结合了开发者填写的token参数和请求中的timestamp参数、nonce参数。  
timestamp  时间戳  
nonce  随机数  
echostr  随机字符串  
开发者通过检验signature对请求进行校验（下面有校验方式）。若确认此次GET请求来自微信服务器，请原样返回echostr参数内容，则接入生效，成为开发者成功，否则接入失败。  

加密/校验流程如下：   
1. 将token、timestamp、nonce三个参数进行字典序排序  
2. 将三个参数字符串拼接成一个字符串进行sha1加密  
3. 开发者获得加密后的字符串可与signature对比，标识该请求来源于微信  

检验signature的JAVA示例代码：
{% highlight java %}
import java.io.IOException;
import java.util.Arrays;
 
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
 
import org.apache.commons.lang.StringUtils;
import org.apache.log4j.Logger;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
 
import com.company.project.util.Util;
 
import javacommon.base.BaseRestSpringController;
 
@Controller
@RequestMapping("/wxapi")
public class WeixinApiController extends BaseRestSpringController<Object, java.lang.Long>{
public static final Logger log = Logger.getLogger(WeixinApiController.class);
public static final String WX_TOKEN = "weixin";
/**
 * 微信回调地址
 *
 * @author qincd
 * @throws IOException 
 * @date Nov 3, 2014 4:01:42 PM
 */
@RequestMapping(method=RequestMethod.GET)
public void doGet(HttpServletRequest request,HttpServletResponse response) throws IOException {
// 微信会在配置的回调地址上加上signature,nonce,timestamp,echostr4个参数
String signature = request.getParameter("signature");
String timestamp = request.getParameter("timestamp");
String nonce = request.getParameter("nonce");
String echostr = request.getParameter("echostr");
log.info("微信传递的参数：");
log.info("signature："+signature);
log.info("timestamp："+timestamp);
log.info("nonce："+nonce);
log.info("echostr："+echostr);
// 1).排序
String sortString = sort(WX_TOKEN, timestamp, nonce);
// 2).加密
String mytoken = Util.sha1(sortString);
// 3).校验签名
if (StringUtils.isNotEmpty(mytoken) && mytoken.equals(signature)) {
log.info("签名校验通过。");
response.getWriter().println(echostr);
}
else {
log.warn("签名校验失败。");
}
}
/**
 * 将token,timestamp,nonce按字典序排序，并返回拼接的字符串
 *
 * @author qincd
 * @date Nov 3, 2014 4:09:43 PM
 */
public static String sort(String token,String timestamp,String nonce) {
String[] strArray = {token,timestamp,nonce};
Arrays.sort(strArray);
StringBuilder sbuilder = new StringBuilder();
for (String str : strArray) {
sbuilder.append(str);
}
return sbuilder.toString();
}
}
 
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
 
import org.apache.log4j.Logger;
 
public class Util {
private static Logger log = Logger.getLogger(Util.class);
 
/**
 * sha1加密
 *
 * @author qincd
 * @date Nov 3, 2014 4:16:39 PM
 */
public static String sha1(String str) {
if (str == null || str.length() == 0) return "";
try {
MessageDigest md = MessageDigest.getInstance("SHA1");
byte[] bytes = md.digest(str.getBytes());
return byte2Hex(bytes);
} catch (NoSuchAlgorithmException e) {
e.printStackTrace();
log.error("SHA1加密出错：" + e.getMessage());
throw new RuntimeException("SHA1加密出错");
}
}
public static String byte2Hex(byte[] data) {
if (data == null || data.length == 0) return "";
StringBuilder sbu = new StringBuilder();
for (int i=0;i<data.length;i++) {
if ((data[i] & 0xff) < 0x10) {
sbu.append("0");
}
sbu.append(Integer.toHexString(data[i] & 0xff));
}
return sbu.toString();
}
}
{% endhighlight %}

###第三步：成为开发者
验证URL有效性成功后即接入生效，成为开发者。  如果公众号类型为服务号（订阅号只能使用普通消息接口），可以在公众平台网站中申请认证，认证成功的服务号将获得众多接口权限，以满足开发者需求。  
此后用户每次向公众号发送消息、或者产生自定义菜单点击事件时，响应URL将得到推送。  
公众号调用各接口时，一般会获得正确的结果，具体结果可见对应接口的说明。返回错误时，可根据返回码来查询错误原因。全局返回码说明  
用户向公众号发送消息时，公众号方收到的消息发送者是一个OpenID，是使用用户微信号加密后的结果，每个用户对每个公众号有一个唯一的OpenID。  

此外请注意，微信公众号接口只支持80接口。