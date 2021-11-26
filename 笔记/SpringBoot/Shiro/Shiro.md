# Shiro(代码在Shiro-demo)

## 1、SpringBoot整合Shiro

### （1）导入依赖

```xml
Shiro整合Spring
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring</artifactId>
    <version>1.4.0</version>
</dependency>
```

### （2）创建自定义类

```java
//自定义的类UserRealm需要extends AuthorizingRealm
public class UserRealm extends AuthorizingRealm {
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        System.out.println("执行了=>授权doGetAuthorizationInfo");
        return null;
    }

    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        System.out.println("执行了=>认证doGetAuthenticationInfo");
        return null;
    }
}
```

### （3）创建配置类

```java
@Configuration
public class ShiroConfig {
    //ShiroFilterFactoryBean:3
    @Bean
    public ShiroFilterFactoryBean getShiroFilterFactoryBean(@Autowired DefaultWebSecurityManager defaultWebSecurityManager){
        ShiroFilterFactoryBean bean = new ShiroFilterFactoryBean();
        //设置安全管理器
        bean.setSecurityManager(defaultWebSecurityManager);
        return bean;
    }

    //DefaultWebSecurityManager:2
    @Bean
    public DefaultWebSecurityManager getDefaultWebSecurityManager(@Autowired UserRealm userRealm){
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        //关联UserRealm
        securityManager.setRealm(userRealm);
        return securityManager;
    }

    //创建realm对象，需要自定义类：1
    @Bean
    public UserRealm userRealm(){
        return new UserRealm();
    }
```

## 2、实现登录拦截

### **（1）修改ShiroConfig**

```java
public class ShiroConfig {
    //ShiroFilterFactoryBean:3
    @Bean
    public ShiroFilterFactoryBean getShiroFilterFactoryBean(@Autowired DefaultWebSecurityManager defaultWebSecurityManager){
        ShiroFilterFactoryBean bean = new ShiroFilterFactoryBean();
        //设置安全管理器
        bean.setSecurityManager(defaultWebSecurityManager);

        //添加shiro的内置过滤器
        /*
        * anon:无需认证就可以访问
        * authc:必须认证了才能访问
        * user:必须拥有 记住我 功能才能用
        * perms：拥有对某个资源的权限才能访问
        * role：拥有某个角色权限才能访问
        * */
        Map<String,String> filterMap = new LinkedHashMap<>();
        filterMap.put("/user/toAdd","authc");
        filterMap.put("/user/toUpdate","authc");
        bean.setFilterChainDefinitionMap(filterMap);
        //设置登录的请求页面（没有通过就跳转这里）
        bean.setLoginUrl("/user/toLogin");
        return bean;
    }
```

### **(2)在Controller编写login方法**

```java
@RequestMapping("/login")
public String login(String username,String password,Model model){
    //获取当前的用户
    Subject subject = SecurityUtils.getSubject();
    //封装用户的登录数据
    UsernamePasswordToken token = new UsernamePasswordToken(username,password);

    try{
        subject.login(token);//执行登录方法，如果没有异常就说明可以了
        return "index";
    }catch (UnknownAccountException e){//用户名不存在
        model.addAttribute("msg","用户名错误");
        return "login";
    }catch (IncorrectCredentialsException e){//密码不存在
        model.addAttribute("msg","密码错误");
        return "login";
    }
}
```

### **(3)在UserRealm进行认证方法的编写**

```java
@Override
protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
    System.out.println("执行了=>认证doGetAuthenticationInfo");
    //用户名，密码
    String name = "root";
    String password = "123456";

    UsernamePasswordToken userToken = (UsernamePasswordToken)authenticationToken;
    if (!userToken.getUsername().equals(name)){
        return null;//抛出异常 UnknowAccountException
    }
    //密码认证，shiro帮我们做了
    return new SimpleAuthenticationInfo("",password,"");

}
```

### **（4）通过认证就可以跳转页面了**

## 3、连接数据库

### (1)整合Myabtis

### (2)编写UserRealm

