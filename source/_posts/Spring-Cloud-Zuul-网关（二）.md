---
title: Spring Cloud Zuul 网关（二）
date: 2018-04-20 16:02:09
tags:
- Spring Cloud
categories:
- Tech
---

上篇文章 [Spring Cloud Zuul 网关（一）](http://blog.ulyssesss.com/2018/04/17/Spring-Cloud-Zuul-%E7%BD%91%E5%85%B3%EF%BC%88%E4%B8%80%EF%BC%89/) 介绍了 Spring Cloud Zuul 的基本使用方法，本篇文章会对 Zuul 的过滤器机制、hystrix 及 ribbon 支持做进一步介绍。





<!-- more -->

## 过滤器

Zuul 中的过滤器负责在转发外部请求的过程中对处理过程进行干预，在实际运行时，路由映射和请求转发等步骤都是由不同的过滤器完成。

Spring Cloud Zuul 中的过滤器包含以下 4 个基本特征：

* 过滤类型

> 过滤器所处的生命周期，Zuul 默认定义了 4 种不同生命周期，分别为 pre（请求被路由前调用）、routing（在路由请求时被调用）、post（在 routing 和 error 过滤器之后被调用） 和 error（发生错误时被调用）。

* 执行顺序

> 过滤器执行顺序，数值越小优先级越高

* 执行条件

> 通过返回的 boolean 值判断过滤是否执行

* 具体操作

> 过滤器的具体逻辑



### 生命周期

外部 HTTP 请求到达网关直到返回请求结果的整个生命周期如图：

![生命周期](http://ofu79o924.bkt.clouddn.com/201804201.png)

请求到达网关时首先被 pre 类型的过滤器处理，主要目的是在请求路由前做一些请求校验等前置加工。

完成 pre 阶段后进入 routing 请求转发阶段，将外部请求转发到具体服务实例。

routing 之后进入 post，此阶段过滤器不仅可以获取请求信息，还能获得服务实例返回的信息，做一些加工处理。

error 在上述三个阶段发生异常时出发，最后还是流向 post 类型的过滤器。



### 自定义过滤器

Zuul 允许通过定义过滤器实现对请求的拦截和过滤，实现时只需继承 ZuulFilter 抽象类并实现抽象方法。

```java
package com.ulyssesss.apigateway;

import com.netflix.zuul.ZuulFilter;
import com.netflix.zuul.context.RequestContext;
import org.springframework.stereotype.Component;

import javax.servlet.http.HttpServletRequest;

@Component
public class AccessFilter extends ZuulFilter {

    @Override
    public String filterType() {
        return "pre";
    }

    @Override
    public int filterOrder() {
        return 0;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request = ctx.getRequest();
        String token = request.getParameter("token");
        if (token == null) {
            ctx.setSendZuulResponse(false);//过滤请求，不对其进行路由
            ctx.setResponseBody("error token");
            ctx.setResponseStatusCode(401);
        }
        return null;
    }
}
```

定义 AccessFilter 过滤器后重新启动，向 api-gateway 发起请求，请求包含 token 参数时响应正常，不包含 token 参数时返回 error token，响应码 401。



## Ribbon 和 Hystrix 支持

spring-cloud-starter-zuul 包含对 spring-cloud-starter-ribbon 和 spring-cloud-starter-hystrix 模块的依赖，天然拥有线程隔离和断路器的自我保护功能。

需要注意的是通过传统路由方式配置时不会受到保护，也没有负载均衡能力。只有通过服务路由才会有自我保护和负载均衡的功能。

```properties
## 指定路由请求转发的超时时间
hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds=10000

## 指定路由转发请求创建连接的超时时间
ribbon.ConnectTimeout=250

## 指定路由转发请求处理过程的超时时间
ribbon.ReadTimeout=1000
```

Zuul 在默认配置下请求异常时不会发起重试，如需开启重试功能，首先需要添加 spring-retry 依赖。

```xml
<dependency>
    <groupId>org.springframework.retry</groupId>
    <artifactId>spring-retry</artifactId>
</dependency>
```

添加依赖后针对重试添加如下配置：

```properties
## 全局重试机制开关
zuul.retryable=true

## 关闭指定路由的重试机制
## zuul.routes.<route>.retryable
zuul.routes.hello-service.retryable=false

## 对当前服务实例的重试次数
ribbon.MaxAutoRetries=2
## 重试切换的实例数
ribbon.MaxAutoRetriesNextServer=0
```



[示例代码](https://github.com/Ulyssesss/spring-cloud-example) 欢迎 Star 