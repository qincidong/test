---
layout: post
title:  Maven+Hibernate4注解0配置示例
description: Maven+Hibernate4注解0配置示例~~~
date:   2013-11-29 16:49:39
categories: blog
tags: maven hibernate 0配置
---
工程使用Maven来 管理依赖，Hibernate版本4.2.0.Final。  
工程结构图：  
![project](http://images.cnitblog.com/blog/548748/201311/29110530-5432c49db2a74acfbe180b32943ed57c.png) 

pom.xml文件：
{% highlight xml %}
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <groupId>com.purple_river.itat.maven.demo.user.dao</groupId>
    <artifactId>user-dao</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>
    
    <name>user-dao</name>
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
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-core</artifactId>
        <version>4.2.0.Final</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.27</version>
    </dependency>
  </dependencies>
</project>
{% endhighlight %}

hibernate.cfg.xml文件：  
{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>  
<!DOCTYPE hibernate-configuration PUBLIC  
    "-//Hibernate/Hibernate Configuration DTD 3.0//EN"  
    "http://hibernate.sourceforge.net/hibernate-configuration-3.0.dtd">
<hibernate-configuration>
    <session-factory>
        <!--显示执行的SQL语句 -->
        <property name="show_sql">true</property>
        <!-- 格式化输出 -->
        <property name="format_sql">true</property>
        <!--连接字符串 -->
        <property name="connection.url">jdbc:mysql://localhost:3306/test</property>
        <!--连接数据库的用户名 -->
        <property name="connection.username">root</property>
        <!--数据库用户密码 -->
        <property name="connection.password">soft</property>
        <!--数据库驱动 -->
        <property name="connection.driver_class">com.mysql.jdbc.Driver</property>
        <!--选择使用的方言 -->
        <property name="dialect">org.hibernate.dialect.MySQLDialect</property>
        <property name="hibernate.hbm2ddl.auto">update</property>
        <property name="configurationClass">org.hibernate.cfg.AnnotationConfiguration</property>
        <mapping class="com.purple_river.itat.maven.demo.bean.user.User" />
    </session-factory>
</hibernate-configuration>
{% endhighlight %}

User类：  
{% highlight java %}
package com.purple_river.itat.maven.demo.bean.user;

import java.io.Serializable;
import java.util.Date;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.Table;

import org.hibernate.annotations.GenericGenerator;

@Entity
@Table(name="user")
public class User implements Serializable
{
    private static final long    serialVersionUID    = -2602075959996355784L;
    @Id
    @GenericGenerator(name="generator",strategy="increment")
    @GeneratedValue(generator="generator")
    @Column(name="id",nullable=false,unique=true)
    private long id;
    /**
     * @return the id
     */
    public long getId() {
        return id;
    }

    /**
     * @param id the id to set
     */
    public void setId(long id) {
        this.id = id;
    }

    @Column(name="username",nullable=false)
    private String username;
    @Column(name="age")
    private int age;
    @Column(name="birthday")
    private Date birthday;
    
    /**
     * @return the username
     */
    public String getUsername() {
        return username;
    }

    /**
     * @param username the username to set
     */
    public void setUsername(String username) {
        this.username = username;
    }

    /**
     * @return the age
     */
    public int getAge() {
        return age;
    }

    /**
     * @param age the age to set
     */
    public void setAge(int age) {
        this.age = age;
    }

    /**
     * @return the birthday
     */
    public Date getBirthday() {
        return birthday;
    }

    /**
     * @param birthday the birthday to set
     */
    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }

    /**
     * 
     */
    public User() {
        super();
        // TODO Auto-generated constructor stub
    }

    /**
     * @param username
     * @param age
     * @param birthday
     */
    public User(String username, int age, Date birthday) {
        super();
        this.username = username;
        this.age = age;
        this.birthday = birthday;
    }

    /* (non-Javadoc)
     * @see java.lang.Object#toString()
     */
    @Override
    public String toString() {
        return "User [username=" + username + ", age=" + age + ", birthday="
                + birthday + "]";
    }
    
    
}
{% endhighlight %}

HibernateUtil类：  
{% highlight java %}
/*
 * @packageName:com.purple_river.itat.maven.demo.user.dao.util
 * @fileName:HibernateUtil.java
 * @description:Hibernate操作工具类
 * @author:luckystar
 * @date:2013-11-28
 */
package com.purple_river.itat.maven.demo.user.dao.util;

import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;
import org.hibernate.service.ServiceRegistry;
import org.hibernate.service.ServiceRegistryBuilder;

public final class HibernateUtil {
    private static SessionFactory factory = null;
    
    public static SessionFactory getSessionFactory() {
        if (factory == null) {
            Configuration conf = new Configuration().configure();
            ServiceRegistry serviceRegistry = new ServiceRegistryBuilder().applySettings(conf.getProperties()).buildServiceRegistry();
            factory = conf.buildSessionFactory(serviceRegistry);
        }
        
        return factory;
    }
    
    /*public static void main(String[] args) {
        SessionFactory sf = HibernateUtil.getSessionFactory();
        System.out.println(sf.toString());
    }*/
}
{% endhighlight %}

IUserDao类 ：  
{% highlight java %}
/*
 * @packageName:com.purple_river.itat.maven.demo.user.dao
 * @fileName:IUserDao.java
 * @description:用户管理数据库操作接口
 * @author:luckystar
 * @date:2013-11-28
 */
package com.purple_river.itat.maven.demo.user.dao;

import java.util.List;

import com.purple_river.itat.maven.demo.bean.user.User;

public interface IUserDao {

    public void addUser(User user);
    public User getUser(String name);
    public List<User> getAll();
}
{% endhighlight %}

UserDao类：  
{% highlight java %}
/*
 * @packageName:com.purple_river.itat.maven.demo.user.dao
 * @fileName:UserDao.java
 * @description:用户管理数据库操作接口
 * @author:luckystar
 * @date:2013-11-28
 */
package com.purple_river.itat.maven.demo.user.dao;

import java.util.List;

import org.hibernate.HibernateException;
import org.hibernate.Session;
import org.hibernate.SessionFactory;

import com.purple_river.itat.maven.demo.bean.user.User;
import com.purple_river.itat.maven.demo.user.dao.util.HibernateUtil;

public class UserDao implements IUserDao {

    /* (non-Javadoc)
     * @see com.purple_river.itat.maven.demo.user.dao.IUserDao#addUser(com.purple_river.itat.maven.demo.bean.user.User)
     */
    public void addUser(User user) {
        if (user == null) {
            throw new NullPointerException("the entity[user] which to be saved is null!");
        }
        
        Session session = null;
        try {
            SessionFactory factory = HibernateUtil.getSessionFactory();
            session = factory.openSession();
            session.beginTransaction();
            session.save(user);
            session.getTransaction().commit();
        } catch (HibernateException e) {
            session.getTransaction().rollback();
            e.printStackTrace();
        } finally {
            if (session != null) {
                session.close();
            }
        }
        
    }

    /* (non-Javadoc)
     * @see com.purple_river.itat.maven.demo.user.dao.IUserDao#getUser(java.lang.String)
     */
    public User getUser(String name) {
        User user = null;
        Session session = null;
        try {
            SessionFactory factory = HibernateUtil.getSessionFactory();
            session = factory.openSession();
            user = (User) session.createQuery("from User where username = ?").setParameter(0, name).uniqueResult();
        } catch (HibernateException e) {
            e.printStackTrace();
        } finally {
            if (session != null) {
                session.close();
            }
        }
        
        return user;
    }

    /* (non-Javadoc)
     * @see com.purple_river.itat.maven.demo.user.dao.IUserDao#getAll()
     */
    public List<User> getAll() {
        List<User> list = null;
        try {
            list = HibernateUtil.getSessionFactory().openSession().createCriteria(User.class).list();
        } catch (HibernateException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        return list;
    }

}
{% endhighlight %}

TestUserDao类：  
{% highlight java %}
/*
 * @packageName:com.purple_river.itat.maven.demo.user.dao
 * @fileName:TestUserDao.java
 * @description:Test Case for UserDao
 * @author:luckystar
 * @date:2013-11-28
 */
package com.purple_river.itat.maven.demo.user.dao;

import java.util.Calendar;
import java.util.List;

import org.jboss.logging.Logger;
import org.junit.After;
import org.junit.Assert;
import org.junit.Before;
import org.junit.Test;

import com.purple_river.itat.maven.demo.bean.user.User;

public class TestUserDao {
    public final static org.jboss.logging.Logger logger = Logger.getLogger(TestUserDao.class);
    
    private IUserDao userDao ;
    
    @Before
    public void setUp() {
        userDao = new UserDao();
    }
    
    @Test
    public void testAddUser() {
        logger.info("Test [UserDao.addUser].");
        Calendar c = Calendar.getInstance();
        c.set(2012, 01,01);
        User user = new User("admin",1,c.getTime());
        
        userDao.addUser(user);
    }
    
    @Test
    public void testGetUser() {
        logger.info("Test [UserDao.getUser].");
        
        User user = userDao.getUser("admin");
        logger.info(user);
        Assert.assertEquals(user.getUsername(),"admin");
        Assert.assertEquals(user.getAge(),1);
    }
    
    @Test
    public void testGetAll() {
        logger.info("Test [UserDao.listAll].");
        
        List<User> userList = userDao.getAll();
        Assert.assertTrue(userList.size() == 1);
        logger.info(userList.get(0));
    }
    
    @After
    public void tearDown() {
        userDao = null;
    }
}
{% endhighlight %} 

之所以记录下来，是因为搭建环境时遇到了一些错误，所以这里记录对以后有个参考。  
下载地址：[user-dao.rar](http://files.cnblogs.com/luckystar/user-dao.rar)
