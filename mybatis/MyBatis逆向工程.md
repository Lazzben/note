# MyBatis逆向工程

[代码地址](https://github.com/Lazyben-pg/mybatis-learn/tree/main/mybatis-generator)

[TOC]

## MyBatis逆向工程概述

根据数据库表的信息，反向创建如下资源：

- Java POJO类
- Mapper接口
- Mapper xml文件

**自动生成大量代码，极大提升研发效率**



## 使用MyBatis Generator插件自动生成代码文件

### 所需依赖

需要在pom文件中添加mybatis genreator插件，mybatis genreator插件还需要将overwrite配置为true，以便每次生成都可以覆盖之前的代码，值得注意的是，**每次生成前我们需要删除掉Mapper.xml文件，这类文件不会自动覆盖，而是继续追加。**

```xml
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>

<build>
    <plugins>
        <plugin>
            <groupId>org.mybatis.generator</groupId>
            <artifactId>mybatis-generator-maven-plugin</artifactId>
            <configuration>
                <overwrite>true</overwrite>
            </configuration>
            <dependencies>
                <dependency>
                    <groupId>mysql</groupId>
                    <artifactId>mysql-connector-java</artifactId>
                    <version>8.0.33</version>
                </dependency>
            </dependencies>
        </plugin>
    </plugins>
</build>
```

### 配置generatorConfig.xml文件

该文件名不能修改，如果修改则导致插件无法成功找到配置。

配置完该文件后，就可以使用mvn命令生成代码了。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>

    <!--加载jdbc.properties中的属性-->
    <properties resource="jdbc.properties"/>

    <!-- context id 可以自由定义 targetRuntime 选择 Mybatis3 -->
    <context id="lazyben" targetRuntime="Mybatis3">
        <!--自动生成toString方法-->
        <plugin type="org.mybatis.generator.plugins.ToStringPlugin"/>
        <!-- 注释信息配置 -->
        <commentGenerator>
            <!-- 是否去除自动生成的注释，默认为 false -->
            <property name="suppressAllComments" value="true"/>
            <!-- 是否阻止生成的注释包含时间戳，默认为 false -->
            <property name="suppressDate" value="true"/>
        </commentGenerator>

        <!--连接数据库信息-->
        <jdbcConnection driverClass="${jdbc.driver}"
                        connectionURL="${jdbc.url}"
                        userId="${jdbc.user}"
                        password="${jdbc.password}"/>

        <!--自动生成的 POJO 类存放包路径 -->
        <javaModelGenerator targetPackage="com.lazyben.pojo" targetProject="src/main/java"/>

        <!--自动生成的*Mapper.xml文件存放路径 -->
        <sqlMapGenerator targetPackage="com.lazyben.mapper" targetProject="src/main/resources"/>

        <!--自动生成的*Mapper.java存放路径 -->
        <javaClientGenerator type="XMLMAPPER" targetPackage="com.lazyben.mapper" targetProject="src/main/java"/>

        <!-- 逆向解析的表 domainObjectName 是生成的 POJO 类类名 -->
        <table tableName="account" domainObjectName="Account"/>

    </context>
</generatorConfiguration>
```



## QBC风格的条件查询

Query By Criteria，是一种面向对象的查询方式，这种查询方式以函数API的方式动态地设置查询条件，组成查询语句。MyBatis generator 生成的examle就属于一种QBC条件查询。通过调用方法凭借查询：

方法名为 andXxxYyy ，Xxx 为字段名，Yyy 为判断条件，例如 andIdEqualTo(value)，表示拼接 字段 id 等于 value 的查询条件。

```java
@Test
public void selectUserBySomeCondition() throws IOException {
    String resource = "mybatis-config.xml";
    InputStream inputStream = Resources.getResourceAsStream(resource);
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
    try (SqlSession sqlSession = sqlSessionFactory.openSession()) {
        AccountMapper mapper = sqlSession.getMapper(AccountMapper.class);
        AccountExample example = new AccountExample();
        AccountExample.Criteria criteria = example.createCriteria();
        criteria.andBalanceBetween(new BigDecimal(10), new BigDecimal(200));
        List<Account> accountList = mapper.selectByExample(example);
        System.out.println(accountList);
    }
}
```

