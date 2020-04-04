#### Mybatis的多表（一对一，一对多）查询

***

>示例：用户和账户
>
>​			一个用户可以有多个账户（一对多）
>
>​			一个账户只能属于一个用户（一对一）

##### 解决办法

1. ##### **建立两张表**：用户表，账户表

   让用户表和账户表之间具备一对多的关系：需要使用外键在账户中添加。

2. ##### **建立两个实体类**：用户实体类和账户实体类

   让用户的实体类和账户的实体类能体现处出一对多的关系。

3. ##### **建立两个配置文件**：

   用户和帐户的配置文件

4. ##### **实现配置**：

   查询用户时，可以同时得到用户下所包含的账户信息。

   查询账户时，可以同时得到账户的所属用户信息。

---

##### 一对一，一对多公共代码部分

##### *接口类：IAccountDao,IUserDao*

##### IAccountDao

~~~java
package com.itheima.dao;

import com.itheima.domain.Account;
import com.itheima.domain.AccountUser;

import javax.security.auth.login.AccountException;
import java.util.List;

/**
 * @Author: lijiahao
 * @Description:
 * @Data: Create in 12:25 2020/2/26
 * @Modified By:
 */
public interface IAccountDao {
    /**
     * @Author Lijiahao
     * @Description 查询所有账户, 同时还要获取到当前账户的所有信息
     * @Date 16:32 2020/2/26
     * @Param []
     * @return java.util.List<com.itheima.domain.Account>
     **/
    List<Account> findAll();

    /**
     * @Author Lijiahao
     * @Description 查询所有帐户，并且带有用户名称和地址信息
     * @Date 16:43 2020/2/26
     * @Param []
     * @return java.util.List<com.itheima.domain.AccountUser>
     **/
    List<AccountUser> findAllAccount();
}

~~~

##### IUserDao

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

    //查询所有用户，同时获取到用户下所有账户的信息
    List<User> findAll();

    //根据id查询用户信息
    User findById(Integer userId);
}
~~~

##### *实体类：Account，User*

##### Account

~~~java
package com.itheima.domain;

import java.io.Serializable;

/**
 * @Author: lijiahao
 * @Description:
 * @Data: Create in 12:23 2020/2/26
 * @Modified By:
 */
public class Account implements Serializable {
    private Integer id;
    private Integer uid;
    private Double money;

    //从表实体应该包含一个主表实体的对象引用
    private User user;

    public User getUser() {
        return user;
    }

    public void setUser(User user) {
        this.user = user;
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public Integer getUid() {
        return uid;
    }

    public void setUid(Integer uid) {
        this.uid = uid;
    }

    public Double getMoney() {
        return money;
    }

    public void setMoney(Double money) {
        this.money = money;
    }

    @Override
    public String toString() {
        return "account{" +
                "id=" + id +
                ", uid=" + uid +
                ", money=" + money +
                '}';
    }
}

~~~

##### User

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

    //一对多关系映射，主表实体应该包含从表实体的集合引用
    private List<Account> accounts;

    public List<Account> getAccounts() {
        return accounts;
    }

    public void setAccounts(List<Account> accounts) {
        this.accounts = accounts;
    }

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

##### 资源文件：*jdbcConfig.properties*,log4j.properties,SqlMapConfig.xml

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

    <!--指定映射配置文件的位置，映射配置文件指的是每个dao独立的配置文件-->
    <mappers>
        <package name="com.itheima.dao"></package>
    </mappers>
</configuration>
~~~

---

#### 一对一

>帐户—用户      一个账号对应一个用户

##### IAccountDao映射配置文件

~~~java
<?xml version="1.0" encoding="utf-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.itheima.dao.IAccountDao">


    <!--定义封装account和user的ResultMap account为主表，user为从表，实现一对一-->
    <resultMap id="accountUserMap" type="account">
        <!--主键配置，property为属性名，column为数据库列名（sql语句起了别名as后面的,下方sql语句可以搜索到的）-->
        <!--aid为-->
        <id property="id" column="aid"></id>
        <result property="uid" column="uid"></result>
        <result property="money" column="money"></result>
        <!--一对一的关系映射，配置封装user的内容-->
        <association property="user" column="uid" javaType="user">
            <id property="id" column="id"></id>
            <result property="username" column="username"></result>
            <result property="address" column="address"></result>
            <result property="sex" column="sex"></result>
            <result property="birthday" column="birthday"></result>
        </association>
    </resultMap>
    <!--查询所有-->
    <select id="findAll" resultMap="accountUserMap">
        select u.* , a.id as aid , a.uid , a.money from account a , user u where u.id = a.uid
    </select>

