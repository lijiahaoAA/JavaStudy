#### Mybatis注解开发多表一对一，一对多

***

#### 一对一

>示例：帐户和用户的对应关系为，多个帐户对应一个用户，在实际开发中，查询一个帐户并同时查询该账户所属的用户信息，即立即加载且在mybatis中表现为一对一的关系。应为账户的实体类Account中添加User为一个属性。

##### 实体类：User，Account

###### User

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
    private Integer userId;
    private String userName;
    private Date userBirthday;
    private String userSex;
    private String userAddress;

    public Integer getUserId() {
        return userId;
    }

    public void setUserId(Integer userId) {
        this.userId = userId;
    }

    public String getUserName() {
        return userName;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }

    public Date getUserBirthday() {
        return userBirthday;
    }

    public void setUserBirthday(Date userBirthday) {
        this.userBirthday = userBirthday;
    }

    public String getUserSex() {
        return userSex;
    }

    public void setUserSex(String userSex) {
        this.userSex = userSex;
    }

    public String getUserAddress() {
        return userAddress;
    }

    public void setUserAddress(String userAddress) {
        this.userAddress = userAddress;
    }

    @Override
    public String toString() {
        return "User{" +
                "userId=" + userId +
                ", userName='" + userName + '\'' +
                ", userBirthday=" + userBirthday +
                ", userSex='" + userSex + '\'' +
                ", userAddress='" + userAddress + '\'' +
                '}';
    }
}

~~~

###### Account

~~~java
package com.itheima.domain;

import java.io.Serializable;

/**
 * @Author: lijiahao
 * @Description:
 * @Data: Create in 14:09 2020/2/28
 * @Modified By:
 */
public class Account implements Serializable {

    private Integer id;
    private Integer uid;
    private Double money;

    //多对一，mybatis中称之为一对一的映射：一个账户只能属于一个用户
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
        return "Account{" +
                "id=" + id +
                ", uid=" + uid +
                ", money=" + money +
                '}';
    }
}

~~~

##### 接口：IUserDao，IAccountDao

###### IUserDao

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

###### IAccountDao

~~~java
package com.itheima.dao;

import com.itheima.domain.Account;
import org.apache.ibatis.annotations.One;
import org.apache.ibatis.annotations.Result;
import org.apache.ibatis.annotations.Results;
import org.apache.ibatis.annotations.Select;
import org.apache.ibatis.mapping.FetchType;

import java.util.List;

/**
 * @Author: lijiahao
 * @Description:
 * @Data: Create in 14:10 2020/2/28
 * @Modified By:
 */
public interface IAccountDao {

    /**
     * @Author Lijiahao
     * @Description 查询所有帐户，并且获取每个帐户所属的用户信息
     * @Date 14:11 2020/2/28
     * @Param []
     * @return List<Account>
     **/
    @Select("select * from account")
    @Results(id="accountMap",value = {
            //封装account
            @Result(id = true , column = "id" ,property = "id"),
            @Result(column = "uid" ,property = "uid"),
            @Result(column = "money" ,property = "money"),
            //封装user，account-user 是一对一关系
            @Result(property = "user",column = "uid",one = @One(select="com.itheima.dao.IUserDao.findById",fetchType = FetchType.EAGER))
    })
    List<Account> findAll();
}

~~~

##### 配置文件：主配置文件，日志，连接池

###### 主配置文件：SqlMapConfig.xml

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

##### 测试

~~~java
package com.itheima;

import com.itheima.dao.IAccountDao;
import com.itheima.dao.IUserDao;
import com.itheima.domain.Account;
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
    private IAccountDao iAccountDao;
    private SqlSessionFactory sqlSessionFactory;

    @Before
    public void init() throws Exception {
        in = Resources.getResourceAsStream("SqlMapConfig.xml");
        sqlSessionFactory = new SqlSessionFactoryBuilder().build(in);
        sqlSession = sqlSessionFactory.openSession();
        iAccountDao = sqlSession.getMapper(IAccountDao.class);
    }

    @After
    public void destroy() throws IOException {
        sqlSession.commit();
        sqlSession.close();
        in.close();
    }

    @Test
    public void testFindAll() {
        List<Account> accounts = iAccountDao.findAll();
        for (Account account : accounts) {
            System.out.println("获取每个用户的信息");
            System.out.println(account);
            System.out.println(account.getUser());
        }

    }
}
~~~

