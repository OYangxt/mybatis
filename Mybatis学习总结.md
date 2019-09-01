# Mybatis浅析

+++++

# 1 JDBC

## 1.1 引入mysql依赖

```xml
<!--引入mysql依赖包-->
<dependency>    
    <groupId>mysql</groupId>   
    <artifactId>mysql-connector-java</artifactId>    
    <version>5.1.32</version>
 </dependency>
```

## 1.2 创建数据库

### 1.2.1 创建数据库&建表

```sql
CREATE DATABASE mybatisdemo;
USE mybatisdemo;
DROP TABLE IF EXISTS tb_user;
CREATE TABLE tb_user
(  
    id         bigint AUTO_INCREMENT NOT NULL,  
    user_name  varchar(32) DEFAULT NULL,  
    password   varchar(32) DEFAULT NULL,  
    name       VARCHAR(32) DEFAULT NULL,  
    age        int(10)     DEFAULT NULL, 
    sex        int(2)      DEFAULT NULL, 
    birthday   date        DEFAULT NULL,  
    gmt_create datetime    DEFAULT NULL,  
    gmt_update datetime    DEFAULT NULL,  
    PRIMARY KEY (id)
) ENGINE = InnoDB  DEFAULT CHARSET = utf8;

INSERT INTO tb_user (user_name, password, name, age, sex, birthday, gmt_create, gmt_update)
VALUES ('pc', '123456', '鹏程', '26', '1', '1993-09-02', sysdate(), sysdate());
INSERT INTO tb_user (user_name, password, name, age, sex, birthday, gmt_create, gmt_update)
VALUES ('hj', '123456', '静静', '22', '1', '1993-09-05', sysdate(), sysdate());
```

### 1.2.3  插入数据

```sql
INSERT INTO tb_user (user_name, password, name, age, sex, birthday, gmt_create, gmt_update)
VALUES ('pc', '123456', '鹏程', '26', '1', '1993-09-02', sysdate(), sysdate());

INSERT INTO tb_user (user_name, password, name, age, sex, birthday, gmt_create, gmt_update)
VALUES ('hj', '123456', '静静', '22', '1', '1993-09-05', sysdate(), sysdate());
```

### 1.2.4  Date 、Datetime、Timestamp和Time的区别

**Date**

|   名称   |                         解释                         |
| :------: | :--------------------------------------------------: |
| 显示格式 |                      YYYY-MM-DD                      |
| 显示范围 |               1601-01-01 到 9999-01-0                |
| 应用场景 | 当业务需求中只需要**精确到天**时，可以用这个时间格式 |
| 后台取值 |           @JSONField(format=”yyyy-MM-dd”)            |
**Datetime**

|   名称   |                         解释                         |
| :------: | :--------------------------------------------------: |
| 显示格式 |                 YYYY-MM-DD HH:mm:ss                  |
| 显示范围 |      1601-01-01 00:00:00 到 9999-12-31 23:59:59      |
| 应用场景 | 当业务需求中只需要**精确到秒时**，可以用这个时间格式 |
| 后台取值 |           @JSONField(format=”yyyy-MM-dd”)            |

**Datetime**

|   名称   |                         解释                         |
| :------: | :--------------------------------------------------: |
| 显示格式 |                 YYYY-MM-DD HH:mm:ss                  |
| 显示范围 |      1601-01-01 00:00:00 到 9999-12-31 23:59:59      |
| 应用场景 | 当业务需求中只需要**精确到秒时**，可以用这个时间格式 |
| 后台取值 |           @JSONField(format=”yyyy-MM-dd”)            |

****

**Timestamp**

|   名称   |                             解释                             |
| :------: | :----------------------------------------------------------: |
| 显示格式 |                     YYYY-MM-DD HH:mm:ss                      |
| 显示范围 |          1601-01-01 00:00:00 到 9999-12-31 23:59:59          |
| 应用场景 | 当业务需求中只需要**精确到秒或者毫秒时**，可以用这个时间格式 |
| 后台取值 |               @JSONField(format=”yyyy-MM-dd”)                |

****

**Time**

