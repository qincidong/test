---
layout: post
title: Maven模块与模块间的依赖
description: Maven模块与模块间的依赖
date:   2014-12-11 16:49:39
categories: blog
tags: maven module dependency easymock
---
在Maven+Hibernate4注解0配置示例中创建用户操作数据库的模块，现在创建用户管理服务模块user-service。  
user-service模块是要依赖user-dao模块的，怎么用maven来管理依赖呢？

1.在user-dao模块下,pom.xml中右键，选择maven install命令，将user-dao发布到本地仓库中；  
2.在user-service模块下,pom.xml中添加对user-dao模块的依赖。Maven会将user-dao中依赖的Jar一同拷贝过来。

user-service模块包括：  
1个用户操作服务接口类IUserService;  
1个用户操作服务实现类UserService;  
1个单元测试类TestUserSerivice。  

IUserService中包含3个方法，分别为添加用户，根据用户名查找某个用户，以及列出所有的用户。  
单元测试使用easymock来mock对象的预期行为。  

工程结构图：  
![project](http://images.cnitblog.com/blog/548748/201311/29152021-fe66d7456f3c4674b4e2ad03b38c4d84.png)

####代码：  
#####pom.xml:  
{% highlight xml %}
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.purple_river.itat.maven.demo.service</groupId>
  <artifactId>user-service</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>user-service</name>
  <url>http://maven.apache.org</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.10</version>
      <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>com.purple_river.itat.maven.demo.user.dao</groupId>
        <artifactId>user-dao</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </dependency>
    <dependency>
        <groupId>org.easymock</groupId>
        <artifactId>easymock</artifactId>
        <version>3.2</version>
    </dependency>
            
  </dependencies>
</project>
{% endhighlight %}

#####IUserService:
{% highlight java %}
/*
 * @packageName:com.purple_river.itat.maven.demo.service
 * @fileName:IUserService.java
 * @description:用户管理接口类
 * @author:luckystar2010
 * @date:2013-11-29
 */
package com.purple_river.itat.maven.demo.service;

import java.util.List;

import com.purple_river.itat.maven.demo.bean.user.User;

public interface IUserService {

    public void addUser(User user);
    
    public User queryUserByUserName(String username);
    
    public List<User> queryAll();
}
{% endhighlight %}

#####UserService:  
{% highlight java %}
/*
 * @packageName:com.purple_river.itat.maven.demo.service
 * @fileName:UserService.java
 * @description:用户管理服务类
 * @author:luckystar2010
 * @date:2013-11-29
 */
package com.purple_river.itat.maven.demo.service;

import java.util.List;

import com.purple_river.itat.maven.demo.bean.user.User;
import com.purple_river.itat.maven.demo.user.dao.IUserDao;

public class UserService implements IUserService {
    private IUserDao userDao;
    
    public UserService(IUserDao userDao) {
        this.userDao = userDao;
    }
    
    /* (non-Javadoc)
     * @see com.purple_river.itat.maven.demo.service.IUserService#addUser(com.purple_river.itat.maven.demo.bean.user.User)
     */
    public void addUser(User user) {
        this.userDao.addUser(user);
    }

    /* (non-Javadoc)
     * @see com.purple_river.itat.maven.demo.service.IUserService#queryUserByUserName(java.lang.String)
     */
    public User queryUserByUserName(String username) {
        return this.userDao.getUser(username);
    }

    /* (non-Javadoc)
     * @see com.purple_river.itat.maven.demo.service.IUserService#queryAll()
     */
    public List<User> queryAll() {
        return this.userDao.getAll();
    }

}
{% endhighlight %}

#####TestUserService:  
{% highlight java %}
/*
 * @packageName:com.purple_river.itat.maven.demo.service
 * @fileName:TestUserService.java
 * @description:用户管理服务测试类
 * @author:luckystar2010
 * @date:2013-11-29
 */
package com.purple_river.itat.maven.demo.service;

import java.util.ArrayList;
import java.util.Calendar;
import java.util.List;

import junit.framework.Assert;

import org.easymock.EasyMock;
import org.jboss.logging.Logger;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;

import com.purple_river.itat.maven.demo.bean.user.User;
import com.purple_river.itat.maven.demo.user.dao.IUserDao;

public class TestUserService {
    public final static Logger logger = Logger.getLogger(TestUserService.class);
    private IUserService userService;
    private IUserDao userDao;
    
    @Before
    public void setUp() {
        //this.userDao = EasyMock.createMock(IUserDao.class);
        this.userDao = EasyMock.createMock(IUserDao.class);
        logger.info("created mock:" + this.userDao);
        this.userService = new UserService(this.userDao);
    }
    
    @Test
    public void testAddUser() {
        logger.info("Test [UserService.addUser].");
        
        Calendar c = Calendar.getInstance();
        c.set(2012, 1,1);
        User user = new User("admin",1,c.getTime());
        // 当Mock的对象调用的是返回值为void方法时，可以用EasyMock.expectLastCall();来设置返回值是void
        // 或者不用显示的设定它的返回值。
        // 比如下面2行
        // this.userDao.addUser(user);
        // EasyMock.expectLastCall();
        // 可以不要。
        this.userDao.addUser(user);
        EasyMock.expectLastCall();
        this.userService.addUser(user);
    }
    
    @Test
    public void testQueryByUesrName() {
        logger.info("Test [UserService.queryByUserName]");
        
        // record
        User user = new User();
        user.setUsername("admin");
        user.setAge(1);
        EasyMock.expect(this.userDao.getUser("admin")).andReturn(user);
        
        // replay
        EasyMock.replay(this.userDao);
        User u = this.userService.queryUserByUserName("admin");
        
        // verify
        Assert.assertEquals(u.getUsername(), "admin");
        Assert.assertEquals(u.getAge(), 1);
        
        EasyMock.verify(this.userDao);
    }
    
    @Test
    public void testQueryAll() {
        logger.info("Test [UserService.queryAll].");
        
        // record，设置方法的返回值
        User user = new User();
        user.setUsername("admin");
        user.setAge(1);
        List<User> userList = new ArrayList<User>();
        userList.add(user);
        // 设定当UserService调用UserDao.getAll()方法时返回userList
        EasyMock.expect(this.userDao.getAll()).andReturn(userList);
        
        // replay，在调用方法时EasyMock将我们设置的值返回
        EasyMock.replay(this.userDao);
        List<User> list = this.userService.queryAll();
        
        // verify，验证方法返回的结果与我们预期的是否一致
        Assert.assertEquals(list.size(), 1);
        EasyMock.verify(this.userDao);
    }
    
    @After
    public void tearDown() {
        this.userDao = null;
        this.userService = null;
    }
}
{% endhighlight %}
下载地址：[user-service.rar](http://files.cnblogs.com/luckystar2010/user-service.rar)