```java
package com.wenwen.shirodemo.config;

import com.wenwen.shirodemo.pojo.User;
import com.wenwen.shirodemo.service.UserService;
import org.apache.shiro.SecurityUtils;
import org.apache.shiro.authc.*;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.authz.SimpleAuthorizationInfo;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.session.Session;
import org.apache.shiro.subject.PrincipalCollection;
import org.apache.shiro.subject.Subject;
import org.springframework.beans.factory.annotation.Autowired;


//自定义的类UserRealm需要extends AuthorizingRealm
public class UserRealm extends AuthorizingRealm {
    @Autowired
    UserService userService;
    //授权
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        System.out.println("执行了=>授权doGetAuthorizationInfo");
        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();


        //拿到当前登录的对象
        Subject subject = SecurityUtils.getSubject();
        User currentUser = (User)subject.getPrincipal();//拿到User对象
        //设置当前用户的权限
        info.addStringPermission(currentUser.getRole());//可以从currentUser.getxxx获取权限，从而去进行设置权限
        return info;
    }
    //认证
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        System.out.println("执行了=>认证doGetAuthenticationInfo");

        UsernamePasswordToken userToken = (UsernamePasswordToken)authenticationToken;
        //连接真实的数据库
        User user = userService.getUserByUsername(userToken.getUsername());
        if (user==null){ //没有这个用户
            return null;
        }
        //登录成功放进session里面（Shiro的session）
        Subject currentSubject = SecurityUtils.getSubject();
        Session session = currentSubject.getSession();
        session.setAttribute("loginUser",user);

        //密码认证，shiro帮我们做了
        return new SimpleAuthenticationInfo(user,user.getPassword(),"");

    }
}

```

### (3)整合thymeleaf

```xml
<dependency>
    <groupId>org.thymeleaf</groupId>
    <artifactId>thymeleaf-spring5</artifactId>
</dependency>
<dependency>
    <groupId>org.thymeleaf.extras</groupId>
    <artifactId>thymeleaf-extras-java8time</artifactId>
</dependency>
		//整合thymeleaf和shiro
        <dependency>
            <groupId>com.github.theborakompanioni</groupId>
            <artifactId>thymeleaf-extras-shiro</artifactId>
            <version>2.0.0</version>
        </dependency>
```

### (4)编写ShiroConfig

```java
package com.wenwen.shirodemo.config;

import at.pollux.thymeleaf.shiro.dialect.ShiroDialect;
import org.apache.shiro.spring.web.ShiroFilterFactoryBean;
import org.apache.shiro.web.mgt.DefaultWebSecurityManager;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.LinkedHashMap;
import java.util.Map;

@Configuration
public class ShiroConfig {
    //ShiroFilterFactoryBean:3
    @Bean
    public ShiroFilterFactoryBean getShiroFilterFactoryBean(@Autowired DefaultWebSecurityManager defaultWebSecurityManager){
        ShiroFilterFactoryBean bean = new ShiroFilterFactoryBean();
        //设置安全管理器
        bean.setSecurityManager(defaultWebSecurityManager);

        //添加shiro的内置过滤器
        /*
        * anon:无需认证就可以访问
        * authc:必须认证了才能访问
        * user:必须拥有 记住我 功能才能用
        * perms：拥有对某个资源的权限才能访问
        * role：拥有某个角色权限才能访问
        * */
        Map<String,String> filterMap = new LinkedHashMap<>();
        //授权，正常情况下，没有授权会跳转到未授权页面
        filterMap.put("/user/toAdd","perms[1]");
        filterMap.put("/user/toUpdate","perms[2]");
        filterMap.put("/user/*","authc");
        bean.setFilterChainDefinitionMap(filterMap);
        //设置登录的请求页面
        bean.setLoginUrl("/user/toLogin");
        //设置未授权页面
        bean.setUnauthorizedUrl("/noAuthor");
        return bean;
    }

    //DefaultWebSecurityManager:2
    @Bean
    public DefaultWebSecurityManager getDefaultWebSecurityManager(@Autowired UserRealm userRealm){
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        //关联UserRealm
        securityManager.setRealm(userRealm);
        return securityManager;
    }

    //创建realm对象，需要自定义类：1
    @Bean
    public UserRealm userRealm(){
        return new UserRealm();
    }

    //整合ShiroDialect：用来整合shiro thymeleaf
    @Bean
    public ShiroDialect getShiroDialect(){
        return new ShiroDialect();
    }
}
```

### (5)Controller的Login方法

```java
@RequestMapping("/user/toLogin")
public String toLogin(){
    return "login";
}
@RequestMapping("/login")
public String login(String username,String password,Model model){
    //获取当前的用户
    Subject subject = SecurityUtils.getSubject();
    //封装用户的登录数据
    UsernamePasswordToken token = new UsernamePasswordToken(username,password);

    try{
        subject.login(token);//执行登录方法，如果没有异常就说明可以了
        return "index";
    }catch (UnknownAccountException e){//用户名不存在
        model.addAttribute("msg","用户名错误");
        return "login";
    }catch (IncorrectCredentialsException e){//密码不存在
        model.addAttribute("msg","密码错误");
        return "login";
    }
}
```

