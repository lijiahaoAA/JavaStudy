### Mybatis注解开发单表CRUD

***

>mybatis注解开发和xml开发不可兼容，要么全部使用注解，要么全部使用xml，个人建议注解，简单。

### 当实体类属性名称和数据库表属性名称一致时：无需配置Results

#### 实体类User

~~~java
package com.itheima.domain;

import java.io.Serializable;
import java.util.Date;
import java.util.List;

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

#### 接口IUserDao

~~~java
package com.itheima.dao;

import com.itheima.domain.User;
import org.apache.ibatis.annotations.Delete;
import org.apache.ibatis.annotations.Insert;
import org.apache.ibatis.annotations.Select;
import org.apache.ibatis.annotations.Update;

import java.beans.IntrospectionException;
import java.util.List;

/**
 * @Author: lijiahao
 * @Description: 用户的持久层接口
 * @Data: Create in 21:42 2020/2/22
 * @Modified By:
 */
public interface IUserDao {

    //查询所有用户，同时获取到用户下所有账户的信息
    @Select("select * from user")
    List<User> findAll();

    @Insert("Insert into user (username,address,sex,birthday) values (#{username},#{address},#{sex},#{birthday})")
    void  saveUser(User user);

    @Update("update user set username=#{username},address=#{address},sex=#{sex},birthday=#{birthday} where id = #{id}")
    void updateUser(User user);

    @Delete("delete from user where id = #{id}")
    void deleteUser(Integer integer);

    @Select("select * from user where id = #{id}")
    User findById(Integer integer);

    @Select("Select * from user where username like #{username}")
 /*   @Select("Select * from user where username like '%${value}%'")*/
    List<User> findUserByName(String username);

    @Select("select count(*) from user")
    int findTotalUser();
}

~~~

#### 配置文件

##### 主配置文件

~~~java
<?xml version="1.0" encoding="utf-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<!--mybatis的主配置文件-->
<configuration>

    <properties resource="jdbcConfig.properties"/>

    <!--配置别名 类名就是别名，不区分大小写-->
    <typeAliases>
        <package name="com.itheima.domain" ></package>
    </typeAliases>

    <!--配置环境-->
    <environments default="mysql">
        <!--配置mysql的环境-->
        <environment id="mysql">
            <!--配置事务的类型-->
            <transactionManager type="JDBC"></transactionManager>
            <!--配置数据源（连接池）-->
            <dataSource type="POOLED">
                <!--配置数据库的四个基本信息-->
                <property name="driver" value="${jdbc.driver}" />
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}" />
                <property name="password" value="${jdbc.password}" />
            </dataSource>
        </environment>
    </environments>

    <!--指定dao接口所在位置-->
    <mappers>
        <package name="com.itheima.dao"></package>
    </mappers>
</configuration>
~~~

##### 连接池

~~~java
jdbc.driver=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/eesy?serverTimezone=Asia/Shanghai&characterEncoding=utf8&useSSL=false
jdbc.username=root
jdbc.password=123456
~~~

##### 日志

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

***

#### 测试类

~~~java
package com.itheima;

import com.itheima.dao.IUserDao;
import com.itheima.domain.User;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;

import java.io.IOException;
import java.io.InputStream;
import java.util.Date;
import java.util.List;

/**
 * @Author: lijiahao
 * @Description:
 * @Data: Create in 22:56 2020/2/22
 * @Modified By:
 */
public class AnnotationCRUDTest {
    private InputStream in;
    private SqlSession sqlSession;
    private IUserDao iUserDao;
    private SqlSessionFactory sqlSessionFactory;

    @Before
    public void init() throws Exception {
        in = Resources.getResourceAsStream("SqlMapConfig.xml");
        sqlSessionFactory = new SqlSessionFactoryBuilder().build(in);
        sqlSession = sqlSessionFactory.openSession();
        iUserDao = sqlSession.getMapper(IUserDao.class);
    }

    @After
    public void destroy() throws IOException {
        sqlSession.commit();
        sqlSession.close();
        in.close();
    }