|   名称   |                            解释                             |
| :------: | :---------------------------------------------------------: |
| 显示格式 |                          HH:mm:ss                           |
| 显示范围 |                   1 00:00:00 到 23:59:59                    |
| 应用场景 |    当业务需求中只需要**每天的时间**，可以用这个时间格式     |
| 后台取值 | @JSONField(format=”HH:mm”)或@JSONField(format=”HH:mm:ss”)） |

DateTime和TimeStamp最大的区别只是用的场景不同，如果你的应用 是用于不同时区（就是国内和国外同时使用），这时候如果用dateTime就会出现各种各样的问题，但是如果使用TimeStamp就不会出现这种时差的问题。

## 1.3 JDBC测试类

### 1.3.1 Jdbc.java

```java
package com.test.mybatis.jdbc;

import java.sql.*;

/**
 * @author 20190712713
 * @date 2019/8/28 11:27
 */
public class Jdbc {
    public static void main(String[] args) {
        Connection connection = null;
        PreparedStatement preparedStatement = null;
        ResultSet resultSet = null;

        try {
            //加载驱动
            Class.forName("com.mysql.jdbc.Driver");
            //获取连接
            connection = DriverManager.getConnection("jdbc:mysql://127.0.0.1:3306/mybatisdemo?characterEncoding=utf-8", "root", "123456");
            //创建sql语句
            String sql = "SELECT user_name,name,age, birthday FROM tb_user WHERE id = ?";
            preparedStatement = connection.prepareStatement(sql);
            //设置参数
            preparedStatement.setLong(1,1L);
            //执行查询
            resultSet = preparedStatement.executeQuery();
            //处理结果
            while (resultSet.next()){
                System.out.println(resultSet.getString("user_name"));
                System.out.println(resultSet.getString("name"));
                System.out.println(resultSet.getString("age"));
                System.out.println(resultSet.getString("birthday"));
            }
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            try {
                resultSet.close();
                preparedStatement.close();
                connection.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
}

```

### 1.3.2 测试结果

![JDBC测试结果](\JDBC测试结果.jpg)

## 1.4 JDBC问题分析

存在硬编码问题，耦合度高，不以利于系统维护：

（1） 将sql语句硬编码到java代码中；

设想：将sql语句配置在xml配置文件中，即使sql变化，不需要对java代码进行重新编译。

（2）preparedStatement中占位符号位置和设置参数值，硬编码在java代码中；

设想：将sql语句及占位符号和参数全部配置在xml中。

（3）resultSet结果处理，存在硬编码；

设想：将查询的结果集，自动映射成java对象。

# 2 Mybatis

## 2.1 基本介绍

​        **MyBatis** 是一款优秀的持久层框架，它支持定制化 SQL、存储过程以及高级映射。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。MyBatis 可以使用简单的 XML 或注解来配置和映射原生信息，将接口和 Java 的 POJOs(Plain Ordinary Java Object,普通的 Java对象)映射成数据库中的记录。

## 2.2 Mybatis整体架构

![Mybatis整体架构](C:\Users\20190712713\Desktop\Springdoc\mybatis架构.png)

1  xxMapper.xml：可以有多个，指定了sql参数和结果集的类型；

2  mybatis-config.xml：整体配置文件，只有一个，用于引入数据源，环境设置等等）；

3  SqlSessionFactory：SqlSession工厂，用于创建SqlSession；

​                                         通过new SqlSessionFactoryBuilder(输入流)创建；

4 SqlSession：以sqlSessionFactory.openSession()的方式获取SqlSession的对象；

5 Executor(执行器)：SqlSession对象通过Executor执行xxMapper.xml中配置的sql语句，执行CRUD；

6 Mapped Statement：其输入（Input）即SQL输入参数，可以为HashMap，简单数据类型或者POJO(Plain Old Java Object)，处理过程（Process）中查询数据库，返回查询结果，作为输出（Output）。

# 3 Mybatis实现方式一：通过创建dao实现类

## 3.1 引入相关依赖（pom.xml）

```xml
<!--引入mybatis依赖-->
<dependency>   
    <groupId>org.mybatis</groupId>   
    <artifactId>mybatis</artifactId>    
    <version>3.2.8</version>
</dependency>
<!--引入日志依赖-->
<dependency>
     <groupId>org.slf4j</groupId>
     <artifactId>slf4j-log4j12</artifactId>
     <version>1.7.5</version>
</dependency>
  <!--引入单元测试依赖-->
  <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.12</version>
   </dependency>
```

