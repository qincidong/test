---
layout: post
title:  微信公众平台开发（3）-回复消息
description: 一、回复文本消息<br>二、回复链接消息<br>三、回复音乐消息<br>四、回复图文消息<br>五、事件-订阅<br>六、事件-取消订阅<br>PS：当然还包括表情。参考：<a href='http://www.360doc.com/content/13/0803/13/13350344_304465190.shtml'>http://www.360doc.com/content/13/0803/13/13350344_304465190.shtml</a>
date:   2014-11-25 16:49:39
categories: blog
tags: 微信
---
>一、回复文本消息  
>二、回复链接消息  
>三、回复音乐消息   
>四、回复图文消息  
>五、事件-订阅  
>六、事件-取消订阅  
>PS：当然还包括表情。参考：[http://www.360doc.com/content/13/0803/13/13350344_304465190.shtml](http://www.360doc.com/content/13/0803/13/13350344_304465190.shtml)  

完整代码：  
微信接口配置的回调地址对应的Controller:
{% highlight java %}
import org.apache.commons.lang.StringUtils;  
import org.apache.log4j.Logger;  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.stereotype.Controller;  
import org.springframework.web.bind.annotation.RequestMapping;  
import org.springframework.web.bind.annotation.RequestMethod;  
  
  
import com.company.project.service.WeixinService;  
import com.company.project.util.Util;  
  
  
import javacommon.base.BaseRestSpringController;  
  
  
@Controller  
@RequestMapping("/wxapi")  
public class WeixinApiController extends BaseRestSpringController<Object, java.lang.Long>{  
    public static final Logger log = Logger.getLogger(WeixinApiController.class);  
    public static final String WX_TOKEN = "weixin";  
    @Autowired  
    private WeixinService wxService;  
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
          
        if (StringUtils.isEmpty(signature)) {  
            return;  
        }  
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
      
    @RequestMapping(method=RequestMethod.POST)  
    public void doPost(HttpServletRequest request,HttpServletResponse response) throws IOException {  
        // 处理请求、响应  
        request.setCharacterEncoding("utf-8");  
        response.setCharacterEncoding("utf-8");  
        String message = wxService.processRequest(request);  
        response.getWriter().println(message);  
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
      
      
    import java.util.ArrayList;  
    import java.util.List;  
    import java.util.Map;  
      
      
    import javax.servlet.http.HttpServletRequest;  
      
      
    import org.apache.log4j.Logger;  
    import org.springframework.stereotype.Service;  
      
      
    import com.company.project.model.resp.Articles;  
    import com.company.project.model.resp.Music;  
    import com.company.project.model.resp.MusicMessage;  
    import com.company.project.model.resp.NewsMessage;  
    import com.company.project.model.resp.TextMessage;  
    import com.company.project.util.MessageUtil;  
      
      
    @Service  
    public class WeixinService {  
    public static Logger log = Logger.getLogger(WeixinService.class);  
      
    public String processRequest(HttpServletRequest req) {  
    // 解析微信传递的参数  
    String str = null;  
    try {  
        Map<String,String> xmlMap = MessageUtil.parseXml(req);  
        str = "请求处理异常，请稍后再试！";  
          
        String ToUserName = xmlMap.get("ToUserName");  
        String FromUserName = xmlMap.get("FromUserName");  
        String MsgType = xmlMap.get("MsgType");  
          
        if (MsgType.equals(MessageUtil.MESSAGG_TYPE_TEXT)) {  
            // 用户发送的文本消息  
            String content = xmlMap.get("Content");  
            log.info("用户：[" + FromUserName + "]发送的文本消息：" + content);  
              
            // 链接  
            if (content.contains("csdn")) {  
                TextMessage tm = new TextMessage();  
                tm.setToUserName(FromUserName);  
                tm.setFromUserName(ToUserName);  
                tm.setMsgType(MessageUtil.MESSAGG_TYPE_TEXT);  
                tm.setCreateTime(System.currentTimeMillis());  
                tm.setContent("我的CSDN博客：<a href=\"http://my.csdn.net/qincidong\">我的CSDN博客</a>\n");  
                return MessageUtil.textMessageToXml(tm);  
            }  
          
            if (content.contains("图文")) {  
                NewsMessage nm = new NewsMessage();  
                nm.setFromUserName(ToUserName);  
                nm.setToUserName(FromUserName);  
                nm.setCreateTime(System.currentTimeMillis());  
                nm.setMsgType(MessageUtil.MESSAGG_TYPE_NEWS);  
                List<Articles> articles = new ArrayList<Articles>();  
                Articles e1 = new Articles();  
                e1.setTitle("马云接受外媒专访：中国的五大银行想杀了“我”");  
                e1.setDescription("阿里巴巴集团上市大获成功，《华尔街日报》日前就阿里巴巴集团、支付宝等话题采访了马云，马云也谈到了与苹果Apple Pay建立电子支付联盟的可能性。本文摘编自《华尔街日报》，原文标题：马云谈阿里巴巴将如何帮助美国出口商，虎嗅略有删节。");  
                e1.setPicUrl("http://img1.gtimg.com/finance/pics/hv1/29/53/1739/113092019.jpg");  
                e1.setUrl("http://finance.qq.com/a/20141105/010616.htm?pgv_ref=aio2012&ptlang=2052");  
                  
                Articles e2 = new Articles();  
                e2.setTitle("史上最牛登机牌：姓名竟是微博名 涉事航空公司公开致歉");  
                e2.setDescription("世上最遥远的距离是飞机在等你登机，你却过不了安检。");  
                e2.setPicUrl("http://p9.qhimg.com/dmfd/328_164_100/t011946ff676981792d.png");  
                e2.setUrl("http://www.techweb.com.cn/column/2014-11-05/2093128.shtml");  
                articles.add(e1);  
                articles.add(e2);  
                  
                nm.setArticles(articles);  
                nm.setArticleCount(articles.size());  
                  
                String newsXml = MessageUtil.NewsMessageToXml(nm);  
                log.info("\n"+newsXml);  
                return newsXml;  
            }  
            if (content.contains("音乐")) {  
                MusicMessage mm =  new MusicMessage();  
                mm.setFromUserName(ToUserName);  
                mm.setToUserName(FromUserName);  
                mm.setMsgType(MessageUtil.MESSAGG_TYPE_MUSIC);  
                mm.setCreateTime(System.currentTimeMillis());  
                Music music = new Music();  
                music.setTitle("Maid with the Flaxen Hair");  
                music.setDescription("测试音乐");  
                music.setMusicUrl("http://yinyueshiting.baidu.com/data2/music/123297915/1201250291415073661128.mp3?xcode=e2edf18bbe9e452655284217cdb920a7a6a03c85c06f4409");  
                music.setHQMusicUrl("http://yinyueshiting.baidu.com/data2/music/123297915/1201250291415073661128.mp3?xcode=e2edf18bbe9e452655284217cdb920a7a6a03c85c06f4409");  
                mm.setMusic(music);  
                  
                String musicXml = MessageUtil.MusicMessageToXml(mm);  
                log.info("musicXml:\n" + musicXml);  
                return musicXml;  
            }  
          
            // 响应  
            TextMessage tm = new TextMessage();  
            tm.setToUserName(FromUserName);  
            tm.setFromUserName(ToUserName);  
            tm.setMsgType(MessageUtil.MESSAGG_TYPE_TEXT);  
            tm.setCreateTime(System.currentTimeMillis());  
            tm.setContent("你好，你发送的内容是：\n" + content);  
              
            String xml = MessageUtil.textMessageToXml(tm);  
            log.info("xml:" + xml);  
            return xml;  
        }  
        else if (MsgType.equals(MessageUtil.MESSAGG_TYPE_EVENT)) {  
            String event = xmlMap.get("Event");  
            if (event.equals(MessageUtil.EVENT_TYPE_SUBSCRIBE)) {  
                // 订阅  
                TextMessage tm = new TextMessage();  
                tm.setToUserName(FromUserName);  
                tm.setFromUserName(ToUserName);  
                tm.setMsgType(MessageUtil.MESSAGG_TYPE_TEXT);  
                tm.setCreateTime(System.currentTimeMillis());  
                tm.setContent("你好，欢迎关注[程序员的生活]公众号！[愉快]/呲牙/玫瑰\n目前可以回复文本消息");  
                return MessageUtil.textMessageToXml(tm);  
            }  
            else if (event.equals(MessageUtil.EVENT_TYPE_UNSUBSCRIBE)) {  
                // 取消订阅  
                log.info("用户【" + FromUserName + "]取消关注了。");  
            }  
        }  
    } catch (Exception e) {  
        e.printStackTrace();  
        log.error("处理微信请求时发生异常：");  
    }  
      
    return str;  
    }  
}
{% endhighlight %}


 