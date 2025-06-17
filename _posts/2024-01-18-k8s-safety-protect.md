---
layout: post
title: k8s-safety-protect.md
categories: [Kubernetes 容器编排]
description: Kubernetes 容器编排
keywords: Kubernetes 容器编排
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---
# Safety Protect

## API服务器的安全防护

### 认证机制

启动API服务器时，通过命令行选项可以开启认证插件。



**了解用户：**

分为两种连接到api服务器的客户端：

1.真实的人

2.pod，使用一种称为ServiceAccount的机制

**了解组：**

认证插件会连同用户名，和用户id返回组，组可以一次性给用户服务多个权限，不用单次赋予，

system:unauthenticated组：用于所有认证插件都不会认证客户端身份的请求。

system:authenticated组：会自动分配给一个成功通过认证的用户。

system:serviceaccount组：包含 所有在系统中的serviceaccount。

system:serviceaccount：<namespace>组：包含了所有在特定命名空间中的serviceAccount。



#### ServiceAccount

每个pod中都包含/var/run/secrets/kubernetes.io/serviceaccount/token文件，如下图所示，文件内容用于对身份进行验证，token文件持有serviceaccount的认证token。

![img](../Image/2022/221206-47.jpg)

应用程序使用token去连接api服务器时，认证插件会对serviceaccount进行身份认证，并将serviceaccount的用户名传回到api服务器内部。

​    serviceaccount的用户名格式如下：

system:serviceaccount:<namespace>:<service account name>

ServiceAccount是运行在pod中的应用程序，和api服务器身份认证的一中方式。

**了解ServiceAccount资源**

ServiceAcount作用在单一命名空间，为每个命名空间创建默认的ServiceAccount。

​    ![img](../Image/2022/221206-48.jpg)

多个pod可以使用相同命名空间下的同一的ServiceAccount，

**ServiceAccount如何与授权文件绑定**

在pod的manifest文件中，可以指定账户名称的方式，将一个serviceAccount赋值给一个pod，如果不指定，将使用该命名空间下默认的ServiceAccount.

可以 将不同的ServiceAccount赋值给pod，让pod访问不同的资源。



#### 创建ServiceAccount

为了集群的安全性，可以手动创建ServiceAccount，可以限制只有允许的pod访问对应的资源。

​    创建方法如下：

```
$ kubectl get sa
NAME      SECRETS   AGE
default   1         21h
 
 
$ kubectl create serviceaccount yaohong
serviceaccount/yaohong created
 
 
$ kubectl get sa
NAME      SECRETS   AGE
default   1         21h
yaohong   1         3s
```



使用describe来查看ServiceAccount。

```
$ kubectl describe sa yaohong
Name:                yaohong
Namespace:           default
Labels:              <none>
Annotations:         <none>
Image pull secrets:  <none>
Mountable secrets:   yaohong-token-qhbxn   //如果强制使用可挂载秘钥。那么使用这个serviceaccount的pod只能挂载这个秘钥
Tokens:              yaohong-token-qhbxn
Events:              <none>
```



查看该token，

```
$ kubectl describe secret yaohong-token-qhbxn
Name:         yaohong-token-qhbxn
Namespace:    default
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: yaohong
              kubernetes.io/service-account.uid: a3d0d2fe-bb43-11e9-ac1e-005056870b4d
 
Type:  kubernetes.io/service-account-token
 
Data
====
ca.crt:     1342 bytes
namespace:  7 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6Inlhb2hvbmctdG9rZW4tcWhieG4iLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoieWFvaG9uZyIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6ImEzZDBkMmZlLWJiNDMtMTFlOS1hYzFlLTAwNTA1Njg3MGI0ZCIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0Onlhb2hvbmcifQ.BwmbZKoM95hTr39BuZhinRT_vHF-typH4anjkL0HQxdVZEt_eie5TjUECV9UbLRRYIqYamkSxmyYapV150AQh-PvdcLYPmwKQLJDe1-7VC4mO2IuVdMCI_BnZFQBJobRK9EdPdbZ9uxc9l0RL5I5WyWoIjiwbrQvtCUEIkjT_99_NngdrIr7QD9S5SxHurgE3HQbmzC6ItU911LjmxtSvBqS5NApJoJaztDv0cHKvlT67ZZbverJaStQdxr4yiRbpSycRNArHy-UZKbNQXuzaZczSjVouo5A5hzgSHEBBJkQpQ6Tb-Ko5XGjjCgV_b9uQvhmgdPAus8GdFTTFAbCBw
```



