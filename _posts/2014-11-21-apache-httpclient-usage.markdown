---
layout: post
title:  apache httpclient和fileupload包使用
description: apache httpclient和fileupload包使用
date:   2014-11-21 16:49:39
categories: blog
tags: frameworks httpclient fileupload
---
{% highlight java %}
package httpclient;
 
import java.awt.Image;
import java.io.BufferedReader;
import java.io.File;
import java.io.FileNotFoundException;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.ObjectInputStream;
import java.nio.charset.Charset;
import java.text.SimpleDateFormat;
import java.util.Date;
 
import javax.imageio.ImageIO;
import javax.swing.ImageIcon;
import javax.swing.JFrame;
import javax.swing.JLabel;
 
import org.apache.commons.httpclient.Cookie;
import org.apache.commons.httpclient.Header;
import org.apache.commons.httpclient.HttpClient;
import org.apache.commons.httpclient.HttpException;
import org.apache.commons.httpclient.HttpMethod;
import org.apache.commons.httpclient.HttpStatus;
import org.apache.commons.httpclient.MultiThreadedHttpConnectionManager;
import org.apache.commons.httpclient.NameValuePair;
import org.apache.commons.httpclient.cookie.CookieSpec;
import org.apache.commons.httpclient.methods.GetMethod;
import org.apache.commons.httpclient.methods.PostMethod;
import org.apache.commons.httpclient.methods.multipart.FilePart;
import org.apache.commons.httpclient.methods.multipart.MultipartRequestEntity;
import org.apache.commons.httpclient.methods.multipart.Part;
import org.apache.commons.httpclient.methods.multipart.StringPart;
import org.apache.commons.httpclient.params.HttpMethodParams;
 
import com.company.project.model.TradePack;
 
/**
 * @author qince
 *
 */
public class HttpClientUseDemo {
 
    /**
     * 这个方法最典型的应用，就是用来获取服务器支持哪些HTTP方法。
     * 当然，在HttpClient组件中有一个名称为OptionsMethod的类，支持这种形式的HTTP请求方式，调用OptionsMethod类的getAllowedMethods方法，
     * 就可以很简单地实现上述的典型应用。
     *
     * @author qincd
     * @date Oct 10, 2014 3:27:27 PM
     */
    public static void main(String[] args) {
        // oneSimpleDemo();
//       loginSystem();
//      getValidateCodeImage();
//      executeInMultiThread();
//      transferObject();
        uploadFile(new File("d:/logs/msp/common-all.log"));
    }
 
    public static void oneSimpleDemo() {
        HttpClient httpClient = new HttpClient();
        // 设置代理
        // httpClient.getHostConfiguration().setProxy(proxyHost, proxyPort);
        HttpMethod getMethod = new GetMethod("http://qincdtest.ematong.com/msp");
        try {
            int httpStatusCode = httpClient.executeMethod(getMethod);
            if (httpStatusCode == HttpStatus.SC_OK) {
                System.out.println(getMethod.getStatusLine());
                System.out.println(getMethod.getResponseBodyAsString());
                System.out.println("+++++++++++++++++++++++++++++++++++++++++");
                System.out.println("+++++++++++++++++++++++++++++++++++++++++");
                System.out.println("+++++++++++++++++++++++++++++++++++++++++");
                InputStream inout = getMethod.getResponseBodyAsStream();
                BufferedReader br = new BufferedReader(new InputStreamReader(
                        inout, Charset.forName("utf-8")));
                String line = null;
 
                while ((line = br.readLine()) != null) {
                    System.out.println(line);
                }
            } else {
                System.out.println("与远程服务器通讯发生故障");
            }
        } catch (HttpException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            getMethod.releaseConnection();
        }
    }
 
    /**
     * 使用httpclient模拟http请求登录系统
     *
     * @author qincd
     * @date Oct 10, 2014 3:58:44 PM
     */
    public static void loginSystem() {
        HttpClient httpClient = new HttpClient();
        String loginUrl = "http://qincdtest.ematong.com/msp/manage/login";
        PostMethod postMethod = new PostMethod(loginUrl);
        postMethod.addRequestHeader("Content-Type",
                "application/x-www-form-urlencoded;charset=utf-8");
        NameValuePair usernamePair = new NameValuePair("username", "admin");
        NameValuePair passwordPair = new NameValuePair("password", "123456");
        postMethod.setRequestBody(new NameValuePair[] { usernamePair,
                passwordPair });
        try {
            int httpStatusCode = httpClient.executeMethod(postMethod);
            System.out.println(httpStatusCode);
            if (httpStatusCode == HttpStatus.SC_OK) {
                System.out.println(postMethod.getStatusLine());
                System.out.println(postMethod.getResponseBodyAsString());
            } else if (httpStatusCode == HttpStatus.SC_MOVED_PERMANENTLY
                    || httpStatusCode == HttpStatus.SC_MOVED_TEMPORARILY
                    || httpStatusCode == HttpStatus.SC_TEMPORARY_REDIRECT) {
                Header locationHeader = postMethod.getResponseHeader("location");
                String redirectUrl = locationHeader.getValue();
                System.out.println("页面需要重定向到" + redirectUrl);
                 
                System.out.println("----重定向页面内容：");
                GetMethod getMethod = new GetMethod(redirectUrl);
                httpClient.executeMethod(getMethod);
                BufferedReader br = new BufferedReader(new InputStreamReader(getMethod.getResponseBodyAsStream(),"utf-8"));
                String temp = null;
                while ((temp = br.readLine()) != null) {
                    System.out.println(temp);
                }
                getMethod.releaseConnection();
            }
 
            CookieSpec cookieSpec = org.apache.commons.httpclient.cookie.CookiePolicy
                    .getDefaultSpec();
            Cookie[] cookies = cookieSpec.match("qincdtest.ematong.com", 8080,
                    "/", false, httpClient.getState().getCookies());
            if (cookies == null || cookies.length == 0) {
                System.out.println("No cookie...");
            } else {
                System.out.println("-----------Cookie------------------");
                for (Cookie cookie : cookies) {
                    System.out.println(cookie.getName() + ":"
                            + cookie.getValue());
                }
            }
        } catch (HttpException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            postMethod.releaseConnection();
        }
 
    }
 