## 3.2 基本配置文件

### 3.2.1 mybatis-config.xml

基本配置中，主要引入数据源

``` xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration        
PUBLIC "-//mybatis.org//DTD Config 3.0//EN"        
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<!-- 根标签 -->
<configuration>    
    <properties>        
        <property name="driver" value="com.mysql.jdbc.Driver"/>        
        <property name="url"  value="jdbc:mysql://127.0.0.1:3306/mybatisdemo"/>                   <property name="username" value="root"/>        
        <property name="password" value="123456"/>    
    </properties>    
    <!-- 可以配置多个，default：指定采用哪个环境 -->   
    <environments default="test">       
        <!-- id：唯一标识 -->        
        <environment id="test">            
            <!-- 事务管理器，JDBC类型的事务管理器 -->            
            <transactionManager type="JDBC"/>            
            <!-- 数据源，池类型的数据源 -->            
            <dataSource type="POOLED">                
                <property name="driver" value="com.mysql.jdbc.Driver"/>                                   <property name="url" value="jdbc:mysql://127.0.0.1:3306/mybatis-110"/>                   <property name="username" value="root"/>                
                <property name="password" value="123456"/>          
            </dataSource>        
        </environment>       
        <environment id="development">           
            <!-- 事务管理器，JDBC类型的事务管理器 -->           
            <transactionManager type="JDBC"/>           
            <!-- 数据源，池类型的数据源 -->            
            <dataSource type="POOLED">               
                <property name="driver" value="${driver}"/>
                <!-- 配置了properties，所以可以直接引用 -->                
                <property name="url" value="${url}"/>               
                <property name="username" value="${username}"/>                
                <property name="password" value="${password}"/>            
            </dataSource>        
        </environment>   
    </environments>
</configuration>
```

### 3.2.2 log4j.properties

打印日志可用于调试和分析代码。

```properties
log4j.rootLogger=DEBUG,A1
log4j.logger.org.apache=DEBUG
log4j.appender.A1=org.apache.log4j.ConsoleAppender
log4j.appender.A1.layout=org.apache.log4j.PatternLayout
log4j.appender.A1.layout.ConversionPattern=%-d{yyyy-MM-dd HH:mm:ss,SSS} [%t] [%c]-[%p] %m%n
```

## 3.3 主要源码编写

### 3.3.1 创建domain对象（User.java）

```java
package com.test.mybatis.domain;
import java.text.SimpleDateFormat;
import java.util.Date;
/** 
 * @author 20190712713 
 * @date 2019/8/28 17:38 
 */
public class User {    
    private String id;   
    private String userName;    
    private String password;    
    private String name;    
    private Integer age;   
    private Integer sex;    
    private Date birthday;    
    private String created;    
    private String updated;   
    
    //省略了getter和setter方法
    
    @Override   
    public String toString() {        
        return "User{" +            "id='" + id + '\'' +                
            ", userName='" + userName + '\'' +                
            ", password='" + password + '\'' +                
            ", name='" + name + '\'' +               
            ", age=" + age +                
            ", sex=" + sex +                
            ", birthday='" + new SimpleDateFormat("yyyy-MM-dd").format(birthday) + '\'' +               
            ", created='" + created + '\'' +                
            ", updated='" + updated + '\'' +               
            '}';   
    } 
}
```

### 3.3.2 创建dao接口（UserDao.java）

```java
package com.test.mybatis.dao;
import com.test.mybatis.domain.User;
import java.util.List;
/** 
 * @author 20190712713 
 * @date 2019/8/28 17:38 
 */
public interface UserDao {    
   /**     
    * 根据id查询用户信息     
    *     
    * @param id 待查询的用户ID     
    * @return   查询出的用户对象     
    */  
    User queryUserById(String id);    
    /**     
    * 查询所有用户信息     
    * @return 用户列表     
    */    
    List<User> queryUserAll();    
    /**     
     * 新增用户     
     * @param user 用户对象     
     */    
    void insertUser(User user);    
    /**     
     * 更新用户信息       
     * @param user 待更新数据的用户对象     
     */    
    void updateUser(User user);    
    /**     
     * 根据id删除用户信息      
     * @param id 待删除用户的ID    
     */    
    void deleteUser(String id);
}
```

