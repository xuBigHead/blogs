---
layout: post
title: Docker Compose.md
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
# Docker Compose

Docker Compose帮助我们批量有规则的管理容器，可以实现批量启动服务并管理服务间的依赖关系。

Docker Compose虽然是官方提供的容器编排工具，但是实际生产环境是不用的（用Swarm、K8S等），因为其局限性很大。Docker Compose只支持单机多容器，不支持集群环境管理。



根据官方提示，使用 Docker Compose 分为三个步骤：

- 第一步：使用 Dockerfile 定义应用程序的环境。
- 第二步：使用 docker-compose.yml 定义构成应用程序的服务，这样它们可以在隔离环境中一起运行。
- 第三步：执行 docker-compose up 命令来启动并运行整个应用程序。



## Configuration

Docker Compose的核心就是docker-compose.yml文件的编写。规则如下：

```yaml
# 第一层：版本
version: "3.9" 
# 第二层：服务    
services:
	# 服务名称
  db:
  	# 镜像名称
    image: mysql:5.7
    # 挂载的容器卷
    volumes:
      - db_data:/var/lib/mysql
    # 服务挂掉是否自动重启
    restart: always
    # 环境变量设置
    environment:
      MYSQL_ROOT_PASSWORD: somewordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
  # 服务名称  
  wordpress:
  	# 依赖的服务
    depends_on:
      - db
    image: wordpress:latest
    volumes:
      - wordpress_data:/var/www/html
    ports:
      - "8000:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
# 第三层：其他配置，包括网络，容器卷等等
volumes:
  db_data: {}
  wordpress_data: {}
```



## Application Example

### Deploy WordPress

> WordPress是一款个人博客系统，并逐步演化成一款内容管理系统软件，它是使用PHP语言和MySQL数据库开发的，用户可以在支持 PHP 和 MySQL数据库的服务器上使用自己的博客。



- 创建项目目录

名称任意，用来存放 docker-compose.yml 文件，按照官方创建一个名为 my_wordpress 目录。

```bash
$ mkdir my_wordpress
```



- 创建 docker-compose.yml

新建一个 docker-compose.yml 文件，内容如下：

```yaml
version: "3.9"
    
services:
  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: somewordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
    
  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    volumes:
      - wordpress_data:/var/www/html
    ports:
      - "8000:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
volumes:
  db_data: {}
  wordpress_data: {}
```



- 构建项目

注意要切换到my_wordpress 目录，否则要增加 -f 参数指定 docker-compose.yml 文件。

```bash
$ docker-compose up -d
```



- 访问页面

通过`http://宿主机IP:8000`来访问页面。



### Custom Application

自定义应用：每次访问 Tomcat的服务，Redis计数器加1。



- 编写 Tomcat服务

新建一个springboot项目，然后新建一个controller类：

```java
@RestController
public class CounterController {
    @Autowired
    StringRedisTemplate redisTemplate;
    @GetMapping("/visit")
    public String count(HttpServletRequest request){
        String remoteHost = request.getRemoteHost();
        Long increment = redisTemplate.opsForValue().increment(remoteHost);
        return remoteHost +"访问次数"+increment.toString();
    }
}
```



Springboot 服务的配置文件 application.yml：

```yaml
server:
  port: 8080
  servlet:
    context-path: /counter
spring:
  redis:
    host: counterRedis
```



- Dockerfile

```dockerfile
FROM openjdk:8-jdk
 
COPY *.jar /counter.jar
 
CMD ["--server.port=8080"]
 
EXPOSE 8080
 
ENTRYPOINT ["java","-jar","/counter.jar"]
```



- docker-compose.yml

```yaml
version: "3.8"
services:
  itcokecounter:
    build: .
    image: itcokecounter
    depends_on:
      - counterRedis
    ports:
      - "8080:8080"
  counterRedis:
    image: "redis:6.0-alpine"
```



- 测试

在Linux服务器新建 counter 文件夹，把下面三个文件拷贝到其中。

```bash
$ ls -l
counter-0.0.1-SNAPSHOT.jar
docker-compose.yml
Dockerfile
```



然后执行如下命令构建：

```bash
$ docker-compose up
```



SpringBoot服务启动后，在浏览器中输入地址 `http://{ip}:8080/counter/visit` 进行访问。