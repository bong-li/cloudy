[toc]
# authentication（认证）
### 概述
#### 1.认证
访问k8s集群需要先经过apiServer的认证
因此需要一个 账号 和 密码（即令牌）

有两类账号：
* UserAccount，用户访问使用的账号
* ServiceAccount，内部服务访问使用的账号

#### 2.特点
* 每个namespace中都有一个默认的Secret资源，存储的是token，用于认证
* 该名称空间内的pod都会挂载该Secret，从而能够通过apiServer的认证
* ServiceAccount可以绑定docker-registry类型的secrets
从而可以指定使用其他仓库

### 使用
#### 1.创建serviceaccount
```shell
kubectl create serviceaccount 账号名

#创建ServiceAccount资源后，会自动生成一个用于 认证 的Secret，并与该ServiceAccount绑定
```
#### 2.指定pod使用哪个账号
```shell
spec.ServiceAcountName

#pod使用指定ServiceAccount，会挂载相应的Secret
```

***

# authorization（授权）
### 概述
#### 1.常用的授权插件：
* Node
* ABAC：attribute-based access control
* RBAC：role-based access control（k8s采用的是RBAC）
* Webhook

#### 2.k8s上角色有两个级别
* 集群级别
* 命名空间级别

#### 3.相关资源
##### （1）Role（命名空间级别资源）				
属于某个命名空间的role
##### （2）ClusterRole（集群级别资源）		
属于集群的role
##### （3）RoleBinding（命名空间级别资源）
用于**在某个namespace中**，将**此namesoace中**的某个role（包括ClusterRole）与指定namespace中的某个serviceaccount绑定
##### （4）ClusterRoleBinding（集群级别资源）
用于**在集群级别**，将某个ClusterRole与某个serviceaccount绑定

##### （5）最常用的方式
* 创建一个ClusterRole，然后使用RoleBinding 在 某个命名空间中 将 角色 与 某个账号 绑定
* 好处是，不需要在每个命名空间中再额外创建role

#### 4.verbs
* create
* get（用于单个资源）
* list（用于集合）
* watch
* update
* patch
* delete（用于单个资源）
* deletecollection（用于集合）

### 使用

#### 1.定义角色
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:           #ClusterRole不需要指定namespace
  name: xx		

#配置该role的权限，即对什么资源有什么权限
#如何配置可以通过此命令查看：kubectl get api-resources -o wide
#设置全部的话就设为："*"
rules:
- apiGroups:
  - xx            #指定api群组，如果是核心组，就填：""
                  #设置全部的话就设为："*"
  resources:      #指定资源类型
  - xx            #比如：pods，设置全部的话就设为："*"

  verbs:          #指定允许的操作
  - xx            #比如：get,list,watch
```

#### 2.将角色与账号（包括UserAccount和ServieAccount）绑定
* 通过ClusterRoleBinding绑定
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: zdgt

roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: zdgt

subjects:
- kind: ServiceAccount
  name: zdgt
  namespace: default
```

* 通过RoleBinding绑定
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: xx
  namespace: xx       #指定在哪个命名空间中绑定
                      #即绑定的账号在此命名空间中才有此role的权限
roleRef:              #指定需要绑定的role
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: xx


#账号需要与RoleBinding在同一个命名空间
subjects:             #指定需要绑定的账号
- apiGroup: ""        #当kind为ServiceAccount时，apiGroup为：""，即核心组
                      #当kind为User或Group时，apiGroup为：rbac.authorization.k8s.io
  kind: ServiceAccount        #还可以填User和Group
  name: xx
```
***
# admissionController（准入控制）

***

# demo
### 使得某个Pod有权限管理整个k8s集群

比如：使用某个有权限的serviceaccount的token即可登录到dashboard
      （启动dashboard这个pod时不必指定serviceaccount）

#### 1.首先创建一个serviceaccount
```shell
kubectl create serviceaccount xx -n xx
```

#### 2.绑定cluster-admin这个角色（这个角色是创建集群时生成的，所以不需要额外创建角色了）
```shell
kubectl create clusterrolebinding xx \
	  --clusterrole=cluster-admin \
	  --serviceaccount=<NAMESPACE>:<SERVICEACCOUNT>
```
#### 3.获取该serviceaccount的secret，从而获取token
利用该token即可登录dashboard
