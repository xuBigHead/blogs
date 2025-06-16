---
layout: post
title: k8s-configmap-and-secret.md
categories: [cate1, cate2]
description: some word here
keywords: keyword1, keyword2
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---
# ConfigMap & Secret

ConfigMap扮演了K8S集群中配置中心的角色。ConfigMap定义了Pod的配置信息，可以以存储卷的形式挂载至Pod中的应用程序配置文件目录，从configmap中读取配置信息；也可以基于环境变量的形式，从ConfigMap中获取变量注入到Pod容器中使用。但是ConfigMap是明文保存的，如果用来保存数据库账号密码这样敏感信息，就非常不安全。一般这样的敏感信息配置是通过`secret`来保存。`secret`的功能和ConfigMap一样，不过secret是通过Base64的编码机制保存配置信息。

从ConfigMap中获取配置信息的方法有两种：

- 一种是利用环境变量将配置信息注入Pod容器中的方式，这种方式只在Pod创建的时候生效，这就意味着在ConfigMap中的修改配置信息后，更新的配置不能被已经创建Pod容器所应用。
- 另一种是将ConfigMap做为存储卷挂载至Pod容器内，这样在修改ConfigMap配置信息后，Pod容器中的配置也会随之更新，不过这个过程会有稍微的延迟。



ConfigMap当作存储卷挂载至Pod中的用法：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-configmap-vol-2
  labels:
    name: pod-configmap-vol-2
spec:
  containers:
  - name: myapp
    image: ikubernetes/myapp:v1
    volumeMounts:
    - name: my-cm-www
      # 将名为my-www的configmap挂载至Pod容器的这个目录下。
      mountPath: /etc/nginx/conf.d/       
  volumes:
  - name: my-cm-www
    configMap:               # 存储卷类型选configMap
```



secret的方法类似，只是secret对数据进行了加密。



## ConfigMap

在使用ConfigMap时需要注意以下几点：

1）ConfigMap必须在Pod之前创建，这样Pod才能使用它；

2）ConfigMap只能用于相同命名空间中的Pod；

3）ConfigMap无法用于静态Pod。



### Create ConfigMap

创建配置文件，文件内容如下：

```sh
$ cat cm-appvars.yaml
```



```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cm-appvars
data:
  var1: a
  var2: b
  var3: c
```



创建ConfigMap并获取其信息：

```sh
$ sudo kubectl create -f cm-appvars.yaml
configmap/cm-appvars created
$ sudo kubectl get configmap
NAME               DATA   AGE
cm-appvars         3      13s
kube-root-ca.crt   1      28d
$ sudo kubectl describe configmap cm-appvars
Name:         cm-appvars
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
var1:
----
a
var2:
----
b
var3:
----
c
Events:  <none>
```



### Use In Pod

#### 通过环境变量方式使用ConfigMap

我们通过Busybox进行实验，Pod定义如下：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["/bin/sh", "-c", "env | grep VAR"]
    env:
    - name: VAR_A
      valueFrom:
        configMapKeyRef:
          name: cm-appvars
          key: var1
    - name: VAR_B
      valueFrom:
        configMapKeyRef:
          name: cm-appvars
          key: var2
  restartPolicy: Never
```



我们busybox-pod在定义时通过valueFrom.configMapKeyRef为要注入的环境变量赋值，pod启动后通过env命令打印环境变量并退出。（这里设置restartPolicy: Never避免Pod执行完命令退出后再次重启）

我们创建Pod并通过kubectl logs命令进行验证：

```sh
$ sudo kubectl create -f busy_pod.yaml
pod/busybox-pod created
$ sudo kubectl logs busybox-pod
VAR_A=a
VAR_B=b
```



根据输出，我们可以得知容器内部的环境变量通过使用ConfigMap的值已经被正确配置。



#### 通过挂载Volume使用ConfigMap

还是以busybox为例，我们在Pod定义中，将cm-appvars中的内容以文件的形式挂载到容器内部的/vars目录下。修改后的Pod定义文件如下：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["/bin/sh", "-c", "top"]
    volumeMounts:
    - name: vars
      mountPath: /vars
  volumes:
  - name: vars
    configMap:
      name: cm-appvars
      items:
      - key: var1   # key=var1
        path: var1  # value将以var1文件名方式进行挂载
      - key: var2   # key=var2
        path: var2  # value将以var2文件名方式进行挂载
