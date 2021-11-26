# Spring注解驱动教程

视频地址：https://www.bilibili.com/video/BV1gW411W7wy?p=2&spm_id_from=pageDriver



## 一、组件注册

> 进行实验前需先导入主要依赖

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.2.12.RELEASE</version>
</dependency>
```

### 1.配置方式创建bean

以前在没有使用注解去进行开发的时候，是需要配置一个beans.xml之类的文件去进行组件注册的。

往spring容器内放进Person的bean实例具体操作：

（1）创建Person实体类

（2）编写beans.xml配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
    //具体的bean，手动注入
    <bean class="com.wenwen.beans.Person" id="person">
        <property name="name" value="小明"/>
        <property name="age" value="11"/>
    </bean>
</beans>
```

（3）使用bean

```java
public class Run {
    public static void main(String[] args) {
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("beans.xml");
        Person bean = (Person)applicationContext.getBean("person");
        System.out.println(bean);
    }
}
```

### 2.注解方式创建bean

使用注解开发后就不需要写配置文件了，就只需要写一个配置类。

（1）配置类Config

```java
@Configuration//告诉spring这是一个配置类
public class Config {
    //如若不填写bean实例的id就是方法的名字
    @Bean("person")
    public Person person(){
        return new Person("小狗",11);
    }
}
```

（2）使用bean

```java
public class Run {
    public static void main(String[] args) {
         ApplicationContext applicationContext = new AnnotationConfigApplicationContext(Config.class);
         Person bean = (Person)applicationContext.getBean(Person.class);
        System.out.println(bean);
    }
}
```



### 3.组件扫描component-scan

组件扫描即是把指定包里的组件放到Spring容器中的方法

bean.xml配置文件下

```xml
<context:component-scan base-package="com.wenwen"/>
```



注解配置下，用法标注在配置类上面

```java
@ComponentScan(value = "com.wenwen",excludeFilters = {		//使用excludeFilters可以指定一类组件
        @ComponentScan.Filter(type = FilterType.ANNOTATION,classes = Controller.class)} )
public class Config {
```

excludeFilters :指定扫描的时候按照什么规则排除那些组件

includeFilters :指定扫描的时候只需要包含哪些组件



### 4.设置组件作用域scope

```java
@Scope
@Bean("person")
public Person person(){
```

@Scope可以设置四个作用域：prototype、singleton、request、session

**prototype 多实例的** ：IOC容器启动并不会去调用方法创建对象然后放在容器中，而是只有获取的时候才会调用方法创建对象。

**singleton 单实例的：**IOC容器启动就会调用方法创建对象放到IOC容器中，以后每次获取都是容器中的同一个实例。

**request：**一次请求创建一个实例

**session: **一个session创建一个实例



`@Lazy`懒加载，可以写在组件上

​			**单实例bean:**默认在容器启动的时候创建对象

​			**懒加载：**容器启动不创建对象。第一次使用Bean创建对象，并初始化

### 5.@Conditional条件注册

它的作用是按照一定的条件进行判断，满足条件给容器注册bean

https://blog.csdn.net/xcy1193068639/article/details/81491071

### 6.@Import快速导入组件

给容器中注册组件：

（1）包扫描+组件标注注解(@Component)

（2）@Bean【导入的第三方包里面的组件】

（3）@Import【快速给容器中导入一个组件】

​		a).@Import（要导入到容器中的组件）;容器中就会自动注册这个组件，id默认是全类名

​		b).ImportSelector:返回需要导入的组件的全类名数组；

​		c).ImportBeanDefinitionRegistrar:手动注册bean到容器中

（4）使用Spring提供的FactoryBean（工厂bean）

​		a).默认获取到的是工厂bean调用getObject创建的对象

​		b).要获取工厂Bean本身，我们需要给id前面加一个&

​				`applicationContext.getBean("&factoryBean");`

**使用FactoryBean例子**

