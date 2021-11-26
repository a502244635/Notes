

# SpringBoot

## 一、SpringBoot简介

### 1、SpringBoot的优点

​	1.创建独立Spring应用

​	2.内嵌web服务器

​	3.自动starter依赖，简化构建配置

​	4.自动配置Spring以及第三方功能

​	5.提供生产级别的监控、健康检查及外部化配置

​	6.无代码生成、无需编写xml

### 2、快速搭建SpringBoot项目

**（1）在项目的pom.xml文件里继承父项目spring-boot-starter-parent**

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.3.7.RELEASE</version>
</parent>
```

> 在导入父项目的时候，idea报错了，他说找不到2.3.4.RELEASE的这个父项目，然后我就去https://mvnrepository.com/里查找spring-boot-starter-parent对应版本依赖，然后写在dependencies里面就好了。这个bug是因为使用阿里云镜像的问题。

**（2）web启动器，说明这个是一个web项目**

> If you run `mvn dependency:tree` again, you see that there are now a number of additional dependencies, including the Tomcat web server and Spring Boot itself.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

**（3）创建启动类MainApplication.java**

```java
package com.wenwen.controller;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class MainApplication {
    public static void main(String[] args){
        SpringApplication.run(MainApplication.class,args);
    }
}
```

> 使用的时候直接运行main方法即可

**（4）测试，创建一个Controller**

```java
package com.wenwen.controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
//返回字符串给页面相当于以前的@Controller和@Requestbody两个注解一起用
public class HelloController {
    @RequestMapping("/hello")
    public String hello(){
        return "hello,springboot";
    }
}
```

### 3、简化配置

**（1）SpringBoot的很多配置可以在application.properties里面进行配置**

**（2）简化部署**

```xml
<build>
 <plugins>
     <plugin>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
  </plugins>
 </build>
```

> 把项目打成jar包，直接在目标服务器执行即可。注意点：取消掉cmd的快速编辑模式



## 二、了解自动配置原理

### 1、父项目做依赖管理

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.3.7.RELEASE</version>
</parent>
```

> 在pom.xml文件里，springboot项目一般会继承`spring-boot-starter-parent`，这个父项目可以帮助我们进行依赖的版本管理

```xml
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-dependencies</artifactId>
  <version>2.3.7.RELEASE</version>
</parent>
```

> 点进`spring-boot-starter-parent`，我们发现他还有一个父项目`spring-boot-dependencies`。这个父项目里面有许许多多的具体依赖版本型号，到时候自动导入依赖的时候就会用`spring-boot-dependencies`里的依赖版本属性。
>
> 这里面是springboot帮项目规划好一些依赖版本号， 几乎声明了所有开发中常用的依赖的版本号,自动版本仲裁机制无需关注版本号，自动版本仲裁
>
> 1、引入依赖默认都可以不写版本
>
> 2、引入非版本仲裁的jar，要写版本号

修改依赖的版本号：

```xml
1、查看spring-boot-dependencies里面规定当前依赖的版本
2、在当前项目pom.xml里面重写配置
 <properties>
     <mysql.version>5.1.43</mysql.version>
 </properties>
```

### 2、自动配置

**（1）先来看看其中的一个依赖**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

> 这个`spring-boot-starter-web`依赖里面引入了很多关于web的依赖，比如：spring-boot-starter-tomcat，spring-boot-starter-json，spring-web，spring-webmvc，spring-boot-starter详细依赖请点进这个依赖里面进行查看

​	在`spring-boot-starter-web`依赖里的`spring-boot-starter`依赖有一个`spring-boot-autoconfigure`。这个`spring-boot-autoconfigure`引入的包里含有所有的自动配置功能，但是这些配置不都是启动的，只有开发者引入某个开发场景才会启动某个开发场景的自动配置功能，也就是按需加载所有自动配置项。

**（2）默认的包结构**

​	主程序（`MainApplication`）所在包及其下面的所有子包里面的组件都会被默认扫描进来，无需以前的包扫描配置   想要改变扫描路径可以采用以下注解进行配置：`@SpringBootApplication(scanBasePackages="com.wenwen")`   

或者`@ComponentScan` 指定扫描路径

```java
@SpringBootApplication
等同于
 @SpringBootConfiguration
 @EnableAutoConfiguration
 @ComponentScan("com.wenwen.cs")
```

## 三、容器功能

#### ###`@Configuration`

1、配置类里面使用@Bean标注在方法上给容器注册组件，默认也是单实例的
2、配置类本身也是组件
3、`proxyBeanMethods()`：代理bean的方法（在@Configuration注解里有这样的一个方法）
	默认为true
  Full(proxyBeanMethods = true)【保证每个@Bean方法被调用多少次返回的组件都是单实例的】
  Lite(proxyBeanMethods = false)【每个@Bean方法被调用多少次返回的组件都是新创建的】