    /**
     * 根据验证码图片生成地址得到验证码图片
     *
     * @author qincd
     * @date Oct 10, 2014 4:23:01 PM
     */
    public static void getValidateCodeImage() {
        // 12306登录的验证码图片地址
        String imageUrl = "https://kyfw.12306.cn/otn/passcodeNew/getPassCodeNew?module=login&rand=sjrand";
        imageUrl = "http://c.hiphotos.baidu.com/baike/c0%3Dbaike150%2C5%2C5%2C150%2C50%3Bt%3Dgif/sign=8e3ef28e1f950a7b613846966bb809bc/3b87e950352ac65cd20ecfcbf9f2b21193138a7b.jpg";
        HttpClient client = new HttpClient();
        GetMethod postMethod = new GetMethod(imageUrl);
        try {
            int status = client.executeMethod(postMethod);
            if (status == HttpStatus.SC_OK) {
                InputStream input = postMethod.getResponseBodyAsStream();
                Image image = ImageIO.read(input);
                JFrame frame = new JFrame();
                frame.setSize(500, 300);
                frame.add(new JLabel("验证码："));
                frame.add(new JLabel(new ImageIcon(image)));
                frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
                frame.setVisible(true);
            } else {
                System.out.println("--network error.");
                System.out.println(postMethod.getStatusLine());
            }
        } catch (HttpException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            postMethod.releaseConnection();
        }
    }
     
    /**
     * 在多线程环境中运行
     *
     * @author qincd
     * @date Oct 10, 2014 4:56:40 PM
     */
    public static void executeInMultiThread() {
        MultiThreadedHttpConnectionManager threadManager = new MultiThreadedHttpConnectionManager();
        HttpClient client = new HttpClient(threadManager);
        GetMethod gm = new GetMethod("http://www.baidu.com");
        try {
            int status = client.executeMethod(gm);
            if (status == HttpStatus.SC_OK) {
                System.out.println(gm.getStatusLine());
                System.out.println(gm.getResponseBodyAsString());
            }
        } catch (HttpException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            gm.releaseConnection();
        }
    }
     
    /**
     * 接收一个Object对象
     *
     * @author qincd
     * @date Oct 10, 2014 6:00:34 PM
     */
    public static void transferObject() {
        MultiThreadedHttpConnectionManager manager = new MultiThreadedHttpConnectionManager();
        HttpClient hc = new HttpClient(manager);
        PostMethod pm = new PostMethod("http://qincdtest.ematong.com/rapiddemo/manage/interface/test/77");
        try {
            int status = hc.executeMethod(pm);
            System.out.println(pm.getStatusLine());
            if (status == HttpStatus.SC_OK) {
                ObjectInputStream ois = new ObjectInputStream(pm.getResponseBodyAsStream());
                TradePack tradePack = (TradePack) ois.readObject();
                System.out.println("==========tradePack:");
                System.out.println("appId:" + tradePack.getAppId());
                System.out.println("tradeNo:" + tradePack.getTradeNo());
                System.out.println("tradeTime:" + tradePack.getTradeTime().toLocaleString());
                System.out.println("myFlowNo:" + tradePack.getMyFlowNo());
                System.out.println("sign:" + tradePack.getSign());
            }
        } catch (HttpException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } finally {
            pm.releaseConnection();
        }
         
    }
     
    /**
     * 上传文件
     *
     * @author qincd
     * @throws FileNotFoundException
     * @date Oct 10, 2014 6:00:50 PM
     */
    public static void uploadFile(File file) {
        HttpClient hc = new HttpClient();
        String uploadUrl = "http://qincdtest.ematong.com/rapiddemo/manage/interface/upload";
        PostMethod pm = new PostMethod(uploadUrl);
        try {
            FilePart fp = new FilePart("filepart",file);
            Part[] parts = {fp,new StringPart("uploadpath",file.getAbsolutePath()),new StringPart("uploadTime",new SimpleDateFormat("yyyy-MM-dd hh:mm:ss").format(new Date()))};
            HttpMethodParams hmps = pm.getParams();
            MultipartRequestEntity requestEntity = new MultipartRequestEntity(parts,hmps);
            pm.setRequestEntity(requestEntity);
            hc.getHttpConnectionManager().getParams().setConnectionTimeout(60*1000);
            int status = hc.executeMethod(pm);
             
            if (status == HttpStatus.SC_OK) {
                System.out.println(pm.getStatusLine());
                System.out.println(pm.getResponseBodyAsString());
                System.out.println("file upload success.");
            }
            else {
                System.out.println("upload error.");
            }
        } catch (HttpException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            pm.releaseConnection();
        }
    }
}
{% endhighlight %}
