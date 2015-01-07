---
layout: post
title:  微信公众平台开发（5）-上传下载多媒体文件
description: 回复图片、音频、视频消息都是需要media_id的，这个是需要将多媒体文件上传到微信服务器才有的。<br>将多媒体文件上传到微信服务器，以及从微信服务器下载文件，可以参考：<a href='http://mp.weixin.qq.com/wiki/index.php?title=上传下载多媒体文件'>http://mp.weixin.qq.com/wiki/index.php?title=上传下载多媒体文件</a>
date:   2014-11-25 16:49:39
categories: blog
tags: 微信
---
>回复图片、音频、视频消息都是需要media_id的，这个是需要将多媒体文件上传到微信服务器才有的。  
>将多媒体文件上传到微信服务器，以及从微信服务器下载文件，可以参考：[http://mp.weixin.qq.com/wiki/index.php?title=上传下载多媒体文件](http://mp.weixin.qq.com/wiki/index.php?title=上传下载多媒体文件)  
>上传下载多媒体文件的方法还是写到WeixinUtil.java中。  

代码如下：
{% highlight java %}
import java.io.BufferedOutputStream;  
import java.io.BufferedReader;  
import java.io.DataInputStream;  
import java.io.DataOutputStream;  
import java.io.File;  
import java.io.FileInputStream;  
import java.io.FileOutputStream;  
import java.io.IOException;  
import java.io.InputStream;  
import java.io.InputStreamReader;  
import java.io.OutputStream;  
import java.net.HttpURLConnection;  
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
    public final static String APPID = "wxb927d4280e6db674";  
    public final static String APP_SECRET = "21441e9f3226eee81e14380a768b6d1e";  
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
      
    // 上传多媒体文件到微信服务器  
    public static final String upload_media_url = "http://file.api.weixin.qq.com/cgi-bin/media/upload?access_token=ACCESS_TOKEN&type=TYPE";  
    /** 
     * 上传多媒体文件到微信服务器<br> 
     * @see http://www.oschina.net/code/snippet_1029535_23824 
     * @see http://mp.weixin.qq.com/wiki/index.php?title=上传下载多媒体文件 
     * 
     * @author qincd 
     * @date Nov 6, 2014 4:11:22 PM 
     */  
    public static JSONObject uploadMediaToWX(String filePath,String type,String accessToken) throws IOException{  
        File file = new File(filePath);  
        if (!file.exists()) {  
            log.error("文件不存在！");  
            return null;  
        }  
          
        String url = upload_media_url.replace("ACCESS_TOKEN", accessToken).replace("TYPE", type);  
        URL urlObj = new URL(url);  
        HttpURLConnection conn = (HttpURLConnection) urlObj.openConnection();  
        conn.setDoInput(true);  
        conn.setDoOutput(true);  
        conn.setUseCaches(false);  
          
        conn.setRequestProperty("Connection", "Keep-Alive");  
        conn.setRequestProperty("Charset", "UTF-8");  
   
        // 设置边界  
        String BOUNDARY = "----------" + System.currentTimeMillis();  
        conn.setRequestProperty("Content-Type", "multipart/form-data; boundary="  
                + BOUNDARY);  
   
        // 请求正文信息  
   
        // 第一部分：  
        StringBuilder sb = new StringBuilder();  
        sb.append("--"); // ////////必须多两道线  
        sb.append(BOUNDARY);  
        sb.append("\r\n");  
        sb.append("Content-Disposition: form-data;name=\"file\";filename=\""  
                + file.getName() + "\"\r\n");  
        sb.append("Content-Type:application/octet-stream\r\n\r\n");  
   
        byte[] head = sb.toString().getBytes("utf-8");  
   
        // 获得输出流  
        OutputStream out = new DataOutputStream(conn.getOutputStream());  
        out.write(head);  
   
        // 文件正文部分  
        DataInputStream in = new DataInputStream(new FileInputStream(file));  
        int bytes = 0;  
        byte[] bufferOut = new byte[1024];  
        while ((bytes = in.read(bufferOut)) != -1) {  
            out.write(bufferOut, 0, bytes);  
        }  
        in.close();  
   
        // 结尾部分  
        byte[] foot = ("\r\n--" + BOUNDARY + "--\r\n").getBytes("utf-8");// 定义最后数据分隔线  
   
        out.write(foot);  
   
        out.flush();  
        out.close();  
   
        /** 
         * 读取服务器响应，必须读取,否则提交不成功 
         */  
         try {  
             // 定义BufferedReader输入流来读取URL的响应  
             StringBuffer buffer = new StringBuffer();  
             BufferedReader reader = new BufferedReader(new InputStreamReader(  
             conn.getInputStream()));  
             String line = null;  
             while ((line = reader.readLine()) != null) {  
                 buffer.append(line);  
             }  
               
             reader.close();  
             conn.disconnect();  
               
             return JSONObject.fromObject(buffer.toString());  
         } catch (Exception e) {  
             log.error("发送POST请求出现异常！" + e);  
             e.printStackTrace();  
         }  
        return null;  
    }  
      
    public static final String download_media_url = "http://file.api.weixin.qq.com/cgi-bin/media/get?access_token=ACCESS_TOKEN&media_id=MEDIA_ID";  
    /** 
     * 从微信服务器下载多媒体文件 
     * 
     * @author qincd 
     * @date Nov 6, 2014 4:32:12 PM 
     */  
    public static String downloadMediaFromWx(String accessToken,String mediaId,String fileSavePath) throws IOException {  
        if (StringUtils.isEmpty(accessToken) || StringUtils.isEmpty(mediaId)) return null;  
          
        String requestUrl = download_media_url.replace("ACCESS_TOKEN", accessToken).replace("MEDIA_ID", mediaId);  
        URL url = new URL(requestUrl);  
        HttpURLConnection conn = (HttpURLConnection) url.openConnection();  
        conn.setRequestMethod("GET");  
        conn.setDoInput(true);  
        conn.setDoOutput(true);  
        InputStream in = conn.getInputStream();  
          
        File dir = new File(fileSavePath);  
        if (!dir.exists()) {  
            dir.mkdirs();  
        }  
        if (!fileSavePath.endsWith("/")) {  
            fileSavePath += "/";  
        }  
          
        String ContentDisposition = conn.getHeaderField("Content-disposition");  
        String weixinServerFileName = ContentDisposition.substring(ContentDisposition.indexOf("filename")+10, ContentDisposition.length() -1);  
        fileSavePath += weixinServerFileName;   
        BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream(fileSavePath));  
        byte[] data = new byte[1024];  
        int len = -1;  
          
        while ((len = in.read(data)) != -1) {  
            bos.write(data,0,len);  
        }  
          
        bos.close();  
        in.close();  
        conn.disconnect();  
          
        return fileSavePath;  
    }  
}
{% endhighlight %}