### 3.3.3 创建映射文件（UserDaoMapper.xml）

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- mapper:根标签，namespace：命名空间，一般保证命名空间唯一 -->
<mapper namespace="UserDao">    
      <select id="queryUserById" resultType="com.test.mybatis.domain.User">
          select tuser.id        as id,
          tuser.user_name as userName, 
          tuser.password  as password,
          tuser.name      as name, 
          tuser.age       as age,
          tuser.birthday  as birthday,
          tuser.sex       as sex,
          tuser.gmt_create   as created,
          tuser.gmt_update   as updated
          from tb_user tuser 
          where tuser.id = #{id};
    </select>    
    <select id="queryUserAll" resultType="com.test.mybatis.domain.User">
        select * 
        from tb_user;
    </select> 
    <!--插入数据-->
    <insert id="insertUser" parameterType="com.test.mybatis.domain.User"> 
        INSERT INTO tb_user (
        user_name,
        password,
        name,
        age,
        sex,
        birthday,
        gmt_create,
        gmt_update
        ) 
        VALUES (
        #{userName},
        #{password}, 
        #{name},
        #{age},
        #{sex}, 
        #{birthday},
        now(),
        now()
        ); 
    </insert>
    <update id="updateUser" parameterType="com.test.mybatis.domain.User">
        UPDATE tb_user
        <trim prefix="set" suffixOverrides=","> 
        <if test="userName!=null">user_name = #{userName},</if> 
        <if test="password!=null">password = #{password},</if> 
        <if test="name!=null">name = #{name},</if> 
        <if test="age!=null">age = #{age},</if> 
        <if test="sex!=null">sex = #{sex},</if> 
        <if test="birthday!=null">birthday = #{birthday},</if>
        gmt_update = now(),       
        </trim>
        WHERE 
        (id = #{id}); 
    </update>
    
    <delete id="deleteUser">
        delete
        from tb_user 
        where id = #{id}
    </delete></mapper>
```

### 3.3.4 创建daoImpl实现类(UserDaoImpl)

```java
package com.test.mybatis.dao.impl;
import com.test.mybatis.dao.UserDao;
import com.test.mybatis.domain.User;
import org.apache.ibatis.session.SqlSession;
import java.util.List;
/**
 * @author 20190712713
 * @date 2019/8/29 8:59
 */
public class UserDaoImpl implements UserDao {
    private SqlSession sqlSession;
    public UserDaoImpl(SqlSession sqlSession) {
        this.sqlSession = sqlSession;
    }
    public User queryUserById(String id) {
        return sqlSession.selectOne("UserDao.queryUserById",id);
    }
    public List<User> queryUserAll() {
        return sqlSession.selectList("UserDao.queryUserAll");
    }
    public void insertUser(User user) {
        sqlSession.insert("UserDao.insertUser",user);
    }
    public void updateUser(User user) { 
        sqlSession.update("UserDao.updateUser",user);
    }
    public void deleteUser(String id) {
        sqlSession.delete("UserDao.deleteUser",id);
    }
}
```

### 3.3.5 创建测试类（UserDaoTest.java）

``` java
package com.test.mybatis.dao;
import com.test.mybatis.dao.impl.UserDaoImpl;
import com.test.mybatis.domain.User;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Before;
import org.junit.Test;
import java.io.InputStream;
import java.util.Date;
import java.util.List;
public class UserDaoTest {
    private UserDao userDao = null;
    private SqlSession sqlSession = null;
    @Before 
    public void setUp() throws Exception {
        String resource = "mybatis-config.xml"; 
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream); 
        sqlSession = sqlSessionFactory.openSession();
        userDao = new UserDaoImpl(sqlSession);
    } 
    
    @Test
    public void queryUserById() {
        System.out.println(userDao.queryUserById("1"));
    }
    @Test
    public void queryUserAll() {
        List<User> userList = userDao.queryUserAll();
        for (User user : userList) {
            System.out.println(user); 
        }
    }
    @Test
    public void insertUser() {
        User user = new User();
        user.setAge(16); 
        user.setBirthday(new Date("1990/09/02"));
        user.setName("大鹏");
        user.setPassword("123456");
        user.setSex(1);
        user.setUserName("evan");
        this.userDao.insertUser(user);
        this.sqlSession.commit();
    } 
    @Test
    public void updateUser() {
        User user = new User();
        user.setBirthday(new Date());
        user.setName("静鹏"); 
        user.setPassword("654321"); 
        user.setSex(1); 
        user.setUserName("evanjin");
        user.setId("1"); 
        this.userDao.updateUser(user);
        this.sqlSession.commit();
    }
    @Test
    public void deleteUser() {
        this.userDao.deleteUser("4");
        this.sqlSession.commit();
    }
}
```

### 3.3.6 方式一缺陷分析

方式一，除接口和映射文件外，还要写实现类，且在实现类UserDaoImpl.java中，

（1）sqlSession的执行方式相似；

（2）映射文件中的sql语句以namespace.id的方式硬编码到实现类中；

思考：由于实现类中，sqlSession的使用方式类似，所以是否可以不写实现类，把这些类似的调用交给第三方（mybatis），让mybatis替我们完成。

# 4 Mybatis实现方式二：动态代理

## 4.1  实现步骤

（1）创建接口

（2）创建映射文件

​			**注意：映射文件中的namespace必须以接口的全路径名进行赋值。**

## 4.2 创建接口（UserMapper.java）

```java
package com.test.mybatis.dao.mapper;
import com.test.mybatis.domain.User;
import org.apache.ibatis.annotations.Param;
import java.util.List;
/** 
 * @author 20190712713
 * @date 2019/8/29 15:29
 */
