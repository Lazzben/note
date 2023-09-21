# Spring ioc

[TOC]

## 使用new来创建JavaBean会有什么问题？

高层模块依赖底层模块，违反了**依赖倒置原则DIP（Dependency Inversion Principle）**。

<img src="/Users/lazyben/Library/Application Support/typora-user-images/image-20230920213011671.png" alt="image-20230920213011671" style="zoom:50%;" />

DIP原则是指高层模块不应该依赖于底层模块，它们都应该依赖于抽象。面向接口编程是 DIP 的一种实现方式。

<img src="/Users/lazyben/Library/Application Support/typora-user-images/image-20230920213140849.png" alt="image-20230920213140849" style="zoom:50%;" />

如何实现依赖倒置原则呢？不管我们的类型是接口还是具体的类，我们都需要手动去new一个具体的类，也就是依赖了某个具体的类，所以我们需要把对对象的控制权交出去。为了满足 DIP 原则，我们把创建对象的控制权交给了第三方。Java 程序员完成类的定义，第三方获取类定义，通过反射创建对象并完成依赖注入（DI），这种思想叫做 IoC。



## DIP、IoC、DI、Spring 的关系

<img src="/Users/lazyben/Library/Application Support/typora-user-images/image-20230920215420672.png" alt="image-20230920215420672" style="zoom:50%;" />

- DIP，Dependency Inversion Principle，依赖倒置原则。指高层模块不应该依赖于底层模块，它们都应该依赖于抽象。

- IoC， Inverse of Control，控制反转。指控制权的转移，将对象的创建和管理交给容器来完成，而不是由程序员。IoC 是实现 DIP 的一种手段。

- DI， Dependency Injection，依赖注入。指将一个对象所依赖的其他对象通过构造函数、属性或方法参数的方式传递给它，通过注入的方式实现依赖关系的管理，DI 是实现 IoC 的一种手段。

- Spring 是一个基于 IoC 的框架，Spring 帮 Java 程序员完成了对象的创建和管理。IoC 是 Spring框架最底层的核心。

## 手写一个简单的IOC容器

[代码地址]()

解决单例和循环依赖。

## Spring IOC的使用：xml配置文件方式

[代码地址]()

### 引入依赖

注意不要选择6.x的版本，Spring 6 不支持JDK 8。

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
</dependency>
```

### 定义类

使用lombok的@Data注解，自动生成getter和setter方法。

```java
// 定义User类
@Data
public class User {
    private Long id;

    private String name;

    private Account account;
}
```

```java
// 定义Account类
@Data
public class Account {
    private BigDecimal balance;
}
```

### 配置xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="user" class="com.lazyben.entity.User">
        <property name="name" value="Ben"/>
        <property name="id" value="1"/>
        <property name="account" ref="account"/>
    </bean>
    <bean id="account" class="com.lazyben.entity.Account">
        <property name="balance" value="100"/>
    </bean>
</beans>
```

### 从Spring IOC容器中获取Bean

其中bean.xml是我们配置文件的名字，user是配置文件中声明的User类型的bean id。

```java
@Test
public void getUserTest(){
    ClassPathXmlApplicationContext classPathXmlApplicationContext = new ClassPathXmlApplicationContext("bean.xml");
    Object user = classPathXmlApplicationContext.getBean("user");
    System.out.println(user);
}
```

#### set注入

按以上的写法，事实上使用的是set注入，本质上利用反射调用类的无参构造函数，再调用 set 方法完成依赖注入。

- 属性为简单类型，使用 value

- 属性为 bean，使用 ref

- set 方法的名字很重要。set 方法名去掉 set 后，剩余字符串首字母小写 = bean property 的 name

为了验证，是用set方法注入的，我们去掉lombok的@Data注解。报了以下的错误。

> Caused by: org.springframework.beans.NotWritablePropertyException: Invalid property 'balance' of bean class [com.lazyben.entity.Account]: Bean property 'balance' is not writable or has an invalid setter method. Does the parameter type of the setter match the return type of the getter?

#### 构造方法注入

构造方法需要我们自定义构造器，修改User类，加入全参构造器。由于set注入利用反射调用无参构造器，再调用set方法注入。但是一旦加入全参构造器后，编译器自动加的无参构造器就会消失，为了能让set注入方式也能适应，所以我们需要额外把无参构造器也加上。

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private Long id;

    private String name;

    private Account account;
}
```

修改xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!-- 使用构造方法参数的名字进行参数绑定 -->
    <bean id="user" class="com.lazyben.entity.User">
        <constructor-arg name="name" value="Ben"/>
        <constructor-arg name="id" value="1"/>
        <constructor-arg name="account" ref="account"/>
    </bean>
		<!-- 根据构造方法参数的顺序进行参数绑定 -->
<!--    <bean id="user" class="com.lazyben.entity.User">-->
<!--        <constructor-arg index="0" value="1"/>-->
<!--        <constructor-arg index="1" value="bBen"/>-->
<!--        <constructor-arg index="2" ref="account"/>-->
<!--    </bean>-->
    
    <bean id="account" class="com.lazyben.entity.Account">
        <property name="balance" value="100"/>
    </bean>
</beans>
```