```



接着我们创建Pod并进入容器中查看是否有对应配置信息的文件：

```sh
$ sudo kubectl create -f busy_pod.yaml
pod/busybox-pod created
$ sudo kubectl exec -it busybox-pod  /bin/sh
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
/ # ls
bin   dev   etc   home  proc  root  sys   tmp   usr   var   vars

/ # cd vars
/vars # ls
var1  var2
/vars # cat var1
a/vars #
```



其实我们这里的例子很简单，在实际使用时，我们可以在ConfigMap中存储配置文件内容（如redis.conf、server.xml等），然后通过volume方式挂载到Pod中供应用加载。



## 向容器传递命令行参数

### 在Docker中定义命令与参数

待Docker中定义命令与参数，ENTRYPOINT定义容器启动时被调用的可以执行程序，CMD指定传递给ENTRYP的参数。

dockerfile 内容如下：

```dockerfile
FROM daocloud.io/centos:latest
 
ADD aaa /usr/local/aaa
 
CMD ["-f","/var/log/aa.log"]
ENTRYPOINT ["tail"]
```

当启动镜像时，容器启动时执行如下命令：tail -f /var/log/aa.log

或者在docker run <images> <arguments> 中指定，arguments会覆盖CMD中内容



### 在kubernetes中覆盖命令行和参数

在k8s中定义容器时，镜像的ENTRYPOINT和CMD都可以被覆盖，仅需在容器定义中设置熟悉command和args的值

对应参数如下：

| Docker     | kubernetes | 描述                   |
| ---------- | ---------- | ---------------------- |
| ENTRYPOINT | command    | 容器中运行的可执行文件 |
| CMD        | args       | 传给可执行文件的参数   |

相关yml代码如下：

```yaml
kind: pod
spec:
  containers:
  - image: some/image
    command: ["/bin/command"]
    args: ["args1","args2","args3"]
```



## 为容器设置环境变量

### 在容器定义中指定环境变量

与容器的命令和参数设置相同，环境变量列表无法在pod创建后被修改。在pod的yml文件中设置容器环境变量代码如下：

```yaml
kind: pod
spec:
  containers:
  - image: luksa/fortune:env
    env:
    - name: INTERVAL
      value: "30"
    name: value-test-yh
```



### 在环境变量值中引用其他环境变量

使用$( VAR )引用环境变量，相关ym代码如下：

```yaml
env:
- name: FIRST_VAR
  value: "foo"
- name: SECOND_VAR
  value: "$(FIRST_VAR)bar"   //最终变量值为foobar
```



## 利用ConfigMap解耦配置

kubernetes允许将配置选项分离到独立的资源对象ConfigMap中，本质上就是一个键/值对映射，值可以是短字面变量，也可以是完整的配置文件。

应用无须直接读取ConfigMap，甚至根本不需要知道其是否存在。

映射的内容通过环境变量或者卷文件的形式传递给容器，而并非直接传递给容器，命令行参数的定义中也是通过$(ENV_VAR)语法变量。



### 创建ConfigMap

**使用指令创建ConfigMap**

```
#kubectl creat configmap configmap-yaohong --from-literal=foo=bar --from-literal=sleep-interval=25
```



**从文件内容创建ConfigMap条目**

```
#kubectl create configmap my-conf-yh --from-file=config-file.conf
```

使用如下命令，会将文件内容存储在自定义的条目下。与字面量使用相同：

```
#kubectl create configmap my-conf-yh --from-file=customkey=config-file.conf
```



**从文件夹创建ConfigMap**

```
#kubectl create configmap my-conf-yh --from-file=/path/to/dir
```



**合并不同选项**

```
#kubectl create configmap my-conf-yh
  --from-file=/path/to/dir/
  --from-file=bar=foobar.conf
  --from-literal=some=thing
```



**获取ConfigMap**

```
#kubectl -n <namespace> get configmap
```



### 给容器传递ConfigMap条目作为环境变量

引用环境变量中的参数值给当前变量：

```yaml
apiVersion: v1
kind: pod
metadata:
  name: fortune-env-from-configmap
spec:
  containers:
  - image: luksa/fortune:env
    env:
    - name: INTERVAL                  //设置环境变量
      valueFrom:
        configMapkeyRef:
          name: fortune-configmap    
          key: sleep-interval         //变量的值取自fortune-configmap的slee-interval对应的值