测试代码：
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
        String filePath = "C:\\Users\\qince\\Pictures\\壁纸20141029091612.jpg";
        JSONObject uploadJsonObj = testUploadMedia(filePath,"image",accessToken.getToken());
        if (uploadJsonObj == null) {
            System.out.println("上传图片失败");
            return;
        }
        
        int errcode = 0;
        if (uploadJsonObj.containsKey("errcode")) {
            errcode = uploadJsonObj.getInt("errcode");
        }
        if (errcode == 0) {
            System.out.println("图片上传成功");
            
            String mediaId = uploadJsonObj.getString("media_id");
            long createAt = uploadJsonObj.getLong("created_at");
            System.out.println("--Details:");
            System.out.println("media_id:" + mediaId);
            System.out.println("created_at:" + createAt);
        }
        else {
            System.out.println("图片上传失败！");
            
            String errmsg = uploadJsonObj.getString("errmsg");
            System.out.println("--Details:");
            System.out.println("errcode:" + errcode);
            System.out.println("errmsg:" + errmsg);
        }
        
        String mediaId = "6W-UvSrQ2hkdSdVQJJXShwtFDPLfbGI1qnbNFy8weZyb9Jac2xxxcAUwt8p4sYPH";
        String filepath = testDownloadMedia(accessToken.getToken(), mediaId, "d:/test");
        System.out.println(filepath);
    }


    /**
     * 上传多媒体文件到微信
     *
     * @author qincd
     * @date Nov 6, 2014 4:15:14 PM
     */
    public static JSONObject testUploadMedia(String filePath,String type,String accessToken) {
        try {
            return WeixinUtil.uploadMediaToWX(filePath, type, accessToken);
        } catch (IOException e) {
            e.printStackTrace();
        }
        
        return null;
    }
    
    /**
     * 从微信下载多媒体文件
     *
     * @author qincd
     * @date Nov 6, 2014 4:56:25 PM
     */
    public static String testDownloadMedia(String accessToken,String mediaId,String fileSaveDir) {
        try {
            return WeixinUtil.downloadMediaFromWx(accessToken, mediaId, fileSaveDir);
        } catch (IOException e) {
            e.printStackTrace();
        }
        
        return null;
    }
}
{% endhighlight %}