为了验证此时使用构造器方式进行注入的，我们可以在构造器内部打上断点进行debug。

#### 自动装配

Bean类型的属性可以进行自动装配

- byType

  只需要加上autowire="byType"，就能根据类型自动装配，如果该类型只有一个Bean则自动完成装配。

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://www.springframework.org/schema/beans
             http://www.springframework.org/schema/beans/spring-beans.xsd">
      <bean id="user" class="com.lazyben.entity.User" autowire="byType">
          <property name="name" value="Ben"/>
          <property name="id" value="1"/>
      </bean>
      <bean id="account" class="com.lazyben.entity.Account">
          <property name="balance" value="100"/>
      </bean>
  </beans>
  ```

  如果该类型有两个或以上的Bean，则报错。

  > org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'user' defined in class path resource [bean.xml]: Unsatisfied dependency expressed through bean property 'account'; nested exception is org.springframework.beans.factory.NoUniqueBeanDefinitionException: No qualifying bean of type 'com.lazyben.entity.Account' available: expected single matching bean but found 2: account,account2

- byName

  autowire的值还可以是buName（这里名称是set方法去掉set 后首字母小写），将通过属性名称找到唯一的id进行装配

**自动装配是通过setter方法进的，所以不能去掉setter方法。**

## Spring IOC的使用：注解方式

[代码地址]()

#### Spring IOC注解概述

xml 配置文件方式实现 IoC，本质上是在 xml 中定义 bean，告诉 Spring 要创建哪些 bean，以及 bean 的属性如何注入。而通过注解的方式，则是用注解标记 bean，用注解标记 bean 的属性如何注入。Spring会扫描代码里的注解，创建 bean，完成依赖注入。

#### 使用注解进行标注

```java
@Data
@Component
public class User {
    @Value("2")
    private Long id;

    @Value("mary")
    private String name;

    @Resource
    private Account account;
}
```

```java
@Data
@Component
public class Account {
    @Value("5000")
    private BigDecimal balance;
}
```

#### 配置Spring扫描路径

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="com.lazyben"/>

</beans>
```

#### 标记Bean的注解@Component、@Service、@Controller、@Repository

- @Component注解

  Spring 扫描到装饰有 @Component 注解的类，会给这个类创建 bean 并放入 Spring IoC 容器。@Component 注解的 value 是 bean 的 id，如果没有配置 value，那么取类名首字母小写为 bean id。

- @Service、@Controller、@Repository

  这三个注解上都有 @Component，这三个注解的作用跟 @Component 是一样的。

  **区别在于这三个注解用于装饰不同层的类。 @Controller 装饰 Controller 层， @Service 用于装饰 Service层， @Repository 用于装饰 Dao 层。**

  如果只看实际效果的话， @Component 、@Service、@Controller、@Repository 是一样的。

#### 实现注入的注解@Value、@AutoWired、@Resource

- @Value 

  用于注入简单类型的属性。可以作用在字段、方法（set方法）、参数（构造方法的参数）上。

- @Autowired 

  用于注入 bean 类型的属性。可以作用在字段、set方法、构造方法的参数、构造方法上。

  - @Autowired 优先根据类型装配。

  - 先按照属性的类型寻找 bean，如果该类型只找到一个bean，直接装配

  - 如果该类型找到多个bean，再在这些类型匹配的 bean 之中找与属性名称匹配的，找到则装配

  - 在多个类型匹配的 bean 中找不到名称匹配的，装配失败，抛异常。

  - @Autowired 可配合 @Qualifier 使用， @Qualifier 用于指定使用装配 bean 的名称

- @Resource

  @Resource 优先根据名称装配。名称是 @Resource 的 name，如果不配置 name，以属性名为名称。

  - 先按名称寻找 bean，找到则检查类型是否匹配，类型匹配注入，类型不匹配抛异常

  - 按照名称找不到 bean，按属性的类型寻找，若按类型只找到一个 bean 则装配，按类型找到多个则抛异常。

#### IOC全注解编程

- 使用java类代替xml配置

  ```java
  @Configuration
  @ComponentScan(basePackages = "com.lazyben")
  public class SpringConfiguration {
  }
  ```

- 使用 AnnotationConfigApplicationContext 代替 ClassPathXmlApplicationContext

  ```java
  @Test
  public void getUserTest(){
      ApplicationContext applicationContext = new AnnotationConfigApplicationContext(SpringConfiguration.class);
      Object user = applicationContext.getBean("user");
      System.out.println(user);
  }
  ```