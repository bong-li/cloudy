### 基础
#### 1.目的：
* 机密性
* 完整性（即防止数据被篡改）
* 身份验证
#### 2.几种加密方式：
（1）单向加密		
>输入一样，输出一定一样  

>计算需要发送数据的特征值，并对特征值加密，然后连同数据一起发送给对方   

>对方同样计算接收数据的特征值，然后解密收到的特征值，并进行比较  

>从而判断数据是否完整  

（2）对称加密
>双方约定一个密码

>缺点：  
>>约定时是明文，容易被截取     
>>当与多个人通信时，需要不同的密钥，难管理  

（3）密钥交换（internet key exchange，IKE，该方式也属于对称加密）
>双方协商一个密码  

>>比如：

>>双方项约定两个数字 p 和 g，这两个数字所有人都可以看到  
>>A随机生成一个数 x，只有A自己知道  
>>B随机生成一个数 y，只有B自己知道    
  A发送 p+g+x的结果到B，然后B计算 p+g+x+y的值，就是协商好的密钥   
  同理，B发送 p+g+y的结果到A，然后A计算 p+g+x+y的值，就是协商好的密钥   
  然而其他人都计算不出这个值，从而实现加密  

>>缺点：

>>无法实现身份验证  

（4）非对称加密（很少用来加密数据，速度太慢）
>用私钥加密数据，可以实现身份验证  
>用公钥加密数据，可以保证数据机密性   
>>应用：  
>>用私钥加密特征码，保证完整性和身份验证   

#### 3.实际双方通信时，加密的方式
（1）利用自己的私钥加密信息特征码，从而保证了完整性和身份验证
（2）其中利用对方的公钥加密对称加密密钥，然后发送给对方，从而实现了机密性

#### 4.第三方机构（Certificate Authority）
  是为了非对称加密设置的，确保所有人能够获得彼此正确的公钥

>ca机构本身拥有公钥和私钥，ca机构的作用如下：

（1）认证（即进行数字签名）
  用私钥给别人的公钥的特征码进行加密
  其他人可以用ca机构的公钥解密该公钥
  从而保证了完整性（即该公钥没有问题）和身份验证（即该公钥是经过ca认证的）

（2）数字证书
  经过ca机构认证的公钥
  数字证书的格式为x509

（3）ca证书（格式为x509）
  ca机构自签署证书，即ca机构使用自己的私钥给自己的公钥进行签名
  CA根证书被内置在浏览器中

（4）维护CRL（certificate revocation list，证书吊销列表）
  当某个人的密钥丢了，则其公钥就不可信了，就应该放入CRL中

（5）CSR请求（certificate signature request）
  ca机构根据CSR请求对相应公钥进行签名

#### 5.SSL和TLS区别：
  是两种协议，都是在应用层和传输层之间的，用于给数据加密
  SSL是一个公司发布的标准
  TLS是国际组织发布的标准

#### 6.命名规范
* 私钥的后缀一般为：.key
* 公钥的后缀一般为：.pub
* 证书的后缀一般为：.crt
* ca证书的一般名为：ca.crt
* 请求的后缀一般为：.csr

***
### 应用

#### 1.https建立连接的过程
```mermaid
sequenceDiagram
client-->>server:请求建立连接
client-->server:协商使用的协议（SSLv3,TLS,...）
server-->>client:将公钥发送给客户端
client-->>server:随机生成对称密钥，通过公钥加密，发送给服务端
```
#### 2.ssh建立连接过程
```mermaid
sequenceDiagram
client-->>server:请求建立连接
server-->>client:server端发来公钥，client端确认是否接收
client-->>server:随机生成对称密钥，通过公钥加密，发送给服务端
client-->>server:利用对称密钥加密server端的用户名和密码，完成登录
server-->>client:利用对称密钥进行数据的加密
```
***
### openssl
**配置文件：/etc/pki/tls/openssl.cnf**
#### 1.openssl实现私有ca

（1）生成私钥
```shell
  (umask 077;openssl genrsa -out xx)     #括号里的命令是在子进程中执行的

#公钥是从私钥中提取出来的，因为ca需要的是自签署证书，所以这里不需要这样提取
#openssl rsa -in xx -pubout -out xx
```
（2）生成自签署证书（即ca，数字证书的格式为x509）
```shell
  openssl req -new -x509 -key xx -out xx -days 3650
```
#### 2.openssl利用已有ca，生成数字证书（即对其他公钥进行签名）

（1）生成证书签名请求（利用私钥生成该请求，因为公钥是从私钥中提取出来的）
```shell
  openssl req -new -key xx -out xx -subj '\CN=xx'

#CN很重要！！！！
#CN（Common Name）一定要和访问的域名设置成一样
```
（2）签署证书请求文件，生成数字证书
```shell
  openssl x509 -req -in xx \      #-req -in 后面跟请求文件
          -CA xx -CAkey xx \
          -CAcreateserial \       #当序列号文件不存在则自动创建，如果在openssl的配置的目录下找不到serial文件，该命令就会报错
          -days 3650 -out xx
```
#### 3.查看证书内容
```shell
  openssl x509 -in xx -text
```
***
### 认证机制

#### 1.NTLM认证（new technology lan manager，winodws操作系统用户登录）
密码经过hash后存储在：c:/windows/system32/config/SAM文件中
```mermaid
sequenceDiagram
client-->>server:发送用户名
server-->client:发送chanllenge（生成的随机数），server会根据用户名从SAM文件中取得密码的hash值，然后与chanllenge值计算，生成chanllenge1，保存在本地
client-->server:输入密码，然后利用该密码的hash值和chanllenge计算出reponse发送给server，与chanllenge1比较，如果相同，则认证成功
```
#### 2.域认证体系——kerbroes
（1）kerberos是一种网络认证协议，需要第三方服务
（2）域认证所参与的三个角色：
  * client
  * server
  * KDC（key distribution center，密钥分发中心，部署在域控制器上）
> KDC就是第三方服务，用于保证client和server端安全通信  
> 比证书认证更加复杂  