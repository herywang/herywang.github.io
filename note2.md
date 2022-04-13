# Apache Shiro源码阅读

最近在腾讯实习参与开发Apache-InLong开源项目，其中负责的模块使用到了apache-shiro对访问权限进行控制，因此顺便把Apache-Shiro部分源码进行阅读。  

#### 1. Authorization(授权，接口访问控制) 和 Authentication(认证，登陆)基础
Apache-Shiro中对接口访问控制最常用的两种配置是：`anno`和`authc`, 其中每种配置都代表一个过滤器，接口如果配置`authc`表示当前接口需要登录才能访问, `anno`表示无需什么特殊权限就能够访问. Spring-Boot中具体如何整合Apache-shiro进行权限控制的这里就不在赘述, 我们从入口处看起:  

```java
    @Bean
    public ShiroFilterFactoryBean shiroFilter(SecurityManager securityManager) {
        return inLongShiro.getShiroFilter(securityManager);
    }
```  

默认情况下`authc`对应的过滤器是: `org.apache.shiro.web.filter.authc.FormAuthenticationFilter`, 点进入源码中会发现这个过滤器的实现类继承了一大堆东西, 一层又一层的, 非常复杂, 但是我们最终的目的就是实现一个`Filter`接口就行了,  
在Apache-Inlong开源项目中我们就自定义了`authc`的过滤器`AuthenticationFilter.class`

```java
    @Override
    public ShiroFilterFactoryBean getShiroFilter(SecurityManager securityManager) {
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        shiroFilterFactoryBean.setSecurityManager(securityManager);
        // anon: can be accessed by anyone, authc: only authentication is successful can be accessed
        Map<String, Filter> filters = new LinkedHashMap<>();
        filters.put("authc", new AuthenticationFilter());
        shiroFilterFactoryBean.setFilters(filters);
        Map<String, String> pathDefinitions = new LinkedHashMap<>();
        // login, register request
        pathDefinitions.put("/anno/**/*", "anon");

        // swagger api
        pathDefinitions.put("/doc.html", "anon");
        pathDefinitions.put("/v2/api-docs/**/**", "anon");
        pathDefinitions.put("/webjars/**/*", "anon");
        pathDefinitions.put("/swagger-resources/**/*", "anon");
        pathDefinitions.put("/swagger-resources", "anon");

        // openapi
        pathDefinitions.put("/openapi/**/*", "anon");
        pathDefinitions.put("/**", "authc");
        shiroFilterFactoryBean.setFilterChainDefinitionMap(pathDefinitions);
        return shiroFilterFactoryBean;
    }

```

Apache-Shiro的生命周期从Spring的Configure类中通过注入`ShiroFilterFactoryBean`类开始, 其初始化需要依赖`CredentialsMatcher`, `SecurityManager` 以及自定义的 `AuthorizingRealm`等类信息, 类之间的聚合关系如下图所示:  
![UML图](imgs/note2.1.jpg)  
显然可见, ShiroFilterFactoryBean就是一个过滤器链, 里面具体怎么实现靠用户自定义的securityManager等相关类中的逻辑实现.  


#### 深度剖析ShiroFilterFactoryBean
先看一下其实现的接口:  

<img src="./imgs/note2.2.png" width=400px/>  

看到第一个接口`FactoryBean`相信阅读多Spring源码的童鞋都不会太陌生, 这个时候第一想到的就是看看`ShiroFilterFactoryBean`中的`getObject()`方法. 
> (不熟悉FactoryBean接口的童鞋参考下我的另一篇文章:[关于FactoryBean的那些事](./spring/factorybean.md)) 

在`getObject`方法中会调用`createInstance()`来初始化一个`AbstractShiroFilter`实例, 具体的实现类是`SpringShiroFilter`, 并且将我们自定义的`SecurityManager`作为参数传递到`SpringShrioFilter`实例中. 
![](./imgs/spring1.png)  
查看`SpringShrioFilter`类结构图可发现, 其