组件依赖必须使用Full模式。其他默认使用Lite模式

> ​     配置 类组件之间无依赖关系用Lite模式加速容器启动过程，减少判断配置类组件之间有依赖关系，方法会被调用得到之前单实例组件，用Full模式。



#### ###`@Import`

@Import({User.class})

​	给容器中自动创建出这个类型的组件、默认组件的名字就是全类名



#### ###`@Conditional`

​	条件装配，如果满足某个条件就进行组件注入，底层里有大量的这种注解

```java
@ConditionalOnBean(name = "tom")
如果容器中有名字等于“tom”的组件，那么注解下面的配置类就会生效
 @ConditionalOnMissingBean(name = "tom")
如果容器中没有名字叫做"tom"的组件,那么注解下面的配置类就会生效
```



#### ###`@ImportResource`

> ​	原生配置文件引入

​	 使用场景：当有一些老的第三方工具里面有用xml写的把bean注入到容器中，比如文件是`beans.xml`，然后你想不想一个个在配置类里面进行`@Bean`处理（也就是注册到容器中），你就可以使用`@ImportResource（“classpath：beans.xml”）`注解。



#### ###`@ConfigurationProperties`

**数据绑定**

​	我们喜欢把一些容易改变的数据放在一个properties配置文件里，用的时候从配置文件里再拿出来

如何使用Java读取到properties文件中的内容，并且把它封装到JavaBean中，以供随时使用；

> 第一种用法@ConfigurationProperties+@Component

```java
只有在容器中的组件， 才会拥有SpringBoot提供的强大功能
@Component
@ConfigurationProperties(prefix = "mycar")//application.properties里面前缀为“mycar”的属性
public class Car {
    private String brand;
    private Integer price;
    
#application.properties
    mycar.brand=Benz
    mycar.price=1999
```

> 第二种用法@ConfigurationProperties+@EnableConfigurationProperties

```java
@EnableConfigurationProperties(Car.class)//往容器里注入Car组件
@Configuration
public class MyConfig {
    
}

@ConfigurationProperties(prefix = "mycar")//application.properties里面前缀为“mycar”的属性
public class Car {
    private String brand;
    private Integer price;
```

## 四、自动配置原理入门

### 1、查看启动配置类

```java
@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

}
```

> 启动类里有一个main方法，主要作用是启动整个SpringBoot应用

### 2、我们看到配置类上的注解：`@SpringBootApplication`

点进去看看

```java
@SpringBootConfiguration //代表当前是一个SpringBoot配置类
@EnableAutoConfiguration	
@ComponentScan(		//指定对哪些包进行组件扫描
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
```

### 3、###`@EnableAutoConfiguration`	

> 主要是由下面两个注解组合而成

```java
@AutoConfigurationPackage //自动配置包
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
```

#### 3.1、###`@AutoConfigurationPackage`

> 点进去发现一个导入注解

```java
@Import({Registrar.class})
//利用Registrar给容器中导入一系列组件
//将一个包下的所有组件导入进来，这里导入的是主程序DemoApplication所在包下
```

> 查看Registrar源代码

```java
static class Registrar implements ImportBeanDefinitionRegistrar, DeterminableImports {
    Registrar() {
    }

    public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
        AutoConfigurationPackages.register(registry,
        (String[])(new AutoConfigurationPackages.
                   PackageImports(metadata)).getPackageNames().toArray(new String[0]));
       //metadata就是传入的元数据，也就是获取启动类那里的信息
       //PackageImports(metadata)).getPackageNames().toArray(new String[0]));获得元数据所在的包名字，作为一个数组注册进去
    }
```



#### 3.2、###`@Import({AutoConfigurationImportSelector.class})`

查看核心源代码

```java
 AutoConfigurationImportSelector.AutoConfigurationEntry autoConfigurationEntry = 	this.getAutoConfigurationEntry(annotationMetadata);
```

```java
1、利用getAutoConfigurationEntry(annotationMetadata);
//给容器中批量导入一些组件
2、调用List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes)
    //获取到所有需要加载到容器中的配置类，加载了也不一定会生效，配置了才会生效
3、上面getCandidateConfigurations（）方法里有利用工厂加载 Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader)；
    //通过扫描指定位置得到所有的组件，
4、从META-INF/spring.factories位置来加载一个文件。
    默认扫描我们当前系统里面所有META-INF/spring.factories位置的文件，有些包没有这个文件
    spring-boot-autoconfigure-2.3.4.RELEASE.jar包里面也有META-INF/spring.factories，
    文件里面写死了spring-boot一启动就要给容器中加载的所有配置类
    
```

![image-20210611174647202](C:\Users\fat man\Desktop\笔记\SpringBoot\imgs\image-20210611174647202.png)

> 第二点的配置数量

### 4、按需开启自动配置项