#### **将ServiceAccount分配给pod**

**在pod中定义的spec.serviceAccountName字段上设置，此字段必须在pod创建时设置后续不能被修改。**

**自定义pod的ServiceAccount的方法如下图**

![img](../Image/2022/221206-50.jpg)



### 通过基于角色的权限控制加强集群安全

RBAC授权插件将用户角色作为决定用户能否执行操作的关机因素。



RBAC授权规则通过四种资源来进行配置的，他们可以分为两组：

Role和ClusterRole，他们决定资源上可执行哪些动词。

RoleBinding和ClusterRoleBinding，他们将上述角色绑定到特定的用户，组或者ServiceAccounts上。

Role和RoleBinding是namespace级别资源

ClusterRole和ClusterRoleBinding是集群级别资源



#### **使用Role和RoleBinding**

Role资源定义了哪些操作可以在哪些资源上执行，



**创建Role**

service-reader.yml

```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: kube-system
  name: service-reader
rules:
- apiGroups: [""]
  verbs: ["get", "list"]
  resources: ["services"]
```



在kube-system中创建Role

```
#kubectl -n kube-system create -f service-reader.yml
```



查看该namespace下的role

```
$ kubectl -n kube-system get role
NAME                                             AGE
extension-apiserver-authentication-reader        41h
kube-state-metrics-resizer                       41h
service-reader                                   2m17s
system::leader-locking-kube-controller-manager   41h
system::leader-locking-kube-scheduler            41h
system:controller:bootstrap-signer               41h
system:controller:cloud-provider                 41h
system:controller:token-cleaner                  41h
```



**绑定角色到ServiceAccount**

将service-reader角色绑定到default ServiceAccount

```
$ kubectl  create rolebinding test --role=service-reader
rolebinding.rbac.authorization.k8s.io/test created
```



![img](../Image/2022/221206-51.jpg)

```
$ kubectl  get rolebinding test -o yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  creationTimestamp: 2019-08-11T03:40:51Z
  name: test
  namespace: default
  resourceVersion: "239323"
  selfLink: /apis/rbac.authorization.k8s.io/v1/namespaces/default/rolebindings/test
  uid: d0aff243-bbe9-11e9-ac1e-005056870b4d
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: service-reader
```



#### 使用ClusterRole和ClusterRoleBinding

**查看集群ClusterRole**

```
# kubectl get clusterrole
NAME                                                                   AGE
admin                                                                  42h
cluster-admin                                                          42h
edit                                                                   42h
flannel                                                                42h
kube-state-metrics                                                     42h
system:aggregate-to-admin                                              42h
...
```



**创建ClusterRole**

```
	
kubectl create clusterrole flannel --verb=get,list -n kube-system　
```



查看yaml文件

```
# kubectl get  clusterrole flannel -o yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"rbac.authorization.k8s.io/v1","kind":"ClusterRole","metadata":{"annotations":{},"name":"flannel"},"rules":[{"apiGroups":[""],"resources":["pods"],"verbs":["get"]},{"apiGroups":[""],"resources":["nodes"],"verbs":["list","watch"]},{"apiGroups":[""],"resources":["nodes/status"],"verbs":["patch"]}]}
  creationTimestamp: 2019-08-09T09:58:42Z
  name: flannel
  resourceVersion: "360"
  selfLink: /apis/rbac.authorization.k8s.io/v1/clusterroles/flannel
  uid: 45100f6f-ba8c-11e9-8f57-005056870608
rules:
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - get
- apiGroups:
  - ""
  resources:
  - nodes
  verbs:
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - nodes/status
  verbs:
  - patch
```



**创建clusterRoleBinding**

```
$ kubectl  create clusterrolebinding  cluster-tetst  --clusterrole=pv-reader  --serviceaccount=kuebsystem:yaohong
clusterrolebinding.rbac.authorization.k8s.io/cluster-tetst created
```



![img](../Image/2022/221206-52.jpg)



##### 默认的ClusterRole和ClusterRoleBinding

如下所示使用kubectl get clusterroles和kubectl get clusterrolesbinding可以获取k8s默认资源。

用edit ClusterRole允许对资源进行修改

用admin ClusterRole赋予一个命名空间全部的权限