public interface UserMapper {
    /**
     * 使用注解的方式注入参数 
     * @param tableName 待查询的表名 
     * @return 用户对象列表
     */ 
    List<User> queryUserByTableName(@Param("tableName") String tableName);
    /**
     * 根据Id查询用户信息
     * @param id 用户ID
     * @return 返回依据用户ID查询得到的用户对象
     */ 
    User queryUserById(Long id);
    /**
     * 查询所有用户信息
     * @return 用户对象列表
     */ 
    List<User> queryUserAll(); 
    /**
     * 新增用户信息
     * @param user 新增用户对象
     */
    void insertUser(User user); 
    /**
     * 更新用户信息
     * @param user 待更新的用户对象
     */
    void updateUser(User user); 
    /**
     * 根据id删除用户信息
     * @param id 待删除的用户id 
     */ 
    void deleteUserById(Long id);}
```

## 4.3 创建映射文件（UserMapper.xml）

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- namespace：命名空间必须是接口的全路径名-->
<mapper namespace="com.test.mybatis.dao.mapper.UserMapper"> 
    <select id="queryUserByTableName" resultType="com.test.mybatis.domain.User">                 select *
        from ${tableName}
    </select>
    <select id="queryUserById" resultType="com.test.mybatis.domain.User">
        select *
        from tb_user
        where id = #{id}
    </select>
    <select id="queryUserAll" resultType="com.test.mybatis.domain.User">
        select *
        from tb_user
    </select>
    <insert id="insertUser" useGeneratedKeys="true" keyColumn="id" keyProperty="id"            parameterType="com.test.mybatis.domain.User">
        INSERT INTO tb_user(
          user_name,
       	  password,
       	  name, 
      	  age, 
       	  sex,
       	  birthday,
      	  gmt_create,
       	  gmt_update
        )
        VALUES (
        #{userName},
        #{password},
        #{name},
        #{age},
        #{sex}, 
        #{birthday},
        NOW(),
        NOW());
    </insert> 
    <!--更新的statement  id：唯一标识，动态代理要求和方法名保持一致 parameterType：参数的类型，动态代理要求和方法的参数类型一致  -->   
    <update id="updateUser" parameterType="com.test.mybatis.domain.User">
        UPDATE tb_user
        <trim prefix="set" suffixOverrides=","> 
            <if test="userName!=null">user_name = #{userName},</if>
            <if test="password!=null">password = #{password},</if>
            <if test="name!=null">name = #{name},</if>
            <if test="age!=null">age = #{age},</if>
            <if test="sex!=null">sex = #{sex},</if>
            <if test="birthday!=null">birthday = #{birthday},</if>
            gmt_update = now(),
        </trim> 
        WHERE
        (id = #{id});
    </update> 
    <delete id="deleteUserById" parameterType="java.lang.Long">
        delete
        from tb_user
        where id = #{id}
    </delete>
</mapper>
```