​	虽然我们127个场景的所有自动配置启动的时候默认全部加载。xxxxAutoConfiguration
按照条件装配规则（`@Conditional`），最终会按需配置。



#### 4.1、###文件上传解析器

```java
        @Bean
        @ConditionalOnBean(MultipartResolver.class)  //容器中有这个类型组件
        @ConditionalOnMissingBean(name = DispatcherServlet.MULTIPART_RESOLVER_BEAN_NAME) 
		//容器中没有这个名字 multipartResolver 的组件
        public MultipartResolver multipartResolver(MultipartResolver resolver) {
            //给@Bean标注的方法传入了对象参数，这个参数的值就会从容器中找。
            //SpringMVC multipartResolver。防止有些用户配置的文件上传解析器不符合规范
            // Detect if the user has created a MultipartResolver but named it incorrectly
            return resolver;
        }
给容器中加入了文件上传解析器；
```

> SpringBoot默认会在底层配好所有的组件。但是如果用户自己配置了以用户的优先

### 5、总结

- SpringBoot先加载所有的自动配置类  xxxxxAutoConfiguration
- 每个自动配置类按照条件进行生效，默认都会绑定配置文件指定的值。xxxxProperties里面拿。xxxProperties和配置文件进行了绑定
- 生效的配置类就会给容器中装配很多组件
- 只要容器中有这些组件，相当于这些功能就有了
- 定制化配置

- - 用户直接自己@Bean替换底层的组件
  - 用户去看这个组件是获取的配置文件什么值就去修改。

**xxxxxAutoConfiguration ---> 组件  --->** **xxxxProperties里面拿值  ----> application.properties**（用户定义的配置文件）

### 6、dev-tools

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <optional>true</optional>
        </dependency>
```

> 项目或者页面修改以后：Ctrl+F9；

## 五、配置文件

https://www.yuque.com/atguigu/springboot/rg2p8g

> yaml vlue单引号不会转义，双引号会转义

**配置 提示**：

​	自定义的类和配置文件绑定一般没有提示，加入这个依赖就有提示了。

```xml
       <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
```

打包的时候需要把这个类去除，这样可以减少一些jvm的负担，下面是在pom.xml的打包插件除设置。

```xml
<build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.springframework.boot</groupId>
                            <artifactId>spring-boot-configuration-processor</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

## 六、web开发

### 1、静态资源访问

**（1）默认的静态资源目录**

