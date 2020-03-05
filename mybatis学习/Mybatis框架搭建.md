## Mybatis框架搭建

### 一、Mybatis框架搭建 xml配置

##### 1. 创建maven工程并导入坐标

~~~java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itheima</groupId>
    <artifactId>day01_eesy_01mybatis</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <dependencies>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.16</version>
        </dependency>

        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.4.1</version>
        </dependency>

        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.17</version>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
    </dependencies>
</project>
~~~

##### 2. 创建实体类和dao的接口

~~~java
package com.itheima.domain;

import java.io.Serializable;
import java.util.Date;

/**
 * @Author: lijiahao
 * @Description:
 * @Data: Create in 21:37 2020/2/22
 * @Modified By:
 */
public class User implements Serializable {
    private Integer id;
    private String username;
    private Date birthday;
    private String sex;
    private String address;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public Date getBirthday() {
        return birthday;
    }

    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", username='" + username + '\'' +
                ", birthday=" + birthday +
                ", sex='" + sex + '\'' +
                ", address='" + address + '\'' +
                '}';
    }
}

~~~

~~~java
package com.itheima.dao;

import com.itheima.domain.User;

import java.util.List;

/**
 * @Author: lijiahao
 * @Description: 用户的持久层接口
 * @Data: Create in 21:42 2020/2/22
 * @Modified By:
 */
public interface IUserDao {

    List<User> findAll();
}

~~~

##### 3. 创建mybatis的主配置文件SqlMapConfig.xml

~~~java
<?xml version="1.0" encoding="utf-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<!--mybatis的主配置文件-->
<configuration>
    <!--配置环境-->
    <environments default="mysql">
        <!--配置mysql的环境-->
        <environment id="mysql">
            <!--配置事务的类型-->
            <transactionManager type="JDBC"></transactionManager>
            <!--配置数据源（连接池）-->
            <dataSource type="POOLED">
                <!--配置数据库的四个基本信息-->
                <property name="driver" value="com.mysql.cj.jdbc.Driver" />
                <property name="url" value="jdbc:mysql://localhost:3306/eesy?serverTimezone=Asia/Shanghai&amp;characterEncoding=utf8&amp;useSSL=false" />
                <property name="username" value="root" />
                <property name="password" value="123456" />
            </dataSource>
        </environment>
    </environments>

    <!--指定映射配置文件的位置，映射配置文件指的是每个dao独立的配置文件-->
    <mappers>
        <mapper resource="com/itheima/dao/IUserDao.xml"/>
    </mappers>
</configuration>
~~~

##### 4.log4j.properties配置文件

~~~java
### 设置###
log4j.rootLogger = debug,CONSOLE,LOGFILE

log4j.logger.org.apache.axis.enterprise=FATAL,CONSOLE

log4j.appender.CONSOLE=org.apache.log4j.ConsoleAppender
log4j.appender.CONSOLE.layout=org.apache.log4j.PatternLayout
log4j.appender.CONSOLE.layout.ConversionPattern=%d{ISO8601} %-6r [%15.15t] %-5p %30.30c %x - %m\n

### 输出ERROR 级别以上的日志到=E://logs/error.log ###
log4j.appender.LOGFILE = org.apache.log4j.FileAppender
log4j.appender.LOGFILE.FILE = F://logs/error.log
log4j.appender.LOGFILE.Append = true
log4j.appender.LOGFILE.layout = org.apache.log4j.PatternLayout
log4j.appender.LOGFILE.layout.ConversionPattern = %d{ISO8601} %-6r [%15.15t] %-5p %30.30c %x - %m\n
~~~

##### 5.接口映射配置文件 IUserDao.xml

~~~java
//与接口位置一样
//接口相对位置 java/com/itheima/dao/IUserDao
//接口配置文件相对位置 resources/com/itheima/dao/IUserDao.xml
<?xml version="1.0" encoding="utf-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
//resultType是返回查询要封装实体类
<mapper>
    <!--配置查询所有-->
    <select id="findAll" resultType="com.itheima.domain.User">
        select * from user
    </select>
</mapper>
~~~

##### 6.测试方法

~~~java
package com.itheima;

