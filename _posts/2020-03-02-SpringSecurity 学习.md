---
layout: post
title: SpringSecurity学习
categories: [Security]
description: 
keywords: 
---

* content
{:toc}


## SpringSecurity 基本原理

<img src="https://www.miaomiaoqi.github.io/images/springsecurity/authen_1.png" alt="https://www.miaomiaoqi.github.io/images/springsecurity/authen_1.png" style="zoom: 25%;" />

SpringSecurity 的基本原理是内置了许多的过滤器, 所有的过滤器都可以根据我们的配置进行插拔式的配置, 其中绿色的部分是我们可控的, 其余的部分是必须存在的

## 自定义用户认证逻辑

配置类继承 WebSecurityConfigurerAdapter 重写 configure 方法



### 处理用户信息获取逻辑

实现 UserDetailsService 接口重写 loadUserByUsername 方法, 所有的用户信息都从底层数据库中查找, 返回给 SpringSecurity 进行密码校验

### 处理用户校验逻辑

在 UserDetailsService 的实现类中自己判断用户的状态返回 UserDetails 实现类, enabled(用户是否可用), accountNonExpired(账户没过期), credentialsNonExpired(密码没过期), accountNonLocked(用户没冻结), 返回给 SpringSecurity

### 处理密码加密解密

使用 PasswordEncoder 对密码进行加密, 注册的时候也要使用该方法进行加密, 可以自定义实现类



## 个性化用户认证流程

### 自定义登录页面

将认证失败转发到一个 controller 接口中, 接口返回 json 格式数据, 前端重定向到登录页面进行认证授权

### 自定义登录成功处理

SpringSecurity 登录成功后默认会再请求一次之前的请求, 实现 AuthenticationSuccessHandler 接口可以自定义登录成功的处理

### 自定义登录失败处理

实现 ImoocAuthenticationFailureHandler 接口可以自定义登录失败的处理





## 实现图形验证码功能

### 开发生成图形验证码

根据随机数生成图片

将随机数放到 session 中

将生成的图片写到接口的响应中

### 验证验证码

SpringSecurity 并没有提供验证码的校验过滤器, 需要自定义 Filter 实现验证码的校验





## 实现短信验证码登录

### 开发短信验证码

根据随机数生成图片

将随机数放到 session 中

将生成的验证码发送到手机端

### 短信验证码组件开发

**SpringSecurity 并没有提供短信验证码的登录功能, 所以我们需要自己实现 SmsAuthenticationTokenFilter**

<img src="https://www.miaomiaoqi.github.io/images/springsecurity/authen_2.png" alt="https://www.miaomiaoqi.github.io/images/springsecurity/authen_2.png" style="zoom: 25%;" />

自定义一个 **SmsAuthenticationFilter**, 根据手机号生成未认证的 **SmsAuthenticationToken**(封装登录信息, 身份认证之前封装的是手机号, 认证成功后封装的是用户信息), AuthenticationManager 会检索系统中所有的 Provider, 这时我们提供一个 **SmsAuthenticationProvider** 用这个 provider 来校验手机号的信息, provider 调用 UserDetailsService 获取用户信息进行认证, 认证成功的话将我们的 Token 置为已认证状态

验证短信验证码的过程还要单独写一个 filter 达到可复用的目的



## OAuth 协议简介

### OAuth 协议要解决的问题

<img src="https://www.miaomiaoqi.github.io/images/springsecurity/authen_3.png" alt="https://www.miaomiaoqi.github.io/images/springsecurity/authen_3.png" style="zoom: 50%;" />

用户将微信用户名和密码告诉第三方, 第三方程序员再向微信请求数据, 会存在如下风险

*   第三方应用可以访问用户在微信上的所有数据
*   用户只有修改密码, 才能收回授权
*   密码泄露的可能性大大提高

OAuth 就是用来解决上述问题的

### OAuth 协议中的各种角色

<img src="https://www.miaomiaoqi.github.io/images/springsecurity/authen_4.png" alt="https://www.miaomiaoqi.github.io/images/springsecurity/authen_4.png" style="zoom: 50%;" />

上图是 OAuth 的核心流程, 其中第 2 步同意授权又包含 4 种模式

*   **授权码模式(authorization code)**
*   **密码模式(resource owner password credentials)**
*   客户端模式(client credentials)
*   简化模式(implicit)

#### 授权码模式

<img src="https://www.miaomiaoqi.github.io/images/springsecurity/authen_5.png" alt="https://www.miaomiaoqi.github.io/images/springsecurity/authen_5.png" style="zoom: 50%;" />

用户同意授权的动作是在认证服务器上完成的, 认证服务器可以明确知道是用户同意授权了, 安全性高

第三方应用需要有一个后台服务器接收认证服务器返回的授权码

## SpringSocial

<img src="https://www.miaomiaoqi.github.io/images/springsecurity/authen_7.png" alt="https://www.miaomiaoqi.github.io/images/springsecurity/authen_7.png" style="zoom: 50%;" />

SpringSocial 将 OAuth 的流程封装成了 SocialAuthenticationFilter 中并加入到了过滤器链中

### SpringSocial 开发第三方登录

<img src="https://www.miaomiaoqi.github.io/images/springsecurity/authen_8.png" alt="https://www.miaomiaoqi.github.io/images/springsecurity/authen_8.png" style="zoom: 50%;" />

<img src="https://www.miaomiaoqi.github.io/images/springsecurity/authen_6.png" alt="https://www.miaomiaoqi.github.io/images/springsecurity/authen_6.png" style="zoom: 50%;" />

ServiceProvider(AbstractOAuth2ServiceProvider): 负责 1~6 步的流程

OAuth2Operations(OAuth2Template): 封装了 1~5 步

Api(AbstractOAuth2ApiBinding): 第 6 步封装接口的实现, 因为每个提供商的返回值都不一样



Connection(OAuth2Connection): 封装获取到的用户信息

ConnectionFactory(OAuth2ConnectionFactory): 负责创建 Connection 实例, 通过调用 ServiceProvider

ApiAdapter: 将服务商个性化的结果转化为标准的 OAuth 对象

