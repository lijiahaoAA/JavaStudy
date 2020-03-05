#### Mybatis的延迟加载和立即加载

***

>示例：在一对多中，当我们有一个用户，他有100个帐户
>
>​			问题1：在查询用户时，要不要把关联的账户查出来？
>
>​			问题2：在查询账户时，要不要把关联的用户信息查出来？

##### 问题1：在查询用户时，要不要把关联的账户查出来？

​        用户和帐户之间的关系应该是一对多的关系，一个用户对应多个账户，如果在查询用户时，同时查询到他关联的帐户，那么这些查询到的账户信息就会占据很大的内存，所以应该是是么时候使用，是么时候才查询帐户信息。

##### 问题2：在查询账户时，要不要把关联的用户信息查出来？

​        帐户和用户之间的关系应该是一对一的，一个帐户应该对应一个用户，如果查询账户时，账户的所属用户信息应该是随着一起查询出来的，且用户信息占据内存很小。

##### 延迟加载：（应用于一对多，多对多，即关联的对象为“多”时）

​        在真正使用数据时才发起查询，不用的时候不查询，按需加载（懒加载）。

##### 立即加载：（应用于多对一，一对一，即关联的对象为“一”时）

​        不管用不用，在调用方法的时候就立即执行，马上发起查询。

***

#### 一对一实现延迟：account-user 的一对一延迟加载

##### 帐户account实体类

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

##### user的接口

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

##### 主配置文件

~~~java
<?xml version="1.0" encoding="utf-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<!--mybatis的主配置文件-->
<configuration>
    <!--配置连接池-->
    <properties resource="jdbcConfig.properties"/>

    <!--配置参数-->
    <settings>
        <!--开启mybatis延迟加载，默认为false-->
        <setting name="lazyLoadingEnabled" value="true"/>
        <!--开启立即加载，mybatis3.4.1以上默认为false-->
        <setting name="aggressiveLazyLoading" value="false"/>
    </settings>
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

##### 帐户映射配置文件IAccountDao.xml

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
        <id property="id" column="id"></id>
        <result property="uid" column="uid"></result>
        <result property="money" column="money"></result>
        <!--一对一的关系映射，配置封装user的内容
        select属性指定的内容：查询用户的唯一标识
        column属性指定的内容：用户根据id查询时，所需要的参数的值-->
        <association property="user" column="uid" javaType="user" select="com.itheima.dao.IUserDao.findById">

        </association>
    </resultMap>
    <!--查询所有-->
    <select id="findAll" resultMap="accountUserMap">
        select * from account
    </select>

</mapper>
~~~

##### 测试类

~~~java
package com.itheima;

import com.itheima.dao.IAccountDao;
import com.itheima.domain.Account;
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


}
~~~

***

#### 一对多实现延迟加载：user-account 的一对多的延迟加载

##### 用户user实体类

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

##### account的接口

~~~java
package com.itheima.dao;

import com.itheima.domain.Account;

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

    //根据用户id查询帐户信息
    List<Account> findAccountByUid(Integer uid);

}
~~~

##### 主配置文件

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

##### 用户映射配置文件IUserDao.xml

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
        <!--配置user对象中accounts集合的映射
    	select 后面即为延迟 后的查询语句，根据user中的id查询-->
        <collection property="accounts" ofType="account" select="com.itheima.dao.IAccountDao.findAccountByUid" column="id">
        </collection>
    </resultMap>
    <select id="findAll" resultMap="userAccountMap">
        select * from user
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

