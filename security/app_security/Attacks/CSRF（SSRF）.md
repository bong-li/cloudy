#CSRF

[toc]

### 概述

#### 1.CSRF（cross-site request forgery）
攻击者伪造当前用户的行为，让目标服务器误以为请求由当前用户发起

#### 2.SSRF（server-side request forgery）
服务器从用户的输入提取参数，向第三方发起请求，获取数据，再返回给用户
这样的情况下，会存在SSRF漏洞，攻击者可以利用该漏洞，探测服务器内部环境
此种攻击危害较小，而且较少发生

#### 2.实现CSRF攻击条件
* 用户处于登录状态
* 伪造的请求与正常应用请求一样
* 后台未对用户业务做合法性校验

#### 3.CSRF影响
* 可以执行某些操作（比如删除等）

***

### 防护

#### 1.添加中间环节

##### （1）添加确认过程
执行一些重要的操作时，服务端会返回信息让用户确认（此时攻击者没法收到该信息）

##### （2）添加验证码
本质跟确认过程一样

#### 2.验证用户请求的合法性
##### （1）利用token
用于攻击无法获取用户的token，所以无法伪造请求
