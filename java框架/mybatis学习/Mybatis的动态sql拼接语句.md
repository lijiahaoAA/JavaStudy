#### Mybatis的动态sql拼接语句

***

![项目结构图](C:\Users\13018\Desktop\mybatis学习\动态sql拼接项目结构图.jpg)

##### 1.主配置文件SqlMapConfig.xml

~~~java
<?xml version="1.0" encoding="utf-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<!--mybatis的主配置文件-->
<configuration>

    <properties resource="jdbcConfig.properties"/>

    <!--配置别名-->
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

    <!--指定映射配置文件的位置，映射配置文件指的是每个dao独立的配置文件-->
    <mappers>
        <package name="com.itheima.dao"></package>
    </mappers>
</configuration>
~~~

##### 2.dao接口映射配置文件

~~~java
<?xml version="1.0" encoding="utf-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.itheima.dao.IUserDao">
    <!--配置查询所有-->
    <!--    resultType:返回值要封装的全限定类名
            parameterType:传递自定义参数时，写入参数的全限定类名
                          传递基本类型时，写入java.lang.Integer/INT/Integer都可以
                          -->

    <!--抽取重复的sql代码-->
    <sql id="asdfgh" >
        select * from user;
    </sql>

    <select id="findAll" resultType="user">
        <include refid="asdfgh"></include>
        <!-- select * from user;  -->
    </select>


    <!--根据id查找一个用户的信息-->
    <select id="findById" parameterType="INT" resultType="user">
        select * from user where id = #{userid}
    </select>

    <!--根据名称模糊查询-->
    <select id="findByName" parameterType="string" resultType="com.itheima.domain.User">
        select * from user where username like #{name}
        /* select * from user where username like '%${value}%' */
    </select>

    <!--根据QueryVo的条件查询用户-->
    <select id="findUserByVo" parameterType="com.itheima.domain.QueryVo" resultType="com.itheima.domain.User">
          select * from user where username like #{user.username}
    </select>

    <!--根据条件查询-->
    <select id="findUserByCondition" resultType="com.itheima.domain.User" parameterType="user">
        select * from user
        <where>
            <if test="username != null">
                and username = #{username}
            </if>
            <if test="sex != null">
                and sex = #{sex}
            </if>
        </where>
    </select>

    <!--根据queryvo中的id集合实现查询用户列表-->
    <select id="findUserInIds" resultType="com.itheima.domain.User" parameterType="com.itheima.domain.QueryVo">
        select * from user
        <where>
            <if test="ids != null and ids.size() > 0">
                <foreach collection="ids" item="uid" separator="," open="and id in (" close=")">
                    #{uid}
                </foreach>
            </if>
        </where>
    </select>
</mapper>
~~~

##### 3.jdbc连接池

~~~java
jdbc.driver=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/eesy?serverTimezone=Asia/Shanghai&characterEncoding=utf8&useSSL=false
jdbc.username=root
jdbc.password=123456
~~~

##### 4.日志

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

##### 5.实体类user

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

##### 6.实体类QueryVo

~~~java
package com.itheima.domain;

import java.util.List;

/**
 * @Author: lijiahao
 * @Description:
 * @Data: Create in 16:04 2020/2/24
 * @Modified By:
 */
public class QueryVo {

    private User user;

    private List<Integer> ids;

    public List<Integer> getIds() {
        return ids;
    }

    public void setIds(List<Integer> ids) {
        this.ids = ids;
    }

    public User getUser() {
        return user;
    }

    public void setUser(User user) {
        this.user = user;
    }
}

~~~

##### 7.接口

~~~java
package com.itheima.dao;