    @Test
    public void testSave(){
        User user = new User();
        user.setUsername("李家豪");
        user.setAddress("郑州市");

        iUserDao.saveUser(user);
    }

    @Test
    public void testUpdate(){
        User user = new User();
        user.setId(47);
        user.setUsername("李家豪");
        user.setAddress("郑州市");
        user.setSex("女");
        user.setBirthday(new Date());

        iUserDao.updateUser(user);
        System.out.println(user);
    }

    @Test
    public void testDelete(){
        iUserDao.deleteUser(47);
    }

    @Test
    public void testFindById(){
        User user = iUserDao.findById(57);
        System.out.println(user);
    }

    @Test
    public void testFindUserByName(){
        List<User> users = iUserDao.findUserByName("%王%");
        //List<User> users = iUserDao.findUserByName("王");
        for(User user : users)
        System.out.println(user);
    }

    @Test
    public void testFindTotal(){
        int count = iUserDao.findTotalUser();
        System.out.println(count);
    }
}
~~~

***

### 当实体类属性名称和数据库表属性名称一致时：需配置Results

#### 接口IUserDao

~~~java
package com.itheima.dao;

import com.itheima.domain.User;
import org.apache.ibatis.annotations.*;

import java.beans.IntrospectionException;
import java.util.List;

/**
 * @Author: lijiahao
 * @Description: 用户的持久层接口
 * @Data: Create in 21:42 2020/2/22
 * @Modified By:
 */
public interface IUserDao {

    //查询所有用户，同时获取到用户下所有账户的信息
    @Select("select * from user")
    //当实体类的属性和数据库的属性名称不一致时，需要配置对应关系，id为该Results的唯一标识
    @Results(id = "userMap", value={
            @Result(id=true,column = "id",property = "userId"),
            @Result(column = "username",property = "userName"),
            @Result(column = "sex",property = "userSex"),
            @Result(column = "address",property = "userAddress"),
            @Result(column = "birthday",property = "userBirthday"),
    })
    List<User> findAll();

    @Insert("Insert into user (username,address,sex,birthday) values (#{username},#{address},#{sex},#{birthday})")
    void  saveUser(User user);

    @Update("update user set username=#{username},address=#{address},sex=#{sex},birthday=#{birthday} where id = #{id}")
    void updateUser(User user);

    @Delete("delete from user where id = #{id}")
    void deleteUser(Integer integer);

    @Select("select * from user where id = #{id}")
    @ResultMap(value = {"userMap"})
    User findById(Integer integer);

    @Select("Select * from user where username like #{username}")
    @ResultMap(value = {"userMap"})
    List<User> findUserByName(String username);

}

~~~

#### 测试

~~~java
package com.itheima;

import com.itheima.dao.IUserDao;
import com.itheima.domain.User;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;

import java.io.IOException;
import java.io.InputStream;
import java.util.Date;
import java.util.List;

/**
 * @Author: lijiahao
 * @Description:
 * @Data: Create in 22:56 2020/2/22
 * @Modified By:
 */
public class AnnotationCRUDTest {
    private InputStream in;
    private SqlSession sqlSession;
    private IUserDao iUserDao;
    private SqlSessionFactory sqlSessionFactory;

    @Before
    public void init() throws Exception {
        in = Resources.getResourceAsStream("SqlMapConfig.xml");
        sqlSessionFactory = new SqlSessionFactoryBuilder().build(in);
        sqlSession = sqlSessionFactory.openSession();
        iUserDao = sqlSession.getMapper(IUserDao.class);
    }

    @After
    public void destroy() throws IOException {
        sqlSession.commit();
        sqlSession.close();
        in.close();
    }

    @Test
    public void testFindAll(){
        List<User> users = iUserDao.findAll();
        for(User user : users)
            System.out.println(user);
    }

    @Test
    public void testFindById(){
        User user = iUserDao.findById(57);
        System.out.println(user);
    }

    @Test
    public void testFindUserByName(){
        List<User> users = iUserDao.findUserByName("%王%");
        //List<User> users = iUserDao.findUserByName("王");
        for(User user : users)
        System.out.println(user);
    }

}
~~~