## 4.4 创建测试类

```java
package com.test.mybatis.dao.mapper;
import com.test.mybatis.domain.User;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Before;
import org.junit.Test;
import java.io.InputStream;
import java.util.Date;
import java.util.List;
public class UserMapperTest {
    private UserMapper userMapper;
    @Before
    public void setUp() throws Exception {
        //创建配置文件流
        String resource = "mybatis-config.xml"; 
        InputStream inputStream = Resources.getResourceAsStream(resource);
        //创建SqlSessionFactory工厂
        SqlSessionFactory sqlSessionFactory = new 
            SqlSessionFactoryBuilder().build(inputStream);
        //打开SqlSession 
        SqlSession sqlSession = sqlSessionFactory.openSession();
        userMapper = sqlSession.getMapper(UserMapper.class);
    } 
    @Test
    public void testQueryUserByTableName() {
        List<User> userList = this.userMapper.queryUserByTableName("tb_user");
        for (User user : userList) {
            System.out.println(user);
        }
    } 
    @Test
    public void testQueryUserById() {
        System.out.println(this.userMapper.queryUserById(1l));
    }
    @Test
    public void testQueryUserAll() {
        List<User> userList = this.userMapper.queryUserAll();
        for (User user : userList) {
            System.out.println(user);
        }
    }
    @Test
    public void testInsertUser() {
        User user = new User();
        user.setAge(26);
        user.setBirthday(new Date());
        user.setName("欧阳");
        user.setPassword("123456");
        user.setSex(2);
        user.setUserName("Baron OUYang");
        this.userMapper.insertUser(user);
        System.out.println(user.getId());
    }
    @Test
    public void testUpdateUser() {
        User user = new User();
        user.setBirthday(new Date());
        user.setName("静静");
        user.setPassword("123456");
        user.setSex(0);
        user.setUserName("Jinjin");
        user.setId("8"); 
        this.userMapper.updateUser(user);
    }
    @Test
    public void testDeleteUserById() {
        this.userMapper.deleteUserById(6L);
    }
}
```

# 5 Spring整合mybatis

## 5.1 引入相关依赖（pom.xml）

```xml
<dependency>
    <groupId>c3p0</groupId>
    <artifactId>c3p0</artifactId>
    <version>0.9.1.2</version>
</dependency>
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>1.2.2</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>4.1.3.RELEASE</version>
</dependency>
<!--spring集成Junit测试-->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>4.1.3.RELEASE</version>
    <scope>test</scope>
</dependency>
<!--spring容器-->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>4.1.3.RELEASE</version>
</dependency>
```

## 5.2 创建Spring配置文件

### 5.2.1 jdbc.propertis

```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/mybatisdemo?useUnicode=true&characterEncoding=gbk
jdbc.username=root
jdbc.password=
```

### 5.2.2 applicationContext-dao.xml