import com.itheima.domain.QueryVo;
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

    //根据id查询用户信息
    User findById(Integer userId);

    //根据名称模糊查询用户信息
    List<User> findByName(String username);

    //根据QueryVo中的条件查询用户
    List<User> findUserByVo(QueryVo vo);

    /**
     * @Author Lijiahao
     * @Description 根据传入的参数条件查询
     * @Date 18:35 2020/2/25
     * @Param [user] 传入的条件：有可能有用户名，有可能有性别，也有可能有地址，也可能全都有
     * @return java.util.List<com.itheima.domain.User>
     **/
    List<User> findUserByCondition(User user);

    /**
     * @Author Lijiahao
     * @Description 根据QueryVo中提供的id集合，查询用户信息
     * @Date 23:11 2020/2/25
     * @Param [vo]
     * @return java.util.List<com.itheima.domain.User>
     **/
    List<User> findUserInIds(QueryVo  vo);

}

~~~

##### 8.接口实现

~~~java
package com.itheima.dao.impl;

import com.itheima.dao.IUserDao;
import com.itheima.domain.QueryVo;
import com.itheima.domain.User;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;

import java.util.List;

/**
 * @Author: lijiahao
 * @Description:
 * @Data: Create in 16:39 2020/2/24
 * @Modified By:
 */
public class UserDaoImpl implements IUserDao {

    private SqlSessionFactory factory;

    public UserDaoImpl(SqlSessionFactory factory){
        this.factory = factory;
    }

    public List<User> findAll() {
        //1.根据factory获取SqlSession对象
        SqlSession sqlSession = factory.openSession();
        //2.调用SqlSession中的方法实现查询列表
        List<User> users  = sqlSession.selectList("com.itheima.dao.IUserDao.findAll");//参数就是能获得配置信息的key
        //3.释放资源
        sqlSession.close();
        return users;
    }

    public void saveUser(User user) {
        //1.根据factory获取SqlSession对象
        SqlSession sqlSession = factory.openSession();
        //2.调用SqlSession中的方法实现更新列表
        sqlSession.insert("com.itheima.dao.IUserDao.saveUser",user);
        //3.提交事务
        sqlSession.commit();
        //4.关闭资源
        sqlSession.close();
    }

    public void updateUser(User user) {
        //1.根据factory获取SqlSession对象
        SqlSession sqlSession = factory.openSession();
        //2.调用SqlSession中的方法实现更新列表
        sqlSession.update("com.itheima.dao.IUserDao.updateUser",user);
        //3.提交事务
        sqlSession.commit();
        //4.关闭资源
        sqlSession.close();
    }

    public void deleteUser(Integer userId) {
        //1.根据factory获取SqlSession对象
        SqlSession sqlSession = factory.openSession();
        //2.调用SqlSession中的方法实现更新列表
        sqlSession.delete("com.itheima.dao.IUserDao.deleteUser",userId);
        //3.提交事务
        sqlSession.commit();
        //4.关闭资源
        sqlSession.close();
    }

    public User findById(Integer userId) {
        //1.根据factory获取SqlSession对象
        SqlSession sqlSession = factory.openSession();
        //2.调用SqlSession中的方法，实现查询一个
        User user = sqlSession.selectOne("com.itheima.dao.IUserDao.findById",userId);
        //3.提交事务
        sqlSession.commit();
        //4.关闭资源
        sqlSession.close();
        return user;
    }

    public List<User> findByName(String username) {
        //1.根据factory获取SqlSession对象
        SqlSession sqlSession = factory.openSession();
        //2.调用SqlSession中的方法，实现查询一个
        List<User> users = sqlSession.selectList("com.itheima.dao.IUserDao.findByName",username);
        //3.提交事务
        sqlSession.commit();
        //4.关闭资源
        sqlSession.close();
        return users;
    }

    public List<User> findUserByVo(QueryVo vo) {
        return null;
    }

    public List<User> findUserByCondition(User user) {
        //1.根据factory获取SqlSession对象
        SqlSession sqlSession = factory.openSession();
        //2.调用SqlSession中的方法，实现查询一个
        List<User> users = sqlSession.selectList("com.itheima.dao.IUserDao.findUserByCondition",user);
        //3.提交事务
        sqlSession.commit();
        //4.关闭资源
        sqlSession.close();
        return users;
}