***

#### 一对多

> 示例：用户user-帐户Account是一对多的关系，在mybatis中也是一对多的关系，即使用延迟加载且在user类中添加实体类Account的属性。

##### 实体类：User，Account

###### User

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
    private Integer userId;
    private String userName;
    private Date userBirthday;
    private String userSex;
    private String userAddress;

    //一对多关系映射，一个用户对应多个账户
    private List<Account> accounts;

    public List<Account> getAccounts() {
        return accounts;
    }

    public void setAccounts(List<Account> accounts) {
        this.accounts = accounts;
    }

    public Integer getUserId() {
        return userId;
    }

    public void setUserId(Integer userId) {
        this.userId = userId;
    }

    public String getUserName() {
        return userName;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }

    public Date getUserBirthday() {
        return userBirthday;
    }

    public void setUserBirthday(Date userBirthday) {
        this.userBirthday = userBirthday;
    }

    public String getUserSex() {
        return userSex;
    }

    public void setUserSex(String userSex) {
        this.userSex = userSex;
    }

    public String getUserAddress() {
        return userAddress;
    }

    public void setUserAddress(String userAddress) {
        this.userAddress = userAddress;
    }

    @Override
    public String toString() {
        return "User{" +
                "userId=" + userId +
                ", userName='" + userName + '\'' +
                ", userBirthday=" + userBirthday +
                ", userSex='" + userSex + '\'' +
                ", userAddress='" + userAddress + '\'' +
                '}';
    }
}

~~~

###### Account

~~~java
package com.itheima.domain;

import java.io.Serializable;

/**
 * @Author: lijiahao
 * @Description:
 * @Data: Create in 14:09 2020/2/28
 * @Modified By:
 */
public class Account implements Serializable {

    private Integer id;
    private Integer uid;
    private Double money;

    //多对一，mybatis中称之为一对一的映射：一个账户只能属于一个用户
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
        return "Account{" +
                "id=" + id +
                ", uid=" + uid +
                ", money=" + money +
                '}';
    }
}

~~~

##### 接口：IUserDao，IAccountDao

###### IUserDao

~~~java
package com.itheima.dao;

import com.itheima.domain.User;
import org.apache.ibatis.annotations.*;
import org.apache.ibatis.mapping.FetchType;

import javax.xml.parsers.FactoryConfigurationError;
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
            //配置user-account的一对多的配置
            @Result(column = "id",property = "accounts",
                    many=@Many(select = "com.itheima.dao.IAccountDao.findAccountByUid",
                            fetchType= FetchType.LAZY))
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

###### IAccountDao

~~~java
package com.itheima.dao;

import com.itheima.domain.Account;
import org.apache.ibatis.annotations.One;
import org.apache.ibatis.annotations.Result;
import org.apache.ibatis.annotations.Results;
import org.apache.ibatis.annotations.Select;
import org.apache.ibatis.mapping.FetchType;

import java.util.List;

/**
 * @Author: lijiahao
 * @Description:
 * @Data: Create in 14:10 2020/2/28
 * @Modified By:
 */
public interface IAccountDao {

    /**
     * @Author Lijiahao
     * @Description 查询所有帐户，并且获取每个帐户所属的用户信息
     * @Date 14:11 2020/2/28
     * @Param []
     * @return List<Account>
     **/
    @Select("select * from account")
    @Results(id="accountMap",value = {
            //封装account
            @Result(id = true , column = "id" ,property = "id"),
            @Result(column = "uid" ,property = "uid"),
            @Result(column = "money" ,property = "money"),
            //封装user，account-user 是一对一关系
            @Result(property = "user",column = "uid",
                    one = @One(select="com.itheima.dao.IUserDao.findById",
                            fetchType = FetchType.EAGER))
    })
    List<Account> findAll();

    @Select("select * from account where uid = #{userId}")
    List<Account> findAccountByUid(Integer userId);
}
~~~

##### 测试

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
public class UserTest {
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
        for(User user : users) {
            System.out.println(user);
            System.out.println(user.getAccounts());
        }
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

