# coreDNS
[toc]

### 概述

#### 1.容器如何使用DNS
* 启动coreDNS，会分配一个静态ip
```shell
#通过该命令可以查看到coreDNS的静态ip
kubectl get svc -n kube-system
```
* 在kubelet的启动参数中指定，使用的DNS
通过kubelet启动容器，会自动注入DNS服务器信息到容器中
```shell
--cluster-dns=<dns-service-ip>
--cluster-domain=<default-local-domain>
```

***

### 配置

#### 1.配置文件：Corefile
每一组配置都是一个插件（比如errors是一个插件，health也是一个插件）
```shell
# . 表示根域，即所有匹配所有的域
# 53 表示监听在53端口上
.:53 {

    #将错误输出到标准输出
    errors

    #将coreDNS状态报告到 http://localhost:8080/health
    health

    #在k8s中的设置
    #是 cluster.local in-addr.arpa ip6.arpa 这些域的权威域
    kubernetes cluster.local in-addr.arpa ip6.arpa {
       pods insecure
       upstream
       fallthrough in-addr.arpa ip6.arpa
       ttl 30
    }


    #hosts后面加域，如果域为空，表示匹配配置文件上面设置的域，即所有域
    hosts {
      
        #添加域名解析
        172.28.202.162 gitlab.devops.nari

        #如果域匹配了，但是没有任何结果，则将请求出入下一个插件
        fallthrough
    }

    #暴露0.0.0.0:9153端口，提供prometheus的metric接口
    prometheus :9153

    #转发不能解析的域名
    #. 表示根域，即所有匹配所有的域
    #转发到/etc/resolve.conf中设置的DNS服务器，也可以直接写DNS服务器地址，比如：forward . 114.114.114.114 8.8.8.8
    forward . /etc/resolv.conf
}
```

#### 2.修改配置
```shell
kubectl edit cm coredns -n kube-system
```
