---
layout: post
title:  微信公众平台开发（2）-消息封装
description: 消息包括文本、图片、语音、视频、音乐、图文，消息格式参考<a href='http://mp.weixin.qq.com/wiki/index.php?title=发送被动响应消息'>http://mp.weixin.qq.com/wiki/index.php?title=发送被动响应消息</a> <br>在接入接口时指定了回调URL，在保存时微信会以GET方式调用此URL，并将signature,timestamp,nonce,echostr添加到URL上<br>因此在我们的微信处理的Controller中需要有执行上面URL的GET方法。 
date:   2014-11-26 16:49:39
categories: blog
tags: 微信
---
>消息包括文本、图片、语音、视频、音乐、图文，消息格式参考[http://mp.weixin.qq.com/wiki/index.php?title=发送被动响应消息](http://mp.weixin.qq.com/wiki/index.php?title=发送被动响应消息)   
>在接入接口时指定了回调URL，在保存时微信会以GET方式调用此URL，并将signature,timestamp,nonce,echostr添加到URL上   
>因此在我们的微信处理的Controller中需要有执行上面URL的GET方法。 
  
那么，发送和接收消息怎么来实现呢？   
这个是通过以POST方式调用上述URL来实现的。   
用户发送的数据通过微信包装以后POST到上面的URL，微信包装的各个类型的消息结构如下：   
[http://mp.weixin.qq.com/wiki/index.php?title=接收普通消息](http://mp.weixin.qq.com/wiki/index.php?title=接收普通消息)     
比如， 用户通过微信发送一段文本，那么我们可以从输入流中拿到微信服务器发送的数据。   
比如： 
{% highlight xml %}
<xml>
    <ToUserName><![CDATA[toUser]]></ToUserName>
    <FromUserName><![CDATA[fromUser]]></FromUserName> 
    <CreateTime>1348831860</CreateTime>
    <MsgType><![CDATA[text]]></MsgType>
    <Content><![CDATA[this is a test]]></Content>
    <MsgId>1234567890123456</MsgId>
</xml>
{% endhighlight %} 
 这样，就可以知道用户发送的是什么内容了。   
那么，如果响应用户呢？   
[http://mp.weixin.qq.com/wiki/index.php?title=发送被动响应消息](http://mp.weixin.qq.com/wiki/index.php?title=发送被动响应消息)   
上面定义了响应各个类型消息的结构   
组装响应结构的xml数据，并写到输出流，微信就会解析，并发送给用户了。   
比如回复文本，如：
{% highlight xml %}
<xml>
    <ToUserName><![CDATA[toUser]]></ToUserName>
    <FromUserName><![CDATA[fromUser]]></FromUserName>
    <CreateTime>12345678</CreateTime>
    <MsgType><![CDATA[text]]></MsgType>
    <Content><![CDATA[你好]]></Content>
</xml>
{% endhighlight %}

ToUserName,FromUserName通过接受用户的消息就知道了，那么再组装一下其他内容就可以了。   
首先呢，通过解析输入流可以拿到XML的各个节点内容，然后放到Map中。   
这里使用dom4j来解析，代码参考后面的MessageUtil.parseXml()方法。   
响应时，我们按照文档中给的XML结构组装XML就可以了。   
当然，你用字符串来拼当然可以，但是这样很麻烦。我们可以通过xstream来实现Bean到Xml的转换。   
说到Bean呢，观察一下响应的各个类型的消息的XML结构，发现有一些属性是共有的。   
那么将这些共有的属性组装成一个基类BaseMessage，其他各个类型的消息继承BaseMessage，如下：
{% highlight java %}
import java.io.Serializable;
public class BaseMessage implements Serializable {
    private String ToUserName;
    private String FromUserName;
    private Long CreateTime;
    private String MsgType;
    private Long MsgId;
    /**
    * @return the toUserName
    */
    public String getToUserName() {
        return ToUserName;
    }
    /**
    * @param toUserName the toUserName to set
    */
    public void setToUserName(String toUserName) {
        ToUserName = toUserName;
    }
    /**
    * @return the fromUserName
    */
    public String getFromUserName() {
        return FromUserName;
    }
    /**
    * @param fromUserName the fromUserName to set
    */
    public void setFromUserName(String fromUserName) {
        FromUserName = fromUserName;
    }
    /**
    * @return the createTime
    */
    public Long getCreateTime() {
        return CreateTime;
    }
    /**
    * @param createTime the createTime to set
    */
    public void setCreateTime(Long createTime) {
        CreateTime = createTime;
    }
    /**
    * @return the msgType
    */
    public String getMsgType() {
        return MsgType;
    }
    /**
    * @param msgType the msgType to set
    */
    public void setMsgType(String msgType) {
        MsgType = msgType;
    }
    /**
    * @return the msgId
    */
    public Long getMsgId() {
        return MsgId;
    }
    /**
    * @param msgId the msgId to set
    */
    public void setMsgId(Long msgId) {
        MsgId = msgId;
    }
}
{% endhighlight %}

然后是文本消息：
{% highlight java %}
public class TextMessage extends BaseMessage {
    private String Content;
    /**
    * @return the content
    */
    public String getContent() {
        return Content;
    }

    /**
    * @param content the content to set
    */
    public void setContent(String Content) {
        this.Content = Content;
    }

}
{% endhighlight %}

图片消息：
{% highlight java %}
public class ImageMessage extends BaseMessage {
    /**
    * 通过上传多媒体文件，得到的id。
    */
    private String MediaId;
}
{% endhighlight %}

音乐消息：
{% highlight java %}
public class MusicMessage extends BaseMessage {
    private Music Music;

    /**
    * @return the music
    */
    public Music getMusic() {
        return Music;
    }


    /**
    * @param music the music to set
    */
    public void setMusic(Music music) {
        this.Music = music;
    }

}

public class Music {
    /**
    * 音乐标题
    */
    private String Title;
    /**
    * 音乐描述
    */
    private String Description;
    /**
    * 音乐链接
    */
    private String MusicUrl;
    /**
    * 高质量音乐链接，WIFI环境优先使用该链接播放音乐
    */
    private String HQMusicUrl;
    /**
    * 缩略图的媒体id，通过上传多媒体文件，得到的id
    */
    private String ThumbMediaId;
    /**
    * @return the title
    */
    public String getTitle() {
        return Title;
    }
    /**
    * @param title the title to set
    */
    public void setTitle(String title) {
        Title = title;
    }
    /**
    * @return the description
    */
    public String getDescription() {
        return Description;
    }
    /**
    * @param description the description to set
    */
    public void setDescription(String description) {
        Description = description;
    }
    /**
    * @return the musicUrl
    */
    public String getMusicUrl() {
        return MusicUrl;
    }
    /**
    * @param musicUrl the musicUrl to set
    */
    public void setMusicUrl(String musicUrl) {
        MusicUrl = musicUrl;
    }
    /**
    * @return the hQMusicUrl
    */
    public String getHQMusicUrl() {
        return HQMusicUrl;
    }
    /**
    * @param musicUrl the hQMusicUrl to set
    */
    public void setHQMusicUrl(String musicUrl) {
        HQMusicUrl = musicUrl;
    }
    /**
    * @return the thumbMediaId
    */
    public String getThumbMediaId() {
        return ThumbMediaId;
    }
    /**
    * @param thumbMediaId the thumbMediaId to set
    */
    public void setThumbMediaId(String thumbMediaId) {
        ThumbMediaId = thumbMediaId;
    }
}
{% endhighlight %}

语音消息：
{% highlight java %}
public class VoiceMessage extends BaseMessage {
    /**
    * 通过上传多媒体文件，得到的id
    */
    private String MediaId;


    /**
    * @return the mediaId
    */
    public String getMediaId() {
        return MediaId;
    }


    /**
    * @param mediaId the mediaId to set
    */
    public void setMediaId(String mediaId) {
        MediaId = mediaId;
    }

}
{% endhighlight %}

视频消息：
{% highlight java %}
public class VideoMessage extends BaseMessage {
    /**
    * 通过上传多媒体文件，得到的id
    */
    private String MediaId;
    /**
    * 视频消息的标题
    */
    private String Title;
    /**
    * 视频消息的描述
    */
    private String Description;
    /**
    * @return the mediaId
    */
    public String getMediaId() {
        return MediaId;
    }
    /**
    * @param mediaId the mediaId to set
    */
    public void setMediaId(String mediaId) {
        MediaId = mediaId;
    }
    /**
    * @return the title
    */
    public String getTitle() {
        return Title;
    }
    /**
    * @param title the title to set
    */
    public void setTitle(String title) {
        Title = title;
    }
    /**
    * @return the description
    */
    public String getDescription() {
        return Description;
    }
    /**
    * @param description the description to set
    */
    public void setDescription(String description) {
        Description = description;
    }


}
{% endhighlight %}

图文消息：
{% highlight java %}
public class NewsMessage extends BaseMessage {
    /**
    * 图文消息个数，限制为10条以内
    */
    private int ArticleCount;
    /**
    * 多条图文消息信息，默认第一个item为大图,注意，如果图文数超过10，则将会无响应
    */
    private List<Articles> Articles;
    /**
    * @return the articleCount
    */
    public int getArticleCount() {
        return ArticleCount;
    }
    /**
    * @param articleCount the articleCount to set
    */
    public void setArticleCount(int articleCount) {
        ArticleCount = articleCount;
    }
    /**
    * @return the articles
    */
    public List<Articles> getArticles() {
        return Articles;
    }
    /**
    * @param articles the articles to set
    */
    public void setArticles(List<Articles> articles) {
        Articles = articles;
    }


}


public class Articles implements Serializable {
    /**
    * 图文消息标题
    */
    private String Title;
    /**
    * 图文消息描述
    */
    private String Description;
    /**
    * 图片链接，支持JPG、PNG格式，较好的效果为大图360*200，小图200*200
    */
    private String PicUrl;
    /**
    * 点击图文消息跳转链接
    */
    private String Url;
    /**
    * @return the title
    */
    public String getTitle() {
        return Title;
    }
    /**
    * @param title the title to set
    */
    public void setTitle(String title) {
        Title = title;
    }
    /**
    * @return the description
    */
    public String getDescription() {
        return Description;
    }
    /**
    * @param description the description to set
    */
    public void setDescription(String description) {
        Description = description;
    }
    /**
    * @return the picUrl
    */
    public String getPicUrl() {
        return PicUrl;
    }
    /**
    * @param picUrl the picUrl to set
    */
    public void setPicUrl(String picUrl) {
        PicUrl = picUrl;
    }
    /**
    * @return the url
    */
    public String getUrl() {
        return Url;
    }
    /**
    * @param url the url to set
    */
    public void setUrl(String url) {
        Url = url;
    }


}
{% endhighlight %}

然后将各个类型的消息Bean到XML的转换方法写到一个工具类中。 
因为XML结构中的内容都包含CDATA，因此需要使xstream支持cdata。 
整个微信消息工具类如下：
{% highlight java %}
import java.io.IOException;
import java.io.InputStream;
import java.io.Writer;
import java.util.HashMap;
import java.util.List;
import java.util.Map;


import javax.servlet.http.HttpServletRequest;


import org.apache.log4j.Logger;
import org.dom4j.Document;
import org.dom4j.DocumentException;
import org.dom4j.Element;
import org.dom4j.io.SAXReader;


import com.company.project.model.resp.Articles;
import com.company.project.model.resp.ImageMessage;
import com.company.project.model.resp.MusicMessage;
import com.company.project.model.resp.NewsMessage;
import com.company.project.model.resp.TextMessage;
import com.company.project.model.resp.VideoMessage;
import com.company.project.model.resp.VoiceMessage;
import com.thoughtworks.xstream.XStream;
import com.thoughtworks.xstream.core.util.QuickWriter;
import com.thoughtworks.xstream.io.HierarchicalStreamWriter;
import com.thoughtworks.xstream.io.xml.PrettyPrintWriter;
import com.thoughtworks.xstream.io.xml.XppDriver;


public class MessageUtil {
    public static final Logger log = Logger.getLogger(MessageUtil.class);
    public static final String MESSAGG_TYPE_TEXT = "text";
    public static final String MESSAGG_TYPE_IMAGE = "image";
    public static final String MESSAGG_TYPE_VOICE = "voice";
    public static final String MESSAGG_TYPE_VIDEO = "video";
    public static final String MESSAGG_TYPE_LOCATION = "location";
    public static final String MESSAGG_TYPE_LINK = "link";
    public static final String MESSAGG_TYPE_MUSIC = "music";
    public static final String MESSAGG_TYPE_NEWS = "news";
    public static final String MESSAGG_TYPE_EVENT = "event";
    // 关注
    public static final String EVENT_TYPE_SUBSCRIBE = "subscribe";
    // 取消关注
    public static final String EVENT_TYPE_UNSUBSCRIBE = "unsubscribe";

    /**
    * 扩展XStream，使其支持CDATA
    */
    private static XStream xStream = new XStream(new XppDriver() {
        public HierarchicalStreamWriter createWriter(Writer out) {
            return new PrettyPrintWriter(out) {
                // 对所有的XML节点增加CDATA标记
                boolean cdata = true;

                public void startNode(String name,Class clazz) {
                    super.startNode(name,clazz);
                }

                protected void writeText(QuickWriter writer,String text) {
                    if (cdata) {
                        writer.write("<![CDATA[");
                        writer.write(text);
                        writer.write("]]>");
                    }
                    else {
                        writer.write(text);
                    }
                }
            };
        }
    });
    /**
    * 解析微信发过来的请求（XML）
    *
    * @author qincd
    * @date Nov 4, 2014 11:03:55 AM
    */
    public static Map<String,String> parseXml(HttpServletRequest request) throws IOException, DocumentException {
        Map<String,String> map = new HashMap<String,String>();
        InputStream input = request.getInputStream();
        SAXReader saxReader = new SAXReader();
        Document document = saxReader.read(input);
        Element root = document.getRootElement();
        List<Element> eles = root.elements();
        for (Element ele:eles) {
            log.info(ele.getName() + ":" + ele.getText());
            map.put(ele.getName(), ele.getText());
        }

        input.close();
        input = null;

        return map;
    }

    public static String textMessageToXml(TextMessage textMessage) {
        xStream.alias("xml", textMessage.getClass());
        return xStream.toXML(textMessage);
    }

    public static String ImageMessageToXml(ImageMessage imageMessage) {
        xStream.alias("xml", imageMessage.getClass());
        return xStream.toXML(imageMessage);
    }

    public static String MusicMessageToXml(MusicMessage musicMessage) {
        xStream.alias("xml", musicMessage.getClass());
        return xStream.toXML(musicMessage);
    }
    public static String NewsMessageToXml(NewsMessage newsMessage) {
        xStream.alias("xml", newsMessage.getClass());
        xStream.alias("item", new Articles().getClass());
        return xStream.toXML(newsMessage);
    }
    public static String videoMessageToXml(VideoMessage videoMessage) {
        xStream.alias("xml", videoMessage.getClass());
        return xStream.toXML(videoMessage);
    }
    public static String voiceMessageToXml(VoiceMessage voiceMessage) {
        xStream.alias("xml", voiceMessage.getClass());
        return xStream.toXML(voiceMessage);
    }
}
{% endhighlight %}

这些公共的东西可以打成一个Jar包