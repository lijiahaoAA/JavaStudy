### Mybatis中的缓存

***

##### 1.什么是缓存

​		存在于内存中的临时数据。

##### 2.为什么使用缓存

​		减少和数据库的交互次数，提高执行效率。

##### 3.什么样的数据可以使用缓存

​		经常查询并且不经常改变的数据，数据的正确与否对结果影响不大的。

##### 4.什么样的数据不适用于缓存

​	    经常改变的数据，数据的正确性对最终结果影响很大的（商品库存，银行汇率。。。）。

##### 5.Mybatis的一级缓存（存放的对象）

​	    指的是mybatis的SqlSession对象的缓存。当我们执行查询之后，查询的结果会同时存入到SqlSession为我们提供的一块区域中，该区域的结构是一个HashMap，key为hashcode+statementId+sql语句。Value为查询出来的结果集映射成的java对象。 当我们再次拿到同样的数据时，mybatis会先去SqlSession中查询是否有，有就拿出来用。

​		 当SqlSession对象消失时，mybatis的一级缓存也就消失了。

 		SqlSession执行insert、update、delete等操作commit后会清空该SQLSession缓存。（我的mybatis） 

##### 6.Mybatis基于xml开发的二级缓存（存放的数据）

​		指的是Myabtis中SqlSessionFactory对象的缓存，由同一个SqlSessionFactory对象创建的SqlSession共享其缓存。

###### **xml二级缓存使用步骤：**

- 让mybatis框架支持二级缓存，主配置文件SqlMapConfig配置。
- 让当前的配置映射文件支持二级缓存，IUserDao.xml配置文件配置。
- 让当前的操作支持二级缓存，在select标签中配置，

###### 实体类user

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

}

~~~

###### user接口

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

    //更新用户信息
    void updateUser(User user);

}

~~~

###### 主配置文件（省略日志和连接池）

~~~java
<?xml version="1.0" encoding="utf-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<!--mybatis的主配置文件-->
<configuration>

    <properties resource="jdbcConfig.properties"/>

    <!--配置支持二级缓存-->
    <settings>
        <setting name="cacheEnabled" value="true"/>
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

###### 接口映射配置文件

~~~java
<?xml version="1.0" encoding="utf-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.itheima.dao.IUserDao">
    <!--开启user支持二级缓存-->
    <cache/>

    <select id="findAll" resultType="user">
        select * from user
    </select>

    <!--根据id查找一个用户的信息-->
    <select id="findById" parameterType="INT" resultType="user" useCache="true">
        select * from user where id = #{id}
    </select>

    <!--更新用户的信息-->
    <select id="updateUser" parameterType="user">
        update user set username = #{username},address = #{address} where id = #{id}
    </select>
</mapper>
~~~

###### 测试

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

/**
 * @Author: lijiahao
 * @Description:
 * @Data: Create in 22:56 2020/2/22
 * @Modified By:
 */
public class SecondLevelCache {

    private InputStream in;
    private SqlSessionFactory sqlSessionFactory;

    @Before
    public void init() throws IOException {
        //1.读取配置文件
        in = Resources.getResourceAsStream("SqlMapConfig.xml");
        //2.创建SqlSessionFactory工厂
        SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
        sqlSessionFactory = sqlSessionFactoryBuilder.build(in);
    }

    @After
    public void destory() throws IOException {
        in.close();
    }

    /**
     * @Author Lijiahao
     * @Description 测试一级缓存，SqlSession中的缓存
     * @Date 17:03 2020/2/27
     * @Param []
     * @return void
     **/
    @Test
    public void testFirstLevelCache(){
        SqlSession sqlSession1 = sqlSessionFactory.openSession();
        IUserDao iUserDao1 = sqlSession1.getMapper(IUserDao.class);
        User user1 = iUserDao1.findById(43);
        System.out.println(user1);
        sqlSession1.close();

        SqlSession sqlSession2 = sqlSessionFactory.openSession();
        IUserDao iUserDao2 = sqlSession2.getMapper(IUserDao.class);
        User user2 = iUserDao2.findById(43);
        System.out.println(user2);
        sqlSession2.close();

        System.out.println(user1 == user2);
    }

}

~~~

##### 7.Mybatis基于注解开发的二级缓存

###### 注解二级缓存的使用步骤

- 主配置文件SqlMapConfig.xml开启支持二级缓存

  ~~~java
  <?xml version="1.0" encoding="utf-8" ?>
  <!DOCTYPE configuration
          PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
          "http://mybatis.org/dtd/mybatis-3-config.dtd">
  
  <!--mybatis的主配置文件-->
  <configuration>
  
      <properties resource="jdbcConfig.properties"/>
  
      <settings>
          <setting name="cacheEnabled" value="true"/>
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
  
      <!--指定dao接口所在位置-->
      <mappers>
          <package name="com.itheima.dao"></package>
      </mappers>
  </configuration>
  ~~~

- 接口IUserDao添加注解

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
  @CacheNamespace(blocking = true)
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

