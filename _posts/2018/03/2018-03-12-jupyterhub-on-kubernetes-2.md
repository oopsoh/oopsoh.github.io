---
layout: post
title: jupyterhub on kubernetes（2）
date: 2018-03-12 13:30
description: 解析jupyterhub yaml文件，集成openldap认证
img: jupyterhub.png
tags: [jupyterhub, openldap, docker]
permalink: /2018/03/:title/

---


&emsp;&emsp;在上一篇[kubernetes on jupyterhub (1)](https://oopsoh.github.io/2018/03/jupyterhub-on-kubernetes-1/)中获取到了yaml文件：

- Chart.yaml  
   存放的jupyterhub在helm Chart中的元数据信息。
- schema.yaml  
   定义的value.yaml文件中的key-value的元数据信息。
- templates  
   模版文件包括hub和proxy的模版文件，也就是符合kubernetes的yaml文件。
- validate.py  
   验证脚本，验证配置的values.yaml是否符合schema.yaml中的定义和规范。
- values.yaml  
   参数定义的文件，模版中的变量都是从这里获取，主要就是自定义配置config.yaml覆盖这个文件。

#### 集成openldap用户认证

&emsp;&emsp;[参考文档](https://github.com/jupyterhub/jupyterhub)中的authenticators提供了4种类型，其中默认的就是操作系统的PAM认证，选择[ldapauthenticator](https://github.com/jupyterhub/ldapauthenticator)，查看具体的配置过程，要使用pip安装jupyterhub-ldapauthenticator，这里我们使用的jupyterhub是docker镜像，默认的docker镜像中没有安装，所以要重新编写Dockerfile文件，然后build镜像，push到自己的私有镜像仓库。

```
FROM jupyterhub/k8s-hub:0b2f5d9
RUN pip3 install --no-cache-dir \
         wheel \
         jupyterhub-ldapauthenticator
```
&emsp;&emsp;上面的jupyterhub/k8s-hub:0b2f5d9这个镜像可以在values.yaml中找到，这个就是默认的hub镜像，在它的基础上安装jupyterhub-ldapauthenticator，可以创建一个文件夹，把Dockerfile放到里面，然后build，这里我先假设自己搭建的私有镜像仓库地址是mydocker.registry.com，后期专门写一个搭建自己私有镜像仓库的文章。

``` shell
mkdir hubldapimage
mv Dockerfile ./hubldapimage/
cd hubldapimage
docker build -t mydocker.registry.com/jupyterhub/k8s-hub:v1
docker push mydocker.registry.com/jupyterhub/k8s-hub:v1
```
&emsp;&emsp;上述命令中镜像仓库的地址可以换成你自己的，后面的路径和镜像的名称都可以自定义，v1是我自定义的tag，你也可以换成你自己的，有版本区别就行。接下来把yaml文件中的image替换成上面build的镜像，可以修改values.yaml文件，也可以创建一个config.yaml文件，在helm install的时候 `-f`指定这个config.yaml文件，会自动覆盖掉values.yaml文件中的默认值。这里我和jupyterhub文件夹同级别创建config.yaml文件，里面写入以下内容。

```yaml
hub:
  image:
    name: mydocker.registry.com/jupyterhub/k8s-hub
    tag: v1

```
&emsp;&emsp;上面就已经准备好了集成了ldap模块的hub镜像，还少一步hub连接ldap的配置，在裸机安装hub的时候，hub的配置是jupyterhub_config.py，这里在config.yaml中通过下面的配置自动添加条目到jupyterhub_config.py。

```yaml
hub:
  extraConfig: |
    c.JupyterHub.authenticator_class = 'ldapauthenticator.LDAPAuthenticator'
    c.LDAPAuthenticator.server_address = 'localhost'
    c.LDAPAuthenticator.bind_dn_template = 'uid={username},ou=example,dc=com,dc=cn'
    c.LDAPAuthenticator.server_port = 389
    c.LDAPAuthenticator.user_attribute = 'uid'
    c.LDAPAuthenticator.lookup_dn = True
    c.LDAPAuthenticator.user_search_base = 'ou=example,dc=com,dc=cn'
    c.LDAPAuthenticator.use_ssl = False
```
#### 后端存储类型
&emsp;&emsp;jupyterhub的后端存储类型默认有下列四种类型：

- sqlite-pvc
- sqlite-memory
- mysql
- postgres

&emsp;&emsp;这里我选择使用的是mysql，首先要在数据库中创建一个jupyterhub数据库，赋予相应的访问和操作权限。

```shell
create database jupyterhub;
grant all privileges on jupyterhub.* to 'jupyterhub'@'%' IDENTIFIED BY 'jupyterhub';
flush privileges;
```
&emsp;&emsp;接下来配置config.yaml，这里如果选用mysql或者postgres，则必须同时设置`hub.cookieSecret`，这个是一个64字节的加密字符串用来加密hub的cookie信息，如果使用的是外部数据库，必须显示的设置此值，更改这个值，所有的登陆用户都将无效，使用命令`openssl rand -hex 32`生成随机字符串；这里还有一个`proxy.secretToken`值，用来加密hub和hub的代理`configurable-http-proxy`之间的通信。

- mysql+pymysql://\<db-username\>:\<db-password\>@\<db-hostname\>:\<db-port\>/\<db-name\>
- postgres+psycopg2://\<db-username\>:\<db-password\>@\<db-hostname\>:\<db-port\>/\<db-name\>


```yaml
proxy:
  secretToken: 42313dc5ae5be7087958cf669039daaa047ea86c66211940649ef4cb513e8543
hub:
  db:
    type: mysql
    url: mysql+pymysql://jupyterhub:jupyterhub@localhost:3306/jupyterhub
  cookieSecret: ef7610809b865330f9f57131b1d84e5e50eb67b53ab648c67a348c4c16dd61eb
```


#### 使用Ingress暴露服务
&emsp;&emsp;在jupyterhub的template文件夹中，proxy文件夹是nginx做代理使用的部署yaml文件，其中service里面80和443端口是要暴露出来的，下面截图就是模版文件。

![jupyterhub-proxy-service-template.jpg](https://s1.ax1x.com/2018/03/15/94RKYT.jpg)

&emsp;&emsp;上述yaml文件中，ports里面有80和443端口，暴露的类型取决于`type`的值，values.yaml文件中proxy.service.type的值，默认是loadBalance，这样的话，80和443端口要暴露到本机，会和kubernetes的Ingress Controller traefik以daemonset部署暴露的80和443端口冲突，所以这里要修改设置proxy的service里面的type以clusterIP的类型，然后再配置Ingress去暴露proxy的clusterIP。

```yaml
proxy:
  service:
    type: ClusterIP
```
&emsp;&emsp;接下来创建jupyterhub-ingress-ui.yaml文件，配置Ingress暴露端口。

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: jupyterhub-ingress-ui
  namespace: kube-public
  annotations:
    kubernetes.io/ingress.class: traefik
spec:
  rules:
  - host: jupyterhub.example.com.cn
    http:
      paths:
        - path: /
          backend:
            serviceName: proxy-public
            servicePort: 80
```
#### 设置hub管理员
&emsp;&emsp;设置打开admin账号的访问权限，并且设置几个默认的管理员用户，因为上面配置的是和openldap集成，所以配置的管理员用户，是openldap里面存在的用户，这样才能登陆。

```yaml
auth:
  admin:
    access: true
    users:
      - user1
      - user2
```
&emsp;&emsp;到这里对于hub的设置就基本完成了，其他的一些配置，可以参考values.yaml文件去自定义，下一篇说下hub启动的singleuser镜像的配置，自定义一些基础配置。