```



### 一次性传递ConfigMap的所有条目作为环境变量

```yaml
apiVersion: v1
kind: pod
metadata:
  name: fortune-env-from-configmap
spec:
  containers:
  - image: luksa/fortune:env
    envFrom:
    - prefix: CONFIG_
      confgMapRef:
        name: my-confg-map    //引用my-config-map的ConfigMap并在变量前面都加上CONFIG_
```



### 使用ConfigMap卷将条目暴露为文件

```yaml
apiVersion: v1
kind: pod
metadata:
  name: configmap-volume-yh
spec:
  containers:
  - image: nginx:aplin
    name: web-server
    volumeMounts:
    ...
    - name: config<br>      defaultMode: "6600"     //设置文件的权限为rw-rw
      mountPath: /etc/nginx/con.conf<br>      subPath: my.conf        //subPath字段可以用于挂载卷中某个独立的文件或者文件夹，而且不覆盖该卷下其他文件
    ...
  volume:
  ...
  - name: config
    configMap:
      name: fortune-config     //引用fortune-config configMap的卷，然后挂载在/etc/nginx/conf.d
```



可以使用如下命令查看到/etc/nginx/conf.d文件下面包含fortune-config

```
#kubectl exec config-volume-yh -c web-server ls /etc/nginx/conf.d
```



## 使用Secert给容器传递敏感数据

Secret结构和ConfigMap类似，均是键/值对的映射。

使用方法也和ConfigMap一样，可以：

1.将Secret条目作为环境变量传递给容器，

2.将Secret条目暴露为卷中文件

ConfigMap存储非敏感的文本配置数据，采用Secret存储天生敏感的数据



### 默认令牌Secret

**查看secret**

```
# kubectl get secrets
NAME                  TYPE                                  DATA   AGE
default-token-x9cjb   kubernetes.io/service-account-token   3      78d
```



**描述secret**

```
# kubectl describe secrets default-token-x9cjb
Name:         default-token-x9cjb
Namespace:    default
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: default
              kubernetes.io/service-account.uid: 64a41a09-98ce-11e9-9fa5-fa163e6fdb6b
 
Type:  kubernetes.io/service-account-token
 
Data
====
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5lduaW8vc2VydmljZTxCv6HdtP-ZW3ZC2IKKR5YBhaokFIl35mix79pU4Ia2pJ_fuPTBGNyrCHyNQYH4ex5DhND3_b2puQmn8RSErQ
ca.crt:     1298 bytes
namespace:  7 bytes
```



### 创建Secret

**创建一个名为https-yh的generic secret**

```
#kubectl create secret generic https-yh --from-file=https.key  --from-file=https.cert  --from-file=foo
```



**创建一个secret.yaml文件，内容用base64编码**

```
$ echo -n 'admin' | base64
YWRtaW4=
$ echo -n '1f2d1e2e67df' | base64
MWYyZDFlMmU2N2Rm
```



yaml文件内容：

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  username: YWRtaW4=
  password: MWYyZDFlMmU2N2Rm
```



创建：

```
$ kubectl get secret mysecret -o yaml
apiVersion: v1
data:
  username: YWRtaW4=
  password: MWYyZDFlMmU2N2Rm
kind: Secret
metadata:
  creationTimestamp: 2016-01-22T18:41:56Z
  name: mysecret
  namespace: default
  resourceVersion: "164619"
  selfLink: /api/v1/namespaces/default/secrets/mysecret
  uid: cfee02d6-c137-11e5-8d73-42010af00002
type: Opaque
```



base64解码：

```
$ echo 'MWYyZDFlMmU2N2Rm' | base64 --decode
1f2d1e2e67df
```



### 对比ConfigMap与Secret

Secret的条目内容会进行Base64格式编码，而ConfigMap直接以纯文本展示。

**1.为二进制数据创建Secret**

Base64可以将二进制数据转换为纯文本，并以YAML或Json格式进行展示

但要注意Secret的大小限制是1MB

**2.stringDate字段介绍**

Secret可以通过StringDate字段设置条目的纯文本

```yaml
kind: Secret
apiVersion: v1
stringDate:
  foo: plain txt
date:
  https.cert: HGIOPUPSDF63456BJ3BBJL34563456BLKJBK634563456BLBKJBLKJ63456BLK3456LK
  http.key: OHOPGPIU42342345OIVBGOI3456345OVB6O3456BIPO435B6IPU345UI
```