import com.itheima.dao.IUserDao;
import com.itheima.domain.User;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import javax.annotation.Resource;
import java.io.IOException;
import java.io.InputStream;
import java.util.List;

/**
 * @Author: lijiahao
 * @Description:
 * @Data: Create in 22:56 2020/2/22
 * @Modified By:
 */
public class test {

    public static void main(String[] args) throws IOException {
        //1.读取配置文件
        InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
        //2.创建SqlSessionFactory工厂
        SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
        SqlSessionFactory sqlSessionFactory =  sqlSessionFactoryBuilder.build(in);
        //3.使用工厂生产SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession();
        //4.使用SqlSession创建dao接口的代理对象
        IUserDao userDao = sqlSession.getMapper(IUserDao.class);
        //5.使用代理对象执行方法
        List<User> users = userDao.findAll();
        for(User user : users){
            System.out.println(user);
        }
        //6.释放资源
        sqlSession.close();
        in.close();
    }
}

~~~

##### 7.注意

>- 接口映射配置文件位置必须和dao接口的包结构相同
>
>- 接口映射配置文件的mapper标签namespace属性的取值必须是dao接口的全限定类名
>
>- 接口映射配置文件的操作配置（select），id属性必须是dao接口的方法名
>
>遵从了上述几点，我们就把无需再写dao的实现类

---



### 二、Mybatis框架搭建 注解配置

##### 1.主配置文件SqlMapConfig.xml

~~~java
<?xml version="1.0" encoding="utf-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<!--mybatis的主配置文件-->
<configuration>
    <!--配置环境-->
    <environments default="mysql">
        <!--配置mysql的环境-->
        <environment id="mysql">
            <!--配置事务的类型-->
            <transactionManager type="JDBC"></transactionManager>
            <!--配置数据源（连接池）-->
            <dataSource type="POOLED">
                <!--配置数据库的四个基本信息-->
                <property name="driver" value="com.mysql.cj.jdbc.Driver" />
                <property name="url" value="jdbc:mysql://localhost:3306/eesy?serverTimezone=Asia/Shanghai&amp;characterEncoding=utf8&amp;useSSL=false" />
                <property name="username" value="root" />
                <property name="password" value="123456" />
            </dataSource>
        </environment>
    </environments>

    <!--
    如果是使用注解来配置的话，此处应该使用class属性指定被注解的dao的全限定类名-->
    <mappers>
        <mapper class="com.itheima.dao.IUserDao"/>
    </mappers>
</configuration>
~~~

##### 2.接口添加注解

~~~java
public interface IUserDao {

    @Select("select * from user")
    List<User> findAll();
}
~~~

##### 3.无需创建接口映射配置文件

---



### 三、自定义mybatis开发流程

>1、SqlSessionFactoryBuilder接收SqlMapConfig.xml文件流，构建出SqlSessionFactory对象。
>
>2、SqlSessionFactory读取SqlMapConfig.xml中连接数据库和mapper映射信息。用来生产出真正操作数据库的SqlSession对象。
>
>3、SqlSession对象有两大作用（注：无论哪个分支，除了连接数据库信息，还需要得到sql语句。）
>
>- **生成接口代理对象**
>
>  在SqlSessionImpl对象的getMapper方法中分两步来实现
>
>  1. 先用SqlSessionFactory读取的数据库连接信息创建Connection对象。
>  2. 通过jdk代理模式创建出代理对象作为getMapper方法返回值，这里主要工作是在创建代理对象时第三个参数处理类里面得到sql语句。执行对应CRUD操作。
>
>- **定义通用增删查改方法**
>
>  在SqlSessionImpl对象中提供selectList()方法，这些方法内也分两步
>
>  1. 用SqlSessionFactory读取的数据库连接信息创建出jdbc的Connection对象。
>  2. 直接得到sql语句，使用jdbc的Connection对象进行对应的CRUD操作。
>
>4、封装结果集：无论使用生成代理对象方法，还是使用分支二提供的CRUD方法，我们都要对返回的数据库结果集进行封装，变成java对象返回给调用者。我们还必须要知道调用者所需要的返回类型。

---



