# Apache Shiro源码阅读

最近在腾讯实习参与开发Apache-InLong开源项目，其中负责的模块使用到了apache-shiro对访问权限进行控制，因此顺便把Apache-Shiro部分源码进行阅读。  

## Authorization(授权，接口访问控制) 和 Authentication(认证，登陆)
Apache-Shiro中对接口访问控制最常用的两种配置是：`anno`和`authc`, 其中每种配置都代表一个过滤器，配置`authc`表示当前接口需要登录才能访问, `anno`表示无需什么特殊权限就能够访问. Spring-Boot中具体如何整合Apache-shiro进行权限控制的这里就不在赘述, 我们从入口处看起:  
```JAVA
    @Bean
    public ShiroFilterFactoryBean shiroFilter(SecurityManager securityManager) {
        return inLongShiro.getShiroFilter(securityManager);
    }
```

Apache-Shiro的生命周期从Spring的Configure类中通过注入`ShiroFilterFactoryBean`类开始, 其初始化需要依赖`CredentialsMatcher`, `SecurityManager` 以及自定义的 `AuthorizingRealm`等类信息, 类之间的聚合关系如下图所示:   
![UML图](imgs/note2.1.jpg)  
显然可见, ShiroFilterFactoryBean就是一个过滤器链, 里面具体怎么实现靠用户自定义的securityManager等相关类中的逻辑实现.  