    <!--查询所有账户并同时包含用户名和地址信息-->
    <select id="findAllAccount" resultType="com.itheima.domain.AccountUser">
        select a.*,u.username,u.address from account a , user u where u.id = a.uid
    </select>

</mapper>
~~~

##### 实体类AccountUser

~~~java
package com.itheima.domain;

/**
 * @Author: lijiahao
 * @Description:
 * @Data: Create in 16:36 2020/2/26
 * @Modified By:
 */
public class AccountUser extends Account {

    private String username;
    private String address;

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    @Override
    public String toString() {
        return super.toString()+"       AccountUser{" +
                "username='" + username + '\'' +
                ", address='" + address + '\'' +
                '}';
    }
}
~~~

##### 测试类

~~~java
package com.itheima;

import com.itheima.dao.IAccountDao;
import com.itheima.dao.IUserDao;
import com.itheima.domain.Account;
import com.itheima.domain.AccountUser;
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
import java.util.List;

/**
 * @Author: lijiahao
 * @Description:
 * @Data: Create in 22:56 2020/2/22
 * @Modified By:
 */
public class AccountTest {

    private InputStream in;
    private SqlSession sqlSession;
    private IAccountDao accountDao;

    @Before
    public void init() throws IOException {
        //1.读取配置文件
        in = Resources.getResourceAsStream("SqlMapConfig.xml");
        //2.创建SqlSessionFactory工厂
        SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
        SqlSessionFactory sqlSessionFactory = sqlSessionFactoryBuilder.build(in);
        sqlSession = sqlSessionFactory.openSession(true);
        //3.使用工厂生产dao对象
        accountDao = sqlSession.getMapper(IAccountDao.class);
    }

    @After
    public void destory() throws IOException {
        in.close();
    }

    @Test
    public void testFindAll() throws IOException {
        //5.使用代理对象执行方法
        List<Account> accounts = accountDao.findAll();
        for (Account account : accounts) {
            System.out.println(account);
            System.out.println(account.getUser());
        }
    }

    @Test
    public void testFindAllAccountUser() throws IOException {
        //5.使用代理对象执行方法
        List<AccountUser> accounts = accountDao.findAllAccount();
        for (Account account : accounts) {
            System.out.println(account);
        }
    }

}
~~~

#### 一对多

>用户—帐户   一个用户可以拥有多个账户  实现一对多的关系

##### 映射配置文件

~~~java
<?xml version="1.0" encoding="utf-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.itheima.dao.IUserDao">

    <!--定义user的resultMap user为主表，account为从表，构成一对多关系-->
    <resultMap id="userAccountMap" type="user">
        <id property="id" column="id"></id>
        <result property="username" column="username"></result>
        <result property="address" column="address"></result>
        <result property="sex" column="sex"></result>
        <result property="birthday" column="birthday"></result>
        <!--配置user对象中accounts集合的映射-->
        <collection property="accounts" ofType="account">
            <id property="id" column="aid"></id>
            <result property="uid" column="uid"></result>
            <result property="money" column="money"></result>
        </collection>
    </resultMap>
    <select id="findAll" resultMap="userAccountMap">
        select * from user u left outer join account a on u.id = a.uid
    </select>


    <!--根据id查找一个用户的信息-->
    <select id="findById" parameterType="INT" resultType="user">
        select * from user where id = #{userid}
    </select>

</mapper>
~~~

##### 测试类

~~~java
package com.itheima;

import com.itheima.dao.IAccountDao;
import com.itheima.dao.IUserDao;
import com.itheima.domain.Account;
import com.itheima.domain.AccountUser;
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
import java.util.List;

/**
 * @Author: lijiahao
 * @Description:
 * @Data: Create in 22:56 2020/2/22
 * @Modified By:
 */
public class UserTest {

    private InputStream in;
    private SqlSession sqlSession;
    private IUserDao userDao;

    @Before
    public void init() throws IOException {
        //1.读取配置文件
        in = Resources.getResourceAsStream("SqlMapConfig.xml");
        //2.创建SqlSessionFactory工厂
        SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
        SqlSessionFactory sqlSessionFactory = sqlSessionFactoryBuilder.build(in);
        sqlSession = sqlSessionFactory.openSession(true);
        //3.使用工厂生产dao对象
        userDao = sqlSession.getMapper(IUserDao.class);
    }

    @After
    public void destory() throws IOException {
        in.close();
    }

    @Test
    public void testFindAll() throws IOException {
        //5.使用代理对象执行方法
        List<User> users = userDao.findAll();
        for (User user : users) {
            System.out.println(user);
            System.out.println(user.getAccounts());
        }
    }
}
~~~