静态资源放在类路径下：  `/static` (or `/public` or `/resources` or `/META-INF/resources`

使用当前项目根路径/ + 静态资源名去对静态资源进行访问



原理： 静态映射/**。

请求进来，先去找Controller看能不能处理。不能处理的所有请求又都交给静态资源处理器。静态资源也找不到则响应404页面

> 改变默认的静态资源路径

```yaml
spring:
  mvc://静态资源访问前缀，默认无前缀
    static-path-pattern: /res/**
    //访问静态资源的时候当前项目 + static-path-pattern + 静态资源名 = 静态资源文件夹下找
    //例子localhost:8080/res/jj.jpg

  resources:
    static-locations: [classpath:/haha/]
   	//以haha作为静态资源文件夹
```



**（2）欢迎页的配置**

往静态资源路径放置index.html页面

> 可以配置静态资源路径，但是不可以配置静态资源的访问前缀。否则导致 index.html不能被默认访问



**（3）自定义Favicon**

​	favicon.ico 图片放在静态资源目录下即可。



### 2、静态资源配置原理

- SpringBoot启动默认加载  xxxAutoConfiguration 类（自动配置类）
- SpringMVC功能的自动配置类 WebMvcAutoConfiguration进行生效

```java
@Configuration(
    proxyBeanMethods = false
)
@ConditionalOnWebApplication(
    type = Type.SERVLET
)
@ConditionalOnClass({Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class})
@ConditionalOnMissingBean({WebMvcConfigurationSupport.class})
@AutoConfigureOrder(-2147483638)
@AutoConfigureAfter({DispatcherServletAutoConfiguration.class, TaskExecutionAutoConfiguration.class, ValidationAutoConfiguration.class})
public class WebMvcAutoConfiguration {
```

**这个配置类里面配置了些什么，其中代码如下：**

```java
@Configuration(
    proxyBeanMethods = false
)
@Import({WebMvcAutoConfiguration.EnableWebMvcConfiguration.class})
@EnableConfigurationProperties({WebMvcProperties.class, ResourceProperties.class})//属性绑定
@Order(0)
public static class WebMvcAutoConfigurationAdapter implements WebMvcConfigurer {
```

> 配置文件的相关属性和xxx进行了绑定。WebMvcProperties==**spring.mvc**、ResourceProperties==**spring.resources**



**WebMvcAutoConfigurationAdapter有一个有参构造器**

```java
        public WebMvcAutoConfigurationAdapter(ResourceProperties resourceProperties,
                                              WebMvcProperties mvcProperties,
                                              ListableBeanFactory beanFactory, 		ObjectProvider<HttpMessageConverters> messageConvertersProvider, ObjectProvider<WebMvcAutoConfiguration.ResourceHandlerRegistrationCustomizer> resourceHandlerRegistrationCustomizerProvider, 
                                              ObjectProvider<DispatcherServletPath> dispatcherServletPath,
                                            ObjectProvider<ServletRegistrationBean<?>> servletRegistrations) {
            this.resourceProperties = resourceProperties;
            this.mvcProperties = mvcProperties;
            this.beanFactory = beanFactory;
            this.messageConvertersProvider = messageConvertersProvider;
            this.resourceHandlerRegistrationCustomizer = (WebMvcAutoConfiguration.ResourceHandlerRegistrationCustomizer)resourceHandlerRegistrationCustomizerProvider.getIfAvailable();
            this.dispatcherServletPath = dispatcherServletPath;
            this.servletRegistrations = servletRegistrations;
        }
    //有参构造器所有参数的值都会从容器中确定
//ResourceProperties resourceProperties；获取和spring.resources绑定的所有的值的对象
//WebMvcProperties mvcProperties 获取和spring.mvc绑定的所有的值的对象
//ListableBeanFactory beanFactory Spring的beanFactory
//HttpMessageConverters 找到所有的HttpMessageConverters
//ResourceHandlerRegistrationCustomizer 找到 资源处理器的自定义器。=========
//DispatcherServletPath  
//ServletRegistrationBean   给应用注册Servlet、Filter....
```



**下面有一个方法专门处理资源**

```java
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    if (!this.resourceProperties.isAddMappings()) {
        logger.debug("Default resource handling disabled");
    } else {
        Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
        CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
        //webjars的规则
        if (!registry.hasMappingForPattern("/webjars/**")) {
            this.customizeResourceHandlerRegistration(registry.addResourceHandler(new String[]											  {"/webjars/**"}).addResourceLocations(
                new String[]{"classpath:/META-INF/resources/webjars/"}).                                    						setCachePeriod(this.getSeconds(cachePeriod)).setCacheControl(cacheControl));
        }
		//静态资源的规则
        String staticPathPattern = this.mvcProperties.getStaticPathPattern();
        if (!registry.hasMappingForPattern(staticPathPattern)) {
            this.customizeResourceHandlerRegistration(registry.addResourceHandler(new String[]{staticPathPattern}).addResourceLocations(WebMvcAutoConfiguration.getResourceLocations(this.resourceProperties.getStaticLocations())).setCachePeriod(this.getSeconds(cachePeriod)).setCacheControl(cacheControl));
        }

    }
}
```



**默认路径在ResourceProperties类里：**

```java
@ConfigurationProperties(
    prefix = "spring.resources",
    ignoreUnknownFields = false
)
public class ResourceProperties {
    private static final String[] CLASSPATH_RESOURCE_LOCATIONS = new String[]{"classpath:/META-INF/resources/", "classpath:/resources/", "classpath:/static/", "classpath:/public/"};
    private String[] staticLocations;
```



### 3、请求参数处理

#### ###请求映射

**（1）RestFul使用**

- 一般是`@xxxMapping`注解；
- Rest风格支持（*使用**HTTP**请求方式动词来表示对资源的操作*）

- - *以前的请求路径：**/getUser*  *获取用户*   */deleteUser* *删除用户*   */editUser*  *修改用户*    */saveUser* *保存用户*

  - *现在的请求路径： /user*   *GET-**获取用户*   *DELETE-**删除用户*   *PUT-**修改用户*    *POST-**保存用户*

  - 核心Filter；HiddenHttpMethodFilter

    ```java
        @Bean
        @ConditionalOnMissingBean(HiddenHttpMethodFilter.class)
        @ConditionalOnProperty(prefix = "spring.mvc.hiddenmethod.filter", name = "enabled", matchIfMissing = false)
        public OrderedHiddenHttpMethodFilter hiddenHttpMethodFilter() {
            return new OrderedHiddenHttpMethodFilter();
        }
    ```

- - - 用法： 表单标签里的method=post，设定一个隐藏域 name="_method"，value="put"

    - SpringBoot中手动开启

    - ```yaml
      spring:
        mvc:
          #开启表单rest功能
          hiddenmethod:
            filter:
              enabled: true
      ```

- - 扩展：如何把_method 这个名字换成我们自己喜欢的。

    ```java
    //自定义filter
        @Bean
        public HiddenHttpMethodFilter hiddenHttpMethodFilter(){
            HiddenHttpMethodFilter methodFilter = new HiddenHttpMethodFilter();
            methodFilter.setMethodParam("_m");
            return methodFilter;
        }
    ```

- **(2)RestFul原理**

- Rest原理（表单提交要使用REST的时候）

- - 表单提交会带上**_method=PUT**
  - **请求过来会被**HiddenHttpMethodFilter拦截

- - - 请求是否正常，并且是POST

- - - - 获取到**_method**的值。

      - 兼容以下请求；**PUT**.**DELETE**.**PATCH**

      - **原生request（post），包装模式requesWrapper重写了getMethod方法，返回的是传入的值。**

      - **过滤器链放行的时候用wrapper。以后的方法调用getMethod是调用requesWrapper的**

        

  - **Rest使用客户端工具，**

  - - 如PostMan直接发送Put、delete等方式请求，无需Filter。
  
    

#### 请求映射原理：

![20210205005703527](C:\Users\fat man\Desktop\笔记\Spring\img\20210205005703527.png)

SpringMVC功能分析都从 `org.springframework.web.servlet.DispatcherServlet` -> `doDispatch()`开始



所有的请求映射都在HandlerMapping中：

SpringBoot自动配置欢迎页的 WelcomePageHandlerMapping 。访问 /能访问到index.html；

SpringBoot自动配置了默认 的 RequestMappingHandlerMapping

请求进来，挨个尝试所有的HandlerMapping看是否有请求信息。

如果有就找到这个请求对应的handler
如果没有就是下一个 HandlerMapping
我们需要一些自定义的映射处理，我们也可以自己给容器中放HandlerMapping。自定义 HandlerMapping


### 4、拦截器

#### （1）继承HandlerInterceptor 接口

```java
package com.wenwen.cs.demo.interceptor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

//登录检查，
// 1、配置好拦截器要拦截哪些请求
// 2、把这些容器放在容器中
@Slf4j
public class LoginInterceptor implements HandlerInterceptor {
    //目标方法执行之前
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String requestURI = request.getRequestURI();
        //打印拦截到的路径
        log.info("preHandle拦截的请求路径是{}",requestURI);
        //登录检查逻辑
        HttpSession session = request.getSession();
        Object user = session.getAttribute("user");
        if (user != null){
            //放行
            return true;
        }
        //拦截住。未登录。跳转到登录页
        request.setAttribute("msg","请先登录！");
        request.getRequestDispatcher("/").forward(request,response);
        return false;
    }
    //目标方法执行之后

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        log.info("postHandle执行{}",modelAndView);
    }

    //页面渲染之后
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        log.info("afterCompletion执行异常{}",ex);
    }
}
```

#### （2）配置拦截器

```java
package com.wenwen.cs.demo.config;

import com.wenwen.cs.demo.interceptor.LoginInterceptor;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

/**
 * 1、编写一个拦截器实现HandlerInterceptor接口
 * 2、拦截器注册到容器中（实现WebMvcConfigurer的addInterceptors）
 * 3、指定拦截规则【如果是拦截所有，静态资源也会被拦截】
 */
@Configuration
public class AdminWebConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        //注册拦截器
        registry.addInterceptor(new LoginInterceptor())
                //所有请求都被拦截
                .addPathPatterns("/**")
                //放行的请求
                .excludePathPatterns("/","/login","/css/**","/fonts/**","/images/**","/js/**");
    }
}
```

#### (3)拦截器原理

1、根据当前请求，找到**HandlerExecutionChain【**可以处理请求的handler以及handler的所有 拦截器】

2、先来**顺序执行** 所有拦截器的 preHandle方法

- 1、如果当前拦截器prehandler返回为true。则执行下一个拦截器的preHandle
- 2、如果当前拦截器返回为false。直接    倒序执行所有已经执行了的拦截器的  afterCompletion；

**3、如果任何一个拦截器返回false。直接跳出不执行目标方法**

**4、所有拦截器都返回True。执行目标方法**

**5、倒序执行所有拦截器的postHandle方法。**

**6、前面的步骤有任何异常都会直接倒序触发** afterCompletion

7、页面成功渲染完成以后，也会倒序触发 afterCompletion



![image-20210616193900984](C:\Users\fat man\Desktop\笔记\SpringBoot\imgs\image-20210616193900984.png)



### 5、文件上传

```html
html:
<form action="/upload" method="post" enctype="multipart/form-data">
	//单文件上传
	<input type="file" name="headerImg" id="exampleInputfile">
	//多文件上传
	<input type="file" name="photos" mutiple>
</form>
```

```java
/**
     * MultipartFile 自动封装上传过来的文件
     * @param email
     * @param username
     * @param headerImg
     * @param photos
     * @return
     */
    @PostMapping("/upload")
    public String upload(@RequestParam("email") String email,
                         @RequestParam("username") String username,
                         @RequestPart("headerImg") MultipartFile headerImg,
                         @RequestPart("photos") MultipartFile[] photos) throws IOException {

        log.info("上传的信息：email={}，username={}，headerImg={}，photos={}",
                email,username,headerImg.getSize(),photos.length);

        if(!headerImg.isEmpty()){
            //保存到文件服务器，OSS服务器
            String originalFilename = headerImg.getOriginalFilename();
            //写入计算机硬盘
            headerImg.transferTo(new File("H:\\cache\\"+originalFilename));
        }

        if(photos.length > 0){
            for (MultipartFile photo : photos) {
                if(!photo.isEmpty()){
                    String originalFilename = photo.getOriginalFilename();
                    photo.transferTo(new File("H:\\cache\\"+originalFilename));
                }
            }
        }


        return "main";
    }
```

> 上传的时候springboot会限制你的上传文件大小，可以去yaml文件或properties配置文件进行修改

## 七、数据访问

### 1.1、数据库源的自动配置-HikariDataSource

**(1)导入JDBC场景**

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jdbc</artifactId>
        </dependency>
```

> 点击查看spring-boot-starter-data-jdbc依赖，发现里面导入了Hikari数据源、spring-jdbc还有事务。

```xml
    <dependency>
      <groupId>com.zaxxer</groupId>
      <artifactId>HikariCP</artifactId>
      <version>3.4.5</version>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jdbc</artifactId>
      <version>5.2.12.RELEASE</version>
      <scope>compile</scope>
    </dependency>
```

![image-20210617133536930](C:\Users\fat man\Desktop\笔记\SpringBoot\imgs\image-20210617133536930.png)

疑问：为什么官方没有导入数据库驱动？

答：是因为官方不知道我们接下来要操作什么数据库。

**（2）导入数据库驱动**

```xml
spring-boot默认版本：<mysql.version>8.0.22</mysql.version>

```

想要修改版本
1、直接依赖引入具体版本（maven的就近依赖原则）
2、重新声明版本（maven的属性的就近优先原则）

```xml
直接依赖：
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
<!--            <version>5.1.47</version>-->
        </dependency>

重新声明：
    <properties>
        <java.version>1.8</java.version>
        <mysql.version>5.1.47</mysql.version>
    </properties>
```

#### 1.2、分析自动配置

**（1）自动配置的类**

> 这些类都在spring-boot-autoconfiguration包里能找到

- DataSourceAutoConfiguration ： 数据源的自动配置
  - 修改数据源相关的配置：**spring.datasource**
  - **数据库连接池的配置，是自己容器中没有DataSource才自动配置的**
  - 底层配置好的连接池是：**HikariDataSource**

- DataSourceTransactionManagerAutoConfiguration： 事务管理器的自动配置
- JdbcTemplateAutoConfiguration： **JdbcTemplate的自动配置，可以来对数据库进行crud**
  - 可以修改这个配置项`@ConfigurationProperties(prefix = **"spring.jdbc"**)` 来修改JdbcTemplate
  - `@Bean@Primary`    JdbcTemplate；容器中有这个组件，自动放进容器中的

- JndiDataSourceAutoConfiguration： jndi的自动配置
- XADataSourceAutoConfiguration： 分布式事务相关的

**（2）修改配置项**

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/my_book
    username: root
    password: 123456
    driver-class-name: com.mysql.jdbc.Driver
```

（3）测试

```java
@SpringBootTest
class DemoApplicationTests {

    @Autowired
    JdbcTemplate jdbcTemplate;
    @Test
    void contextLoads() {
        Long num = jdbcTemplate.queryForObject("select count(*) from t_book",Long.class);
        System.out.println("书本总共有："+num);
    }

}
```

### 2、整合Mybatis

> SpringBoot官方的Starter：格式一般都是spring-boot-starter-xxx
>
> 第三方的格式为： xxx-spring-boot-starter

导入starter

```xml
 <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.4</version>
</dependency>
```

![image-20210617230138049](C:\Users\fat man\Desktop\笔记\SpringBoot\imgs\image-20210617230138049.png)

#### （1）xml配置模式

- 全局配置文件
- SqlSessionFactory: 自动配置好了

- SqlSession：自动配置了的 **SqlSessionTemplate 组合了SqlSession**
- @Import(**AutoConfiguredMapperScannerRegistrar**.**class**）；

- Mapper： 只要我们写的操作MyBatis的接口标注了 **@Mapper 就会被自动扫描进来**

```java
@EnableConfigurationProperties(MybatisProperties.class) ： MyBatis配置项绑定类。
@AutoConfigureAfter({ DataSourceAutoConfiguration.class, MybatisLanguageDriverAutoConfiguration.class })
public class MybatisAutoConfiguration{}

//yaml、properties配置前缀
@ConfigurationProperties(prefix = "mybatis")
public class MybatisProperties
```

配置yaml文件

```yaml
mybatis:
  mapper-locations: classpath:mybatis/mapper/*.xml  #sql映射位置
#  config-location: classpath:mybatis/mybatis-config.xml  #全局配置文件位置
  configuration:  #开启驼峰命名方便mapper属性对应数据库字段
    map-underscore-to-camel-case: true
```

编写接口和mapper

```java
@Mapper
public interface UserMapper {
    public List<User> getUserList();
}
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.wenwen.cs.demo.mapper.UserMapper">
    <select id="getUserList" resultType="com.wenwen.cs.demo.pojo.User">
        select * from t_user
    </select>
</mapper>
```

最后进行测试：

```java
@SpringBootTest
class DemoApplicationTests {
    @Autowired
    UserMapper userMapper;
    @Test
    void contextLoads() {
        List<User> userList = userMapper.getUserList();
        for (User user : userList){
            System.out.println(user);
        }
    }

}
```

总结：配置流程

- 导入mybatis官方starter
- 编写mapper接口。记得标注`@Mapper`注解

- 编写sql映射文件并绑定mapper接口
- 在application.yaml中指定Mapper配置文件的位置，以及指定全局配置文件的信息 （建议；**配置在mybatis.configuration**）

#### （2）注解配置模式

```java
@Mapper
public interface UserMapper {

    @Select("select * from t_user where id = #{id}")
    public User getUserById(Integer id);
}
```

> 复杂的话进行混合开发，也就是注解加xml配置

**总结：**

- 引入mybatis-starter
- **配置application.yaml中，指定mapper-location位置即可**

- 编写Mapper接口并标注`@Mapper`注解
- 简单方法直接注解方式

- 复杂方法编写mapper.xml进行绑定映射
- *`@MapperScan("mapper所在的path reference也就是他的位置")` 简化，其他的接口就可以不用标注`@Mapper`注解*

### 3、整合Mybatis-Plus

**（1）什么是Mybatis-Plus**

是一个 [MyBatis](http://www.mybatis.org/mybatis-3/) 的增强工具，在 MyBatis 的基础上只做增强不做改变，为简化开发、提高效率而生。

详细可以去查看Mybatis-Plus官网

建议安装MybatisX插件

**（2）整合Mybatis-Plus**

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.3.2</version>
</dependency>
```

![image-20210618111337603](C:\Users\fat man\Desktop\笔记\SpringBoot\imgs\image-20210618111337603.png)

> 这个注解导入的包包括了jdbc何Mybatis核心包

**自动配置**

- MybatisPlusAutoConfiguration 配置类，MybatisPlusProperties 配置项绑定。**mybatis-plus：xxx 就是对****mybatis-plus的定制**
- **SqlSessionFactory 自动配置好。底层是容器中默认的数据源**

- **mapperLocations 自动配置好的。有默认值。****classpath\*:/mapper/\**/\*.xml；任意包的类路径下的所有mapper文件夹下任意路径下的所有xml都是sql映射文件。  建议以后sql映射文件，放在 mapper下**
- **容器中也自动配置好了** **SqlSessionTemplate**

- **@Mapper 标注的接口也会被自动扫描；
- **建议直接使用** @MapperScan(**"mapper所在的path reference也就是他的位置"**) 批量扫描就行



**优点--便捷开发：**

-  只需要我们的Mapper继承 **BaseMapper** 就可以拥有crud能力

**（3）创建接口和Mapper.xml文件**

```java
@Mapper
@TableName("department")
public interface DepartmentMapper extends BaseMapper<Department> {

}
```

> mapper.xml文件暂时为空,因为暂无使用

测试

```java
@Test
void testMybatisPlus(){
    Department department = departmentMapper.selectById(3);
    System.out.println(department);
}
```

> 这里我遇到了一个小错误，是因为在使用Mapper的selectById(3)方法中查询的数据为null，Department实体类的主键名不是id，所以我需要在实体类进行注解说明。

```java
@Data
public class Department {
    @TableId("departmentid")//标注于此
    private int departmentid;
    private String departmentname;
}
```



#### ###规范开发流程

**（1）创建实体类**

```java
@Data
@TableName("t_user")//标注这个注解说明此实体类对应的数据库名
public class User {
    private int id;
    private String username;
    private String password;
    private String email;
}
```

**（2）创建Service接口**

```java
public interface UserService extends IService<User> {
}
```

**（3）创建ServiceImpl**

```java
@Service
//ServiceImpl<UserMapper, User>操作哪张表用哪个Mapper，右边参数为要操作的Pojo
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService {

}
```

**（4）测试**

```java
@Autowired
UserService userService;
```

```java
@Test
void testUser(){
    User user = userService.getById("1");
    System.out.println(user);
}
```

#### ###分页功能

在上面的基础上进行编写（使用mabatis-plus）

**（1）编写Controller**

```java
@Controller
public class UserController {
    @Autowired
    UserService userService;
    @GetMapping("/getUserList")
    public String getUserList(Model model, @RequestParam(value = "pg",defaultValue = "1") Integer pg){
        List<User> userList = userService.list();
        model.addAttribute("userList",userList);


        //分页查询数据
        Page<User> userPage = new Page<>(pg, 2);
        //分页查询的结果
        Page<User> page = userService.page(userPage,null);
        model.addAttribute("page",page);

        return "pageList";
    }
}
```

**（2）编写MybatisPlusConfig配置类**

```java
@Configuration
public class MybatisPlusConfig {

    // 旧版
    @Bean
    public PaginationInterceptor paginationInterceptor() {
        PaginationInterceptor paginationInterceptor = new PaginationInterceptor();
        // 设置请求的页面大于最大页后操作， true调回到首页，false 继续请求  默认false
         paginationInterceptor.setOverflow(false);
        // 设置最大单页限制数量，默认 500 条，-1 不受限制
         paginationInterceptor.setLimit(500);
        // 开启 count 的 join 优化,只针对部分 left join
        paginationInterceptor.setCountSqlParser(new JsqlParserCountOptimize(true));
        return paginationInterceptor;
    }
}
```

**（3）前端pageList**

```html
<body>
<table>
    <tr th:each="user : ${page.records}">
        <td th:text="${user.username}"></td>
        <td th:text="${user.password}"></td>
        <td th:text="${user.email}"></td>
    </tr>

</table>
<p>当前第[[${page.current}]]</p>
<p>总共[[${page.pages}]]页</p>
<p>共[[${page.total}]]条记录</p>
<ul>
    <li th:each="num:${#numbers.sequence(1,page.total)}">
        //上面为thmeleaf语法，产生1-page.total个数字
        <a th:href="@{/getUserList(pg=${num})}">[[${num}]]</a>
    </li>
</ul>
</body>
```

#### ###删除功能

```html
<tr th:each="user : ${page.records}">
    <td th:text="${user.username}"></td>
    <td th:text="${user.password}"></td>
    <td th:text="${user.email}"></td>
    <td><a th:href="@{/user/delete/{id}(id=${user.id},pg=${page.current})}" type="button">删除</a></td>
</tr>
```

```java
//编写controller
@GetMapping("/user/delete/{id}")
public String deleteUser(@PathVariable("id") Integer id,
                         @RequestParam("pg") Integer pg,
                         RedirectAttributes ra){

    userService.removeById(id);
    //这种方法相当于在重定向链接地址追加传递的参数
    ra.addAttribute("pg",pg);
    return "redirect:/getUserList";

}
```

## 八、单元测试

## 九、高级特性

### 一.自定义starter

1、starter启动原理

- starter-pom引入autoconfiguer包

![image-20211121170824600](C:\Users\fat man\Desktop\笔记\SpringBoot\imgs\image-20211121170824600.png)

- autoconfigure包中配置使用`META-INF/spring.factories`中`EnableAutoConfiguration`的值，使得项目启动加载指定的自动配置类

- 目标：创建HelloService的自定义starter。


- 创建两个工程，分别命名为hello-spring-boot-starter（普通Maven工程），hello-spring-boot-starter-autoconfigure（需用用到Spring Initializr创建的Maven工程）。


- hello-spring-boot-starter无需编写什么代码，只需让该工程引入hello-spring-boot-starter-autoconfigure依赖

- 创建4个文件：
  com/lun/hello/auto/HelloServiceAutoConfiguration
  com/lun/hello/bean/HelloProperties
  com/lun/hello/service/HelloService
  src/main/resources/META-INF/spring.factories

  

  **HelloServiceAutoConfiguration**

  ```java
  import com.lun.hello.bean.HelloProperties;
  import com.lun.hello.service.HelloService;
  import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
  import org.springframework.boot.context.properties.EnableConfigurationProperties;
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;
  
  @Configuration
  @ConditionalOnMissingBean(HelloService.class)
  @EnableConfigurationProperties(HelloProperties.class)//默认HelloProperties放在容器中
  public class HelloServiceAutoConfiguration {
  
      @Bean
      public HelloService helloService(){
          return new HelloService();
      }
  }
  ```

```java
@ConfigurationProperties("hello")
public class HelloProperties {
    private String prefix;
    private String suffix;

    public String getPrefix() {
        return prefix;
    }

    public void setPrefix(String prefix) {
        this.prefix = prefix;
    }

    public String getSuffix() {
        return suffix;
    }

    public void setSuffix(String suffix) {
        this.suffix = suffix;
    }
}

```

**HelloService**



```java
/**

默认不要放在容器中
*/
public class HelloService {

@Autowired
private HelloProperties helloProperties;

public String sayHello(String userName){
    return helloProperties.getPrefix() + ": " + userName + " > " + helloProperties.getSuffix();
}
}
```

- **src/main/resources/META-INF/spring.factories**

```properties
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\ com.lun.hello.auto.HelloServiceAutoConfiguration
```

最后使用maven插件，将两工程install到本地，其他工厂直接引入这个hello-spring-boot-starter即可。
