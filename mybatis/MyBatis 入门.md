# MyBatis入门

[TOC]

## 第一个MyBatis程序：使用非Mapper接口方式执行CURD

### 所需依赖

因为mybatis能代替繁琐的JDBC操作，底层还是基于JDBC实现的，所以需要driver去连接数据库，那么所需的依赖包括：mybatis和mysql-connector-java

```xml
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>xxx</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</atifactId>
    <version>xxx</version>
</dependency>
```

### 创建核心配置文件

核心配置文件主要定义数据库连接所必需的属性，包括driver、username、password、url等等。

除此之外定义了所含有的mappers。

配置文件的名称并不做要求。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <properties resource="jdbc.properties"/>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.user}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="AccountMapper.xml"/>
    </mappers>
</configuration>
```

### 定义POJO类

```java
@Data
public class Account {
    String name;
    Long id;
    BigDecimal balance;
}
```

### 创建并编辑SQL映射文件xxxMapper.xml

namespace命名不做要求，其中的id命名也不做要求，namespace.id的作用是定位这条sql语句，所以必须唯一。

resultType是所返回的类型，必须采用全限定类名，后续也可用别名代替。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="AccountMapper">
    <select id="selectUserById" resultType="com.lazyben.pojo.Account">
        select * from account where id = #{id}
    </select>
</mapper>
```

### 编写测试

由于是基于xml配置文件的mybatis，所以构建SqlSessionFactory时需要读取该xml文件，这是我们自己控制的，所以xml文件的命名不做要求。

selectOne根据namespace.id确定唯一的sql语句执行。

```java
public class MyTest {
    @Test
    public void selectUserByIdTest() throws IOException {
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        try (SqlSession sqlSession = sqlSessionFactory.openSession()) {
            Account account = sqlSession.selectOne("AccountMapper.selectUserById", 2);
            System.out.println(account);
        }
    }
}
```





## 使用Mapper接口方式执行SQL

### 使用非Mapper接口方式执行SQL的弊端

