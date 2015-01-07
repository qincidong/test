---
layout: post
title:  微信公众平台开发（4）-自定义菜单
description: 参考了《微信公众账号开发教程(java).doc》 <br>文档：<a href='http://mp.weixin.qq.com/wiki/index.php?title=自定义菜单创建接口'>http://mp.weixin.qq.com/wiki/index.php?title=自定义菜单创建接口</a> <br>其实就是以POST方式调用 <a href='https://api.weixin.qq.com/cgi-bin/menu/create?access_token=ACCESS_TOKEN'>https://api.weixin.qq.com/cgi-bin/menu/create?access_token=ACCESS_TOKEN</a>，将菜单数据以微信公众平台定义的格式写到输出流中。 
date:   2014-11-26 16:49:39
categories: blog
tags: 微信
---
>参考了《微信公众账号开发教程(java).doc》   
>文档：[http://mp.weixin.qq.com/wiki/index.php?title=自定义菜单创建接口](http://mp.weixin.qq.com/wiki/index.php?title=自定义菜单创建接口)   
>其实就是以POST方式调用[https://api.weixin.qq.com/cgi-bin/menu/create?access_token=ACCESS_TOKEN](https://api.weixin.qq.com/cgi-bin/menu/create?access_token=ACCESS_TOKEN)，将菜单数据以微信公众平台定义的格式写到输出流中。   
>链接中的access_token从哪儿来？   
>参考[http://mp.weixin.qq.com/wiki/index.php?title=获取access_token](http://mp.weixin.qq.com/wiki/index.php?title=获取access_token)   
>不论是获取access_token，还是创建菜单，都需要使用https方式调用提供的URL。   
>将获取access_token，创建菜单等动作封装到一个工具类中。 

如下：
{% highlight java %}
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.OutputStream;
import java.net.MalformedURLException;
import java.net.URL;
import java.security.KeyManagementException;
import java.security.NoSuchAlgorithmException;
import java.security.NoSuchProviderException;
import java.security.SecureRandom;
import java.util.Calendar;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;

import javax.net.ssl.HttpsURLConnection;
import javax.net.ssl.SSLContext;
import javax.net.ssl.SSLSocketFactory;
import javax.net.ssl.TrustManager;

import net.sf.json.JSONObject;

import org.apache.commons.lang.StringUtils;
import org.apache.log4j.Logger;

import com.company.project.model.menu.AccessToken;
import com.company.project.model.menu.Menu;

public class WeixinUtil {
    private static Logger log = Logger.getLogger(WeixinUtil.class);
    public final static String APPID = "****************";
    public final static String APP_SECRET = "************************";
    // 获取access_token的接口地址（GET） 限200（次/天）
    public final static String access_token_url = "https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=APPID&secret=APPSECRET";
    // 创建菜单
    public final static String create_menu_url = "https://api.weixin.qq.com/cgi-bin/menu/create?access_token=ACCESS_TOKEN";
    // 存放：1.token，2：获取token的时间,3.过期时间
    public final static Map<String,Object> accessTokenMap = new HashMap<String,Object>();
    /**
     * 发起https请求并获取结果
     * 
     * @param requestUrl 请求地址
     * @param requestMethod 请求方式（GET、POST）
     * @param outputStr 提交的数据
     * @return JSONObject(通过JSONObject.get(key)的方式获取json对象的属性值)
     */
    public static JSONObject handleRequest(String requestUrl,String requestMethod,String outputStr) {
        JSONObject jsonObject = null;
        
        try {
            URL url = new URL(requestUrl);
            HttpsURLConnection conn = (HttpsURLConnection) url.openConnection();
            SSLContext ctx = SSLContext.getInstance("SSL", "SunJSSE");
            TrustManager[] tm = {new MyX509TrustManager()};
            ctx.init(null, tm, new SecureRandom());
            SSLSocketFactory sf = ctx.getSocketFactory();
            conn.setSSLSocketFactory(sf);
            conn.setDoInput(true);
            conn.setDoOutput(true);
            conn.setRequestMethod(requestMethod);
            conn.setUseCaches(false);
            
            if ("GET".equalsIgnoreCase(requestMethod)) {
                conn.connect();
            }
            
            if (StringUtils.isNotEmpty(outputStr)) {
                OutputStream out = conn.getOutputStream();
                out.write(outputStr.getBytes("utf-8"));
                out.close();
            }
            
            InputStream in = conn.getInputStream();
            BufferedReader br = new BufferedReader(new InputStreamReader(in,"utf-8"));
            StringBuffer buffer = new StringBuffer();
            String line = null;
            
            while ((line = br.readLine()) != null) {
                buffer.append(line);
            }
            
            in.close();
            conn.disconnect();
            
            jsonObject = JSONObject.fromObject(buffer.toString());
        } catch (MalformedURLException e) {
            e.printStackTrace();
            log.error("URL错误！");
        } catch (IOException e) {
            e.printStackTrace();
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
        } catch (NoSuchProviderException e) {
            e.printStackTrace();
        } catch (KeyManagementException e) {
            e.printStackTrace();
        }
        return jsonObject;
    }
    
    /**
     * 获取access_token
     *
     * @author qincd
     * @date Nov 6, 2014 9:56:43 AM
     */
    public static AccessToken getAccessToken(String appid,String appSecret) {
        AccessToken at = new AccessToken();
        // 每次获取access_token时，先从accessTokenMap获取，如果过期了就重新从微信获取
        // 没有过期直接返回
        // 从微信获取的token的有效期为2个小时
        if (!accessTokenMap.isEmpty()) {
            Date getTokenTime = (Date) accessTokenMap.get("getTokenTime");
            Calendar c = Calendar.getInstance();
            c.setTime(getTokenTime);
            c.add(Calendar.HOUR_OF_DAY, 2);
            
            getTokenTime = c.getTime();
            if (getTokenTime.after(new Date())) {
                log.info("缓存中发现token未过期，直接从缓存中获取access_token");
                // token未过期，直接从缓存获取返回
                String token = (String) accessTokenMap.get("token");
                Integer expire = (Integer) accessTokenMap.get("expire");
                at.setToken(token);
                at.setExpiresIn(expire);
                return at;
            }
        }
        String requestUrl = access_token_url.replace("APPID", appid).replace("APPSECRET", appSecret);
        
        JSONObject object = handleRequest(requestUrl, "GET", null);
        String access_token = object.getString("access_token");
        int expires_in = object.getInt("expires_in");
        
        log.info("\naccess_token:" + access_token);
        log.info("\nexpires_in:" + expires_in);
        
        at.setToken(access_token);
        at.setExpiresIn(expires_in);
        
        // 每次获取access_token后，存入accessTokenMap
        // 下次获取时，如果没有过期直接从accessTokenMap取。
        accessTokenMap.put("getTokenTime", new Date());
        accessTokenMap.put("token", access_token);
        accessTokenMap.put("expire", expires_in);
        
        return at;
    }
    
    /**
     * 创建菜单
     *
     * @author qincd
     * @date Nov 6, 2014 9:56:36 AM
     */
    public static boolean createMenu(Menu menu,String accessToken) {
        String requestUrl = create_menu_url.replace("ACCESS_TOKEN", accessToken);
        String menuJsonString = JSONObject.fromObject(menu).toString();
        JSONObject jsonObject = handleRequest(requestUrl, "POST", menuJsonString);
        String errorCode = jsonObject.getString("errcode");
        if (!"0".equals(errorCode)) {
            log.error(String.format("菜单创建失败！errorCode:%d,errorMsg:%s",jsonObject.getInt("errcode"),jsonObject.getString("errmsg")));
            return false;
        }
        
        log.info("菜单创建成功！");
        
        return true;
    }
}

public class AccessToken {
    // 获取到的凭证
    private String token;
    // 凭证有效时间，单位：秒
    private int expiresIn;

    public String getToken() {
        return token;
    }

    public void setToken(String token) {
        this.token = token;
    }

    public int getExpiresIn() {
        return expiresIn;
    }

    public void setExpiresIn(int expiresIn) {
        this.expiresIn = expiresIn;
    }
}

import java.security.cert.CertificateException;
import java.security.cert.X509Certificate;

import javax.net.ssl.X509TrustManager;

public class MyX509TrustManager implements X509TrustManager{

    @Override
    public void checkClientTrusted(X509Certificate[] arg0, String arg1)
            throws CertificateException {
        
    }

    @Override
    public void checkServerTrusted(X509Certificate[] arg0, String arg1)
            throws CertificateException {
        
    }

    @Override
    public X509Certificate[] getAcceptedIssuers() {
        return null;
    }

}
{% endhighlight %}

菜单包含一级菜单和二级菜单，菜单类型又包括点击菜单直接跳转到一个链接，或者是点击菜单触发一个事件，在事件处理中自定义逻辑。   
不论是哪种菜单，都包含菜单名字，封装到Button类中。   
对于点击后直接跳转到某个链接的菜单定义为ViewButton。   
对于点击后触发一个事件的菜单定义为CommandButton   
一级菜单可以包含二级菜单，定义为ComplexButton   
对于整个菜单来讲，可以包含多个一级菜单，定义为Menu。   
各个类的结构如下：
{% highlight java %}
public class Button {
    private String name;

    /**
     * @return the name
     */
    public String getName() {
        return name;
    }

    /**
     * @param name the name to set
     */
    public void setName(String name) {
        this.name = name;
    }
    
}

public class CommandButton extends Button{
    private String type;
    private String key;
    /**
     * @return the type
     */
    public String getType() {
        return type;
    }
    /**
     * @param type the type to set
     */
    public void setType(String type) {
        this.type = type;
    }
    /**
     * @return the key
     */
    public String getKey() {
        return key;
    }
    /**
     * @param key the key to set
     */
    public void setKey(String key) {
        this.key = key;
    }
    
    
}

public class ViewButton extends Button {
    private String type;
    private String url;
    /**
     * @return the type
     */
    public String getType() {
        return type;
    }
    /**
     * @param type the type to set
     */
    public void setType(String type) {
        this.type = type;
    }
    /**
     * @return the url
     */
    public String getUrl() {
        return url;
    }
    /**
     * @param url the url to set
     */
    public void setUrl(String url) {
        this.url = url;
    }
    
    
}
public class ComplexButton extends Button{
    private Button[] sub_button;

    /**
     * @return the sub_button
     */
    public Button[] getSub_button() {
        return sub_button;
    }

    /**
     * @param sub_button the sub_button to set
     */
    public void setSub_button(Button[] sub_button) {
        this.sub_button = sub_button;
    }
    
    
}

public class Menu {
    private Button[] button;

    /**
     * @return the button
     */
    public Button[] getButton() {
        return button;
    }

    /**
     * @param button the button to set
     */
    public void setButton(Button[] button) {
        this.button = button;
    }
    
    
}
{% endhighlight %}

菜单创建测试代码：
{% highlight java %}
public class WeixinUtilTest {

    /**
     *
     * @author qincd
     * @date Nov 6, 2014 9:57:54 AM
     */
    public static void main(String[] args) {
        // 1).获取access_token
        AccessToken accessToken = WeixinUtil.getAccessToken(WeixinUtil.APPID, WeixinUtil.APP_SECRET);
        // 2).创建菜单
        Menu menu = new Menu();
        
        // 菜单1
        ComplexButton cb0 = new ComplexButton();
        cb0.setName("超值预定");
        
        ViewButton cb01 = new ViewButton();
        cb01.setName("团购订单");
        cb01.setType("view");
        cb01.setUrl("http://www.meituan.com");
        
        ViewButton cb02 = new ViewButton();
        cb02.setName("微信团购");
        cb02.setType("view");
        cb02.setUrl("http://www.weixin.com");
        
        cb0.setSub_button(new ViewButton[]{cb01,cb02});
        
        // 菜单2
        ComplexButton cb1 = new ComplexButton();
        cb1.setName("我的服务");
        
        ViewButton cb11 = new ViewButton();
        cb11.setName("办登机牌");
        cb11.setType("view");
        cb11.setUrl("http://www.meituan.com");
        
        ViewButton cb12 = new ViewButton();
        cb12.setName("航班动态");
        cb12.setType("view");
        cb12.setUrl("http://www.meituan.com");
        
        ViewButton cb13 = new ViewButton();
        cb13.setName("里程查询");
        cb13.setType("view");
        cb13.setUrl("http://www.meituan.com");
        
        cb1.setSub_button(new ViewButton[]{cb11,cb12,cb13});
        
        // 菜单3
        ComplexButton cb2 = new ComplexButton();
        cb2.setName("我的测试");
        
        CommandButton cb21 = new CommandButton();
        cb21.setName("回复文字");
        cb21.setType("click");
        cb21.setKey("reply_words");
        
        CommandButton cb22 = new CommandButton();
        cb22.setName("回复音乐");
        cb22.setType("click");
        cb22.setKey("reply_music");
        
        CommandButton cb23 = new CommandButton();
        cb23.setName("回复图文");
        cb23.setType("click");
        cb23.setKey("reply_news");
        
        CommandButton cb24 = new CommandButton();
        cb24.setName("回复链接");
        cb24.setType("click");
        cb24.setKey("reply_link");
        
        cb2.setSub_button(new CommandButton[]{cb21,cb22,cb23,cb24});
        
        menu.setButton(new ComplexButton[]{cb0,cb1,cb2});
        String menuJsonString = JSONObject.fromObject(menu).toString();
        System.out.println(menuJsonString);
        WeixinUtil.createMenu(menu, accessToken.getToken());
    }

}
{% endhighlight %}

上面的代码创建了3个一级菜单，每个一级菜单都包含有二级菜单。   
前2个一级菜单都是点击菜单后直接跳转到一个连接，第3个一级菜单点击后触发click事件   
在后来根据菜单的key来处理。   
如下：
{% highlight java %}
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
                else if (event.equals(MessageUtil.EVENT_TYPE_CLICK)) {  
                    String eventKey = xmlMap.get("EventKey");  
                    if (eventKey.equals("reply_words")) { // 点击了回复文字菜单  
                        TextMessage tm = new TextMessage();  
                        tm.setToUserName(FromUserName);  
                        tm.setFromUserName(ToUserName);  
                        tm.setMsgType(MessageUtil.MESSAGG_TYPE_TEXT);  
                        tm.setCreateTime(System.currentTimeMillis());  
                        tm.setContent("你好，你点击了回复文本菜单：\n" );  
                          
                        String xml = MessageUtil.textMessageToXml(tm);  
                        log.info("xml:" + xml);  
                        return xml;  
                    }  
                    else if (eventKey.equals("reply_music")) { // 点击了回复音乐  
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
                    else if (eventKey.equals("reply_news")) { // 点击了回复图文  
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
                    else if (eventKey.equals("reply_link")) {  
                        TextMessage tm = new TextMessage();  
                        tm.setToUserName(FromUserName);  
                        tm.setFromUserName(ToUserName);  
                        tm.setMsgType(MessageUtil.MESSAGG_TYPE_TEXT);  
                        tm.setCreateTime(System.currentTimeMillis());  
                        tm.setContent("我的CSDN博客：<a href=\"http://my.csdn.net/qincidong\">我的CSDN博客</a>\n");  
                        return MessageUtil.textMessageToXml(tm);  
                    }  
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

![preview](http://images.cnitblog.com/blog/548748/201411/261356183097401.png)