### 在pod中使用Secret

secret可以作为数据卷挂载或者作为环境变量暴露给Pod中的容器使用，也可以被系统中的其他资源使用。比如可以用secret导入与外部系统交互需要的证书文件等。

在Pod中以文件的形式使用secret

1. 创建一个Secret，多个Pod可以引用同一个Secret
2. 修改Pod的定义，在`spec.volumes[]`加一个volume，给这个volume起个名字，`spec.volumes[].secret.secretName`记录的是要引用的Secret名字
3. 在每个需要使用Secret的容器中添加一项`spec.containers[].volumeMounts[]`，指定`spec.containers[].volumeMounts[].readOnly = true`，`spec.containers[].volumeMounts[].mountPath`要指向一个未被使用的系统路径。
4. 修改镜像或者命令行使系统可以找到上一步指定的路径。此时Secret中`data`字段的每一个key都是指定路径下面的一个文件名

下面是一个Pod中引用Secret的列子：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: mysecret
```



每一个被引用的Secret都要在`spec.volumes`中定义

如果Pod中的多个容器都要引用这个Secret那么每一个容器定义中都要指定自己的`volumeMounts`，但是Pod定义中声明一次`spec.volumes`就好了。

映射secret key到指定的路径

可以控制secret key被映射到容器内的路径，利用`spec.volumes[].secret.items`来修改被映射的具体路径

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: mysecret
      items:
      - key: username
        path: my-group/my-username
```



发生了什么呢？

- username被映射到了文件`/etc/foo/my-group/my-username`而不是`/etc/foo/username`
- password没有变

Secret文件权限

可以指定secret文件的权限，类似linux系统文件权限，如果不指定默认权限是`0644`，等同于linux文件的`-rw-r--r--`权限

设置默认权限位

```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
  volumes:
  - name: foo
    secret:
      secretName: mysecret
      defaultMode: 256
```



上述文件表示将secret挂载到容器的`/etc/foo`路径，每一个key衍生出的文件，权限位都将是`0400`

由于JSON不支持八进制数字，因此用十进制数256表示0400，如果用yaml格式的文件那么就很自然的使用八进制了

同理可以单独指定某个key的权限

```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
  volumes:
  - name: foo
    secret:
      secretName: mysecret
      items:
      - key: username
        path: my-group/my-username
        mode: 511
```



从volume中读取secret的值

值得注意的一点是，以文件的形式挂载到容器中的secret，他们的值已经是经过base64解码的了，可以直接读出来使用。

```
$ ls /etc/foo/
username
password
$ cat /etc/foo/username
admin
$ cat /etc/foo/password
1f2d1e2e67df
```



被挂载的secret内容自动更新

也就是如果修改一个Secret的内容，那么挂载了该Secret的容器中也将会取到更新后的值，但是这个时间间隔是由kubelet的同步时间决定的。最长的时间将是一个同步周期加上缓存生命周期(period+ttl)

> 特例：以[subPath](https://kubernetes.io/docs/concepts/storage/volumes/#using-subpath)形式挂载到容器中的secret将不会自动更新

以环境变量的形式使用Secret

1. 创建一个Secret，多个Pod可以引用同一个Secret
2. 修改pod的定义，定义环境变量并使用`env[].valueFrom.secretKeyRef`指定secret和相应的key
3. 修改镜像或命令行，让它们可以读到环境变量

```
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
spec:
  containers:
  - name: mycontainer
    image: redis
    env:
      - name: SECRET_USERNAME
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: username
      - name: SECRET_PASSWORD
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: password
  restartPolicy: Never
```



容器中读取环境变量，已经是base64解码后的值了：

```
$ echo $SECRET_USERNAME
admin
$ echo $SECRET_PASSWORD
1f2d1e2e67df
```



使用imagePullSecrets

创建一个专门用来访问镜像仓库的secret，当创建Pod的时候由kubelet访问镜像仓库并拉取镜像，具体描述文档在 [这里](https://kubernetes.io/docs/concepts/containers/images/#specifying-imagepullsecrets-on-a-pod)

设置自动导入的imagePullSecrets

可以手动创建一个，然后在serviceAccount中引用它。所有经过这个serviceAccount创建的Pod都会默认使用关联的imagePullSecrets来拉取镜像，