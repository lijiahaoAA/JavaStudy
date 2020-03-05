### Mybatis的多表（多对多）查询

***

>示例：用户和角色
>
>​			一个用户可以有多个角色
>
>​			一个角色可以赋予多个用户

#### 解决办法

1. ###### **建立两张表：用户表，角色表**

   让用户表和角色表具有多对多的关系。需要使用中间表，中间表包含各自的主键，在中间表中是外键。

2. ###### **建立两个实体类：用户实体类和角色实体类**

   让用户和角色的实体类能体现出来多对多的关系

   各自包含对方一个集合引用

3. ###### **建立两个配置文件**

   用户的配置文件

   角色的配置文件

4. ###### **实现配置**

   当我们查询用户时，可以同时得到用户所包含的角色信息。

   当我们查询角色时，可以同时得到角色所赋予的用户信息。

***

#### 多对多（可以理解为两个实体类互相一对多）

##### 接口：IRoleDao，IUserDao

##### IRoleDao

~~~java
public interface IRoleDao {
    //查询所有角色
    List<Role> findAll();
}
~~~

##### IUserDao

~~~java
public interface IUserDao {

    //查询所有用户，同时获取到用户下所有账户的信息
    List<User> findAll();

    //根据id查询用户信息
    User findById(Integer userId);
    
}
~~~

##### 实体类：Role，User

##### Role

~~~java
package com.itheima.domain;

import java.io.Serializable;
import java.util.List;

/**
 * @Author: lijiahao
 * @Description:
 * @Data: Create in 21:31 2020/2/26
 * @Modified By:
 */
public class Role implements Serializable {
    private Integer roleId;
    private String roleName;
    private String roleDesc;

    //多对多的关系映射：一个角色可以对应多个用户
    private List<User> users;

    public List<User> getUsers() {
        return users;
    }

    public void setUsers(List<User> users) {
        this.users = users;
    }

    public Integer getRoleId() {
        return roleId;
    }

    public void setRoleId(Integer roleId) {
        this.roleId = roleId;
    }

    public String getRoleName() {
        return roleName;
    }

    public void setRoleName(String roleName) {
        this.roleName = roleName;
    }

    public String getRoleDesc() {
        return roleDesc;
    }

    public void setRoleDesc(String roleDesc) {
        this.roleDesc = roleDesc;
    }

    @Override
    public String toString() {
        return "Role{" +
                "roleId=" + roleId +
                ", roleName='" + roleName + '\'' +
                ", roleDesc='" + roleDesc + '\'' +
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

    //添加多对多的映射关系：一个用户可以对应多个角色
    private List<Role> roles;

    public List<Role> getRoles() {
        return roles;
    }

    public void setRoles(List<Role> roles) {
        this.roles = roles;
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

##### 资源文件：连接池，日志，主配置文件，映射配置文件

##### 连接池jdbcConfig.properties

~~~java
jdbc.driver=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/eesy?serverTimezone=Asia/Shanghai&characterEncoding=utf8&useSSL=false
jdbc.username=root
jdbc.password=123456
~~~

##### 日志log4j

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

##### 主配置文件SqlMapConfig.xml

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

##### user的映射配置文件

~~~java
<?xml version="1.0" encoding="utf-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.itheima.dao.IUserDao">

    <!--定义user的resultMap user为主表，account为从表，构成一对多关系-->
    <resultMap id="userMap" type="user">
        <id property="id" column="id"></id>
        <result property="username" column="username"></result>
        <result property="address" column="address"></result>
        <result property="sex" column="sex"></result>
        <result property="birthday" column="birthday"></result>
        <!--配置user到role的配置 一个user对应多个role-->
        <collection property="roles" ofType="role">
            <id property="roleId" column="id"></id>
            <result property="roleName" column="role_name"></result>
            <result property="roleDesc" column="role_desc"></result>
        </collection>
    </resultMap>
    <select id="findAll" resultMap="userMap">
        select u.*,r.id as rid,r.role_name,r.role_desc from user u
        left outer join user_role ur on u.id = ur.uid
        left outer join role r on r.id = ur.rid
    </select>


    <!--根据id查找一个用户的信息-->
    <select id="findById" parameterType="INT" resultType="user">
        select * from user where id = #{userid}
    </select>

</mapper>
~~~

##### Role的映射配置文件

~~~java
<?xml version="1.0" encoding="utf-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.itheima.dao.IRoleDao">

    <!--定义role的resultMap-->
    <resultMap id="roleMap" type="role">
        <id property="roleId" column="id"></id>
        <result property="roleName" column="role_name"></result>
        <result property="roleDesc" column="role_desc"></result>
        <collection property="users" ofType="user">
            <id property="id" column="id"></id>
            <result property="username" column="username"></result>
            <result property="address" column="address"></result>
            <result property="sex" column="sex"></result>
            <result property="birthday" column="birthday"></result>
        </collection>
    </resultMap>
    <!--查询所有-->
    <select id="findAll" resultMap="roleMap">
        select u.*,r.id as rid,r.role_name,r.role_desc from role r
        left outer join user_role ur on r.id = ur.rid
        left outer join user u on u.id = ur.uid
    </select>
</mapper>
~~~

- ##### 测试类：user-role的一对多测试，role-user的一对多测试

  ##### user-role的一对多测试

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
              System.out.println(user.getRoles());
          }
      }
  
  }
  ~~~

  ##### role-user的一对多测试

  ~~~java
  package com.itheima;
  
  import com.itheima.dao.IRoleDao;
  import com.itheima.dao.IUserDao;
  import com.itheima.domain.Role;
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
  public class RoleTest {
  
      private InputStream in;
      private SqlSession sqlSession;
      private IRoleDao roleDao;
  
      @Before
      public void init() throws IOException {
          //1.读取配置文件
          in = Resources.getResourceAsStream("SqlMapConfig.xml");
          //2.创建SqlSessionFactory工厂
          SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
          SqlSessionFactory sqlSessionFactory = sqlSessionFactoryBuilder.build(in);
          sqlSession = sqlSessionFactory.openSession(true);
          //3.使用工厂生产dao对象
          roleDao = sqlSession.getMapper(IRoleDao.class);
      }
  
      @After
      public void destory() throws IOException {
          in.close();
      }
  
      @Test
      public void testFindAll() throws IOException {
          //5.使用代理对象执行方法
          List<Role> roles = roleDao.findAll();
          for (Role role : roles) {
              System.out.println(role);
          }
      }
  
  }
  ~~~

  