```xml
<?xml version="1.0" encoding="UTF-8"?><beans xmlns="http://www.springframework.org/schema/beans"       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"       xmlns:context="http://www.springframework.org/schema/context"       xsi:schemaLocation="http://www.springframework.org/schema/beans       http://www.springframework.org/schema/beans/spring-beans.xsd       http://www.springframework.org/schema/context       http://www.springframework.org/schema/context/spring-context.xsd">
    <!-- 配置整合mybatis过程 -->
    <!-- 1.配置数据库相关参数properties的属性：${url} -->
   <context:property-placeholder location="classpath:jdbc.properties"/>
    <!-- 2.数据库连接池 --> 
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource"> 
        <!-- 配置连接池属性 --> 
        <property name="driverClass" value="${jdbc.driver}"/>
        <property name="jdbcUrl" value="${jdbc.url}"/>
        <property name="user" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/> 
        <!-- c3p0连接池的私有属性 --> 
        <property name="maxPoolSize" value="30"/>
        <property name="minPoolSize" value="10"/> 
        <!-- 关闭连接后不自动commit -->
        <property name="autoCommitOnClose" value="false"/>
        <!-- 获取连接超时时间 -->
        <property name="checkoutTimeout" value="10000"/>
        <!-- 当获取连接失败重试次数 -->
        <property name="acquireRetryAttempts" value="2"/> 
    </bean>
    <!-- 3.配置SqlSessionFactoryBean --> 
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">               <property name="dataSource" ref="dataSource"/> 
        <!-- 自动扫描mapping.xml文件 --> 
        <property name="mapperLocations" value="classpath:mapper/*.xml"/>
        <!--如果mybatis-config.xml没有特殊配置也可以不需要下面的配置-->
        <property name="configLocation" value="classpath:mybatis-config.xml"/> 
    </bean>
    <!-- 4.DAO接口所在包名，Spring会自动查找其下的类 -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.test.mybatis.dao.mapper"/> 
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
    </bean>
    <!-- (事务管理)transaction manager -->
    <bean id="transactionManager" 
          class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/> 
    </bean>
</beans>
```

## 5.3 创建测试类（UserMapperSpringTest.java）

**该测试类的域对象为User.java，接口为UserMapper.java**

```java
package com.test.mybatis.dao.spring;
import com.test.mybatis.dao.mapper.UserMapper;
import com.test.mybatis.domain.User;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import java.util.Date;
import java.util.List;
/**
 * @author 20190712713 
 * @date 2019/8/30 11:00 
 */
//junit整合spring的测试
@RunWith(SpringJUnit4ClassRunner.class)
//加载核心配置文件，自动构建spring容器
@ContextConfiguration("classpath:applicationContext.xml")
public class UserMapperSpringTest {
    @Autowired
    private UserMapper userMapper;
    @Test
    public void testQueryUserByTableName() {
        List<User> userList = this.userMapper.queryUserByTableName("tb_user"); 
        for (User user : userList) { 
            System.out.println(user);
        }
    }
    @Test
    public void testQueryUserById() { 
        System.out.println(this.userMapper.queryUserById(2L));
        User user = new User();
        user.setName("翠花"); 
        user.setId("1");
        userMapper.updateUser(user);
        System.out.println(this.userMapper.queryUserById(1L));
    } 
    @Test
    public void testQueryUserAll() {
        List<User> userList = this.userMapper.queryUserAll(); 
        for (User user : userList) {
            System.out.println(user);
        }
    }
    @Test
    public void testInsertUser() {
        User user = new User();
        user.setAge(20);
        user.setBirthday(new Date());
        user.setName("王二");
        user.setPassword("123456");
        user.setSex(2);
        user.setUserName("we");
        this.userMapper.insertUser(user);
        System.out.println(user.getId());
    }
    @Test
    public void testUpdateUser() {
        User user = new User();
        user.setBirthday(new Date());
        user.setName("张三"); 
        user.setPassword("123456"); 
        user.setSex(0);
        user.setUserName("ZS");
        user.setId("1");
        this.userMapper.updateUser(user);
    } 
    @Test
    public void testDeleteUserById() {
        this.userMapper.deleteUserById(1l);
    }
}
```

## 5.4 注意的问题

（1）测试类中没有sqlSession.commit()语句，所以数据库中没有插入数据的记录；

（2）由于在applicaitonContext-dao.xml文件中在3步已经配置了自动扫描映射文件，所以要将mybatis-config.xml文件中删除对应的mapper映射文件；

（3）在applicationContext-dao.xml中，若不配置第4步，可以通过SqSessionTemplate类中的getMapper(UserMapper.class)直接创建接口的实例对象，通过该实例对象调用相应的方法。



# 6 SpringMVC整合mybatis

# 出现的问题

## 1 org.apache.ibatis.exceptions.PersistenceException: 

​       Error opening session.  Cause: java.lang.NullPointerException

​       Cause: java.lang.NullPointerException

​      **原因：**mybatis-config.xml文件中environments 标签中default的值与environment中的id值不一致；