```
$ kubectl get clusterroles
NAME                                                                   AGE
admin                                                                  44h
cluster-admin                                                          44h
edit                                                                   44h
flannel                                                                44h
kube-state-metrics                                                     44h
system:aggregate-to-admin                                              44h
system:aggregate-to-edit                                               44h
system:aggregate-to-view                                               44h
system:auth-delegator                                                  44h
system:aws-cloud-provider                                              44h
system:basic-user                                                      44h
system:certificates.k8s.io:certificatesigningrequests:nodeclient       44h
system:certificates.k8s.io:certificatesigningrequests:selfnodeclient   44h
system:controller:attachdetach-controller                              44h
system:controller:certificate-controller                               44h
system:controller:clusterrole-aggregation-controller                   44h
。。。
```



```
$ kubectl get clusterrolebindings
NAME                                                   AGE
clust-tetst                                            17m
cluster-admin                                          44h
cluster-tetst                                          13m
flannel                                                44h
kube-state-metrics                                     44h
kubelet-bootstrap                                      44h
system:aws-cloud-provider                              44h
system:basic-user                                      44h
system:controller:attachdetach-controller              44h
system:controller:certificate-controller               44h
。。。
```



## 保障集群内节点和网络安全

### 在pod中使用宿主节点的Linux命名空间

#### 在pod中使用宿主节点的网络命名空间

在pod的yaml文件中就设置spec.hostNetwork: true

这个时候pod使用宿主机的网络，如果设置了端口，则使用宿主机的端口。



```
apiVersion: v1
kind: pod
metadata:
    name: pod-host-yaohong
spec:
    hostNetwork: true  //使用宿主节点的网络命名空间
    containers:
    - image: luksa/kubia
      command: ["/bin/sleep", "9999"]
```





#### 绑定宿主节点上的端口而不使用宿主节点的网络命名空间

在pod的yaml文件中就设置spec.containers.ports字段来设置

在ports字段中可以使用

containerPorts设置通过pod 的ip访问的端口

container.hostPort设置通过所在节点的端口访问



```
apiVersion: v1
kind: pod
metadata:
    name: kubia-hostport-yaohong
spec:
    containers:
    - image: luksa/kubia
    - name: kubia
      ports:
      - containerport: 8080 //该容器通过pod IP访问该端口
        hostport: 9000  //该容器可以通过它所在节点9000端口访问
        protocol: Tcp
```



#### 使用宿主节点的PID与IPC