```java
public class DogFactoryBean implements FactoryBean {
    @Override
    public Object getObject() throws Exception {
        System.out.println("factorybean getObject....");
        return new Dog();
    }

    @Override
    public Class<?> getObjectType() {
        return Dog.class;
    }
    // 返回true的话就是单实例bean,
    //返回false就是多实例bean
    @Override
    public boolean isSingleton() {
        return true;
    }
}
```

配置类Config.class

```java
@Bean
public DogFactoryBean factoryBean(){
    return new DogFactoryBean();
}
```

*在创建Spring容器时，这个工厂bean会加载到spring容器中，当你想把这个bean拿出来的时候，他实际是Dog bean，主要是因为执行了FactoryBean接口的getObject方法。*

## 二.Bean生命周期

> 生命周期：从对象创建到对象销毁的过程

1. 通过@Bean 指定init-method和destroy-method

```java
@Bean(initMethod = "你的方法1",destroyMethod = "你的方法2")
```

​	2.使用类实现InitializingBean（定义初始化逻辑）接口初始化逻辑,实现DisposableBean（定义销毁逻辑）接口销毁方法

​	3.实现BeanPostProcessor接口的后置拦截器放入容器中，可以拦截bean初始化，并可以在被拦截的Bean的初始化前后进行一些处理工作。

## 三.属性赋值@Value

使用@Value赋值

（1）基本数值

（2）可以写SpEL;#{}

（3）可以写${};取出配置文件【properties】中的值

```java
@Value("asd")
String name;

@Value(${person.nickName})
String nickName
--------------------
配置类
//使用@PropertySource读取外部配置文件中的k/v保存在运行的环境变量中
@PropertySource(value = {"classpath:/person.properties"})
```



## 四.自动装配

Spring利用依赖注入（DI），完成对IOC容器中各个组件的依赖关系赋值

1.`@Autowired`：自动注入

​	A、默认优先按照类型去容器中找对应的组件：applicationContext.getBean(BookDao.class)

​	B、如果找到多个相同类型的组件，再将属性的名称作为组件的id去容器中查找：app.getBean("bookDao")

​	C、@Qualifier("bookDao"):使用@Qualifier指定需要装配的组件的id,而不是属性名

​	D、@Autowired(require=false)如果没有找到指定的bean也不会报错

2.`@Autowired`还能放在构造器、方法、参数，属性上；都是从容器中获取参数组件的值

3.自定义组件想要使用Spring容器底层的一些组件比如（ApplicationContext,BeanFactory,...）

​		自定义组件实现xxxAware;在创建对象的时候，会调用接口规定的方法注入相关组件

​		Aware把Spring底层一些组件注入到自定义的Bean中

​		xxxAware使用xxxProcessor;



2.`@Profile`

​	Spring为我们提供了可以根据当前环境，动态的激活和切换一系列组件的功能

比如数据源的切换，开发环境（A数据源）测试环境（B数据源）生存环境（C数据源）

​	（1）加了环境表示的bean，只有这个环境被激活的时候才能注册到容器中。默认是default环境

​	（2）写在配置类上，只有是指定的环境的时候，整个配置类里面的所有配置才能开始生效

​	（3）没有标注环境表示的bean在任何环境下都是加载的

## 五.AOP

1. @EnableAspectJAutoProxy 开启基于注解的aop模式（在配置类中加上）
2. @Aspect：定义切面类，切面类里定义通知
3. @PointCut 切入点，可以写切入点表达式，指定在哪个方法切入
4. 通知方法
   - @Before(前置通知)  在目标方法运行之前运行
   - @After(后置通知)  在目标方法运行之后运行
   - @AfterReturning(返回通知)  在目标方法正常返回之后运行
   - @AfterTrowing(异常通知)   在目标方法出现异常之后运行
   - @Around(环绕通知)
5. JoinPoint：连接点,是一个类，配合通知使用，用于获取切入的点的信息

## 六.声明式事务

1.需要开启基于注解的事务管理功能，在配置类上标注@EnableTransactionManagement

2.给方法上标注@Transactional表示当前方法是一个事务方法

3.配置事务管理器来控制事务，把他加进容器中

`@Bean`

`public PlatformTransactionManager transactionManager()`