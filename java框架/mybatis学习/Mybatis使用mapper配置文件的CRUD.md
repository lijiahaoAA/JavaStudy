## Mybatis使用映射配置文件的CRUD操作

#### 1.实体类

~~~java
public class User implements Serializable {
    private Integer id;
    private String username;
    private Date birthday;
    private String sex;
    private String address;
	//set,get,tostring方法省略
}
~~~

#### 2.dao接口

~~~java
public interface IUserDao {

    List<User> findAll();

    void saveUser(User user);

    void updateUser(User user);

    void deleteUser(Integer userId);

    //根据id查询用户信息
    User findById(Integer userId);

    //根据名称模糊查询用户信息
    List<User> findByName(String username);

    //查询总用户数
    int findTotal();
}
~~~

#### 3.主配置文件

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

#### 4.映射配置文件

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
    <select id="findAll" resultType="com.itheima.domain.User">
        select * from user;
    </select>

    <!--保存用户-->
    <insert id="saveUser" parameterType="com.itheima.domain.User">
        <!-- 配置插入操作后，获取插入数据的自增id值 -->
        <selectKey keyProperty="id" keyColumn="id" resultType="int" order="AFTER">
            select last_insert_id();
        </selectKey>
        insert into user (username,address,sex,birthday) values (#{username},#{address},#{sex},#{birthday});
    </insert>

    <!--更新用户-->
    <update id="updateUser" parameterType="com.itheima.domain.User">
        update user set username = #{username},address = #{address},sex = #{sex},birthday = #{birthday} where id = #{id};
    </update>

    <!--删除用户-->
    <delete id="deleteUser" parameterType="java.lang.Integer">
        delete from user where id = #{uid}
    </delete>

    <!--根据id查找一个用户的信息-->
    <select id="findById" parameterType="INT" resultType="com.itheima.domain.User">
        select * from user where id = #{userid}
    </select>

    <!--根据名称模糊查询-->
    <select id="findByName" parameterType="string" resultType="com.itheima.domain.User">
        select * from user where username like #{name}
        /* select * from user where username like '%${value}%' */
    </select>

    <!--获取用户的总记录条数-->
    <select id="findTotal" resultType="int">
        select count(*) from user;
    </select>
</mapper>
~~~

#### 5.日志配置

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

#### 6.测试类

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

import javax.annotation.Resource;
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
public class test {

    private InputStream in;
    private SqlSession sqlSession;
    private IUserDao userDao;

    @Before
    public void init() throws IOException {
        //1.读取配置文件
        in = Resources.getResourceAsStream("SqlMapConfig.xml");
        //2.创建SqlSessionFactory工厂
        SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
        SqlSessionFactory sqlSessionFactory =  sqlSessionFactoryBuilder.build(in);
        //3.使用工厂生产SqlSession对象
        sqlSession = sqlSessionFactory.openSession();
        //4.使用SqlSession创建dao接口的代理对象
        userDao = sqlSession.getMapper(IUserDao.class);
    }

    @After
    public void destory() throws IOException {
        //提交事务
        sqlSession.commit();
        //6.释放资源
        sqlSession.close();
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

    /*测试保存操作*/
    @Test
    public void testSave() throws IOException {
        User user1 = new User();
        user1.setUsername("mybatis last insert id saveuser");
        user1.setAddress("北京市");
        user1.setSex("女");
        user1.setBirthday(new Date());

        System.out.println("保存操作之前："+user1);
        //5.执行保存方法
        userDao.saveUser(user1);

        System.out.println("保存操作之后："+user1);
    }

    /*测试更新操作*/
    @Test
    public void testUpdate() throws IOException {
        User user1 = new User();
        user1.setId(47);
        user1.setUsername("asdasdasd");
        user1.setAddress("北京市");
        user1.setSex("女");
        user1.setBirthday(new Date());

        //5.执行保存方法
        userDao.updateUser(user1);
    }

    /*测试删除操作*/
    @Test
    public void testDelete() throws IOException {
        //5.执行保存方法
        userDao.deleteUser(42);
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

    /*测试查询总记录条数*/
    @Test
    public void testFindTotal() throws IOException {
        //5.执行保存方法
        int count  = userDao.findTotal();
        System.out.println(count);
    }
}

~~~

