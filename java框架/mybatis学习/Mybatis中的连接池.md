### Mybatis中的连接池

***

####   三种配置方式

- 配置位置

  主配置文件中SqlMapConfig中的dataSource标签，type属性表示使用何种连接池方式。

- type属性的取值

  - POOLED：采用传统的javax.sql.DataSource规范中的连接池，mybatis中有针对规范的实现。
  - UNPOOLED：采用传统的获取连接的方法，虽然也实现了javax.sql.DataSource接口，但是并没有实现池的思想。
  - JNDI： [SUN公司](https://baike.baidu.com/item/SUN公司)提供的一种标准的Java命名系统接口 ，采用服务器提供的JNDI技术实现，来获取DataSource对象，不同的服务器所能拿到的DataSource对象是不同的。但是如果不是web工程或者maven的war工程，是不能使用的。