PID是[进程ID](https://www.baidu.com/s?wd=进程ID&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)，PPID是父[进程ID](https://www.baidu.com/s?wd=进程ID&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)。

在linux下的多个进程间的通信机制叫做IPC(Inter-Process Communication)，它是多个进程之间相互沟通的一种方法。

```
apiVersion: v1
kind: pod
metadata:
    name: pod-with-host-pid-and-ipc-yaohong
spec:
    hostPID: true //你希望这个pod使用宿主节点的PID命名空间
    hostIPC: true //你希望pod使用宿主节点的IPC命名空间
    containers：
    - name: main
      image: alpine
      command: ["/bin/sleep", "99999"]
```



### 配置节点的安全上下文

#### 使用指定用户运行容器

查看某个pod运行的用户

```
$ kubectl -n kube-system exec coredns-7b8dbb87dd-6ll7z id
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video)
```



容器的运行用户再DockerFile中指定，如果没有指定则为root

指定pod的运行的用户方法如下

```
apiVersion: v1
kind: pod
metadata:
    name: pod-as-user
spec:
    containers：
    - name: main
      image: alpine
      command: ["/bin/sleep", "99999"]
      securityContext:
        runAsUser: 405   //你需要指定的用户ID,而不是用户名
```



#### 阻止容器以root用户运行

runAsNonRoot来设置

```
apiVersion: v1
kind: pod
metadata:
    name: pod-as-user
spec:
    containers：
    - name: main
      image: alpine
      command: ["/bin/sleep", "99999"]
      securityContext:
        runAsNonRoot: true   //这个容器只允许以非root用户运行
```



#### 使用特权模式运行pod

为了获得宿主机内核完整的权限，该pod需要在特权模式下运行。需要添加privileged参数为true。

```
apiVersion: v1
kind: pod
metadata:
    name: pod-as-privileged
spec:
    containers：
    - name: main
      image: alpine
      command: ["/bin/sleep", "99999"]
      securityContext:
        privileged: true   //这个容器将在特权模式下运行
```



#### 为容器单独添加内核功能

```
apiVersion: v1
kind: pod
metadata:
    name: pod-as-capability
spec:
    containers：
    - name: main
      image: alpine
      command: ["/bin/sleep", "99999"]
      securityContext:
        capabilities:    //该参数用于pod添加或者禁用某项内核功能
          add:
          - SYS_TIME      //添加修改系统时间参数
```



#### 在容器中禁止使用内核功能

```
apiVersion: v1
kind: pod
metadata:
    name: pod-as-capability
spec:
    containers：
    - name: main
      image: alpine
      command: ["/bin/sleep", "99999"]
      securityContext:
        capabilities:    //该参数用于pod添加或者禁用某项内核功能
          drop:
          - CHOWN      //禁用容器修改文件的所有者
```



#### 阻止对容器根文件系统的写入

securityContext.readyOnlyFilesystem设置为true来实现阻止对容器根文件系统的写入。

```
apiVersion: v1
kind: pod
metadata:
    name: pod-with-readonly-filesystem
spec:
    containers：
    - name: main
      image: alpine
      command: ["/bin/sleep", "99999"]
      securityContext:
         readyOnlyFilesystem: true  //这个容器的根文件系统不允许写入
      volumeMounts:
      - name: my-volume
        mountPath: /volume    //volume写入是允许的，因为这个目录挂载一个存储卷
        readOnly: false 
```



### 限制pod使用安全相关的特性

PodSecurityPolicy是一种集群级别（无命名空间）的资源，它定义了用户能否在pod中使用各种安全相关的特性。



#### runAsUser、fsGroups和supplementalGroup策略

```yaml
runAsUser:
  runle: MustRunAs
  ranges:
  - min: 2             //添加一个max=min的range，来指定一个ID为2的user
    max: 2
  fsGroup:
    rule: MustRunAs
    ranges:
    - min: 2
      max: 10         //添加多个区间id的限制，为2-10 或者20-30
    - min: 20
      max: 30
  supplementalGroups:
    rule: MustRunAs
    ranges:
    - min: 2
      max: 10
    - min: 20
      max: 30
```



#### 配置允许、默认添加、禁止使用的内核功能

三个字段会影响容器的使用

allowedCapabilities:指定容器可以添加的内核功能

defaultAddCapabilities:为所有容器添加的内核功能

requiredDropCapabilities:禁止容器中的内核功能

```
apiVersion: v1
kind: PodSecurityPolicy
spec:
  allowedCapabilities:
  - SYS_TIME                 //允许容器添加SYS_time功能
  defaultAddCapabilities:
  - CHOWN                    //为每个容器自动添加CHOWN功能
  requiredDropCapabilities:
  - SYS_ADMIN                //要求容器禁用SYS_ADMIN和SYS_MODULE功能
```



### 隔离pod网络

#### 在一个命名空间中使用网络隔离

podSelector进行对一个命名空间下的pod进行隔离

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: postgres-netpolicy
spec:
  podSelector:         //这个策略确保了对具有app=databases标签的pod的访问安全性
    matchLabels:
      app: database
  ingress:
  - from:
    - podSelector:    //它只允许来自具有app=webserver标签的pod的访问
      matchLabels:
        app: webserver
    ports:
    - port: 5432      //允许对这个端口的访问
```



#### 在 不同的kubernetes命名空间之间进行网络隔离

namespaceSelector进行对不同命名空间间进行网络隔离

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: postgres-netpolicy
spec:
  podSelector:         //这个策略确保了对具有app=databases标签的pod的访问安全性
    matchLabels:
      app: database
  ingress:
  - from:
    - namespaceSelector:    //只允许tenant: manning标签的命名空间中运行的pod进行互相访问
      matchLabels:
        tenant: manning  
    ports:
    - port: 5432      //允许对这个端口的访问
```



#### 使用CIDR网络隔离

```yaml
ingress:
- from:
  - ipBlock:
      cidr: 192.168.1.0/24    //指明允许访问的ip段
```



#### 限制pod对外访问流量

使用egress进行限制

```yaml
spec:
  podSelector:         //这个策略确保了对具有app=databases标签的pod的访问安全性
    matchLabels:
      app: database
  egress:              //限制pod的出网流量
  - to:
    - podSelector:
        matchLables:   //database的pod只能与有app: webserver的pod进行通信
          app: webserver
```