    public List<User> findUserInIds(QueryVo vo) {
        //1.根据factory获取SqlSession对象
        SqlSession sqlSession = factory.openSession();
        //2.调用SqlSession中的方法，实现查询一个
        List<User> users = sqlSession.selectList("com.itheima.dao.IUserDao.findUserInIds",vo);
        //3.提交事务
        sqlSession.commit();
        //4.关闭资源
        sqlSession.close();
        return users;
    }

    public int findTotal() {
        //1.根据factory获取SqlSession对象
        SqlSession sqlSession = factory.openSession();
        //2.调用SqlSession中的方法，实现查询一个
        Integer count = sqlSession.selectOne("com.itheima.dao.IUserDao.findTotal");
        //3.提交事务
        sqlSession.commit();
        //4.关闭资源
        sqlSession.close();
        return count;
    }
}

~~~

##### 9.测试

~~~java
package com.itheima;

import com.itheima.dao.IUserDao;
import com.itheima.dao.impl.UserDaoImpl;
import com.itheima.domain.QueryVo;
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
import java.util.ArrayList;
import java.util.Date;
import java.util.List;

/**
 * @Author: lijiahao
 * @Description:
 * @Data: Create in 22:56 2020/2/22
 * @Modified By:
 */
public class test {

    private InputStream in;
    private IUserDao userDao;

    @Before
    public void init() throws IOException {
        //1.读取配置文件
        in = Resources.getResourceAsStream("SqlMapConfig.xml");
        //2.创建SqlSessionFactory工厂
        SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
        SqlSessionFactory sqlSessionFactory =  sqlSessionFactoryBuilder.build(in);
        //3.使用工厂生产dao对象
        userDao = new UserDaoImpl(sqlSessionFactory);
    }

    @After
    public void destory() throws IOException {
        in.close();
    }

    @Test
    public void testFindAll() throws IOException {
        //5.使用代理对象执行方法
        List<User> users = userDao.findAll();
        for(User user : users){
            System.out.println(user);
        }
    }

    /*测试根据id查找信息操作*/
    @Test
    public void testFindById() throws IOException {
        //5.执行保存方法
        User user = userDao.findById(43);
        System.out.println(user);
    }

    /*测试模糊查询操作*/
    @Test
    public void testFindByName() throws IOException {
        //5.执行保存方法
        List<User> users = userDao.findByName("%王%");
        //List<User> users = userDao.findByName(王);
        for(User user : users){
            System.out.println(user);
        }
    }

    /*测试Query条件查询操作*/
    @Test
    public void testFindByVo() throws IOException {
        QueryVo queryVo  = new QueryVo();
        User user = new User();
        user.setUsername("%王%");
        queryVo.setUser(user);
        //5.执行保存方法
        List<User> users = userDao.findUserByVo(queryVo);
        //List<User> users = userDao.findByName(王);
        for(User user1 : users){
            System.out.println(user1);
        }
    }

    /*测试Query条件查询操作*/
    @Test
    public void testFindByCondition() throws IOException {
        User user = new User();
        user.setUsername("老王");
        user.setSex("女");

        //5.执行保存方法
        List<User> users = userDao.findUserByCondition(user);
        for(User user1 : users){
            System.out.println(user1);
        }
    }

    /*测试foreach标签的使用*/
    @Test
    public void testFindInIds() throws IOException {
        QueryVo queryVo = new QueryVo();
        List<Integer> list = new ArrayList<Integer>();
        list.add(41);
        list.add(43);
        list.add(44);
        queryVo.setIds(list);

        //5.执行保存方法
        List<User> users = userDao.findUserInIds(queryVo);
        for(User user1 : users){
            System.out.println(user1);
        }
    }
}

~~~

