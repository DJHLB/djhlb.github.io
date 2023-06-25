---
title: CentOS7安装配置Tomcat9
date: 2021-07-01 11:19:43
tags:
---
#### 什么是Tomcat

Tomcat是由Apache软件基金会下属的Jakarta项目开发的一个Servlet容器，按照Sun Microsystems提供的技术规范，实现了对Servlet和JavaServer Page（JSP）的支持，并提供了作为Web服务器的一些特有功能，如Tomcat管理和控制平台、安全域管理和Tomcat阀等。由于Tomcat本身也内含了一个HTTP服务器，它也可以被视作一个单独的Web服务器。但是，不能将Tomcat和Apache HTTP服务器混淆，Apache HTTP服务器是一个用C语言实现的HTTPWeb服务器；这两个HTTP web server不是捆绑在一起的。Apache Tomcat包含了一个配置管理工具，也可以通过编辑XML格式的配置文件来进行配置。

#### Tomcat同类产品

- Resin服务器

> Resin是Caucho公司的产品，是一个非常流行的支持Servlet和JSP的服务器，速度非常快。Resin本身包含了一个支持HTML的Web服务器，这使它不仅可以显示动态内容，而且显示静态内容的能力也毫不逊色，因此许多网站都是使用Resin服务器构建。

- Jetty服务器

>Jetty是一个纯粹的基于Java的网页服务器和Java Servlet容器。尽管网页服务器通常用来为人们呈现文档，但是Jetty通常在较大的软件框架中用于计算机与计算机之间的通信。Jetty作为Eclipse基金会的一部分，是一个自由和开源项目。

- JBoss服务器

> JBoss是一个种遵从JavaEE规范的、开放源代码的、纯Java的EJB服务器，对于J2EE有很好的支持。JBoss采用JML API实现软件模块的集成与管理，其核心服务又是提供EJB服务器，不包含Servlet和JSP的Web容器，不过它可以和Tomcat完美结合。

- WebShpere服务器

> WebSphere是IBM公司的产品，可进一步细分为 WebSphere Performance Pack、Cache Manager 和WebSphere Application Server等系列，其中WebSphere Application Server 是基于Java 的应用环境，可以运行于 Sun Solaris、Windows NT 等多种操作系统平台，用于建立、部署和管理Internet和Intranet Web应用程序。

- WebLogic服务器

> WebLogic 是BEA公司的产品，可进一步细分为 WebLogic Server、WebLogic Enterprise 和 WebLogic Portal 等系列，其中 WebLogic Server 的功能特别强大。WebLogic 支持企业级的、多层次的和完全分布式的Web应用，并且服务器的配置简单、界面友好。对于那些正在寻求能够提供Java平台所拥有的一切应用服务器的用户来说，WebLogic是一个十分理想的选择。

#### 使用yum安装Tomcat

搜索有关Tomcat的软件包。

```bash
# yum list |grep tomcat
jglobus-ssl-proxies-tomcat.noarch       2.1.0-6.el7                    epel     
tomcat.noarch                           7.0.76-3.el7_4                 updates  
tomcat-admin-webapps.noarch             7.0.76-3.el7_4                 updates  
tomcat-docs-webapp.noarch               7.0.76-3.el7_4                 updates  
tomcat-el-2.2-api.noarch                7.0.76-3.el7_4                 updates  
tomcat-javadoc.noarch                   7.0.76-3.el7_4                 updates  
tomcat-jsp-2.2-api.noarch               7.0.76-3.el7_4                 updates  
tomcat-jsvc.noarch                      7.0.76-3.el7_4                 updates  
tomcat-lib.noarch                       7.0.76-3.el7_4                 updates  
tomcat-native.x86_64                    1.2.16-1.el7                   epel     
tomcat-servlet-3.0-api.noarch           7.0.76-3.el7_4                 updates  
tomcat-webapps.noarch                   7.0.76-3.el7_4                 updates  
tomcatjss.noarch                        7.2.1-6.el7                    base     
```

在CentOS7中默认提供Tomcat7版本,安装时只需安装tomcat,tomcat-admin-webapps,tomcat-doc-webapp,tomcat-webapps即可,其它包会自动安装上。

```bash
# yum install -y tomcat tomcat-admin-webapps tomcat-docs-webapp tomcat-webapps
Installed:
  tomcat.noarch 0:7.0.76-3.el7_4                                   
  tomcat-admin-webapps.noarch 0:7.0.76-3.el7_4                     
  tomcat-docs-webapp.noarch 0:7.0.76-3.el7_4                       
  tomcat-webapps.noarch 0:7.0.76-3.el7_4                           
 
Dependency Installed:
  apache-commons-logging.noarch 0:1.1.2-7.el7                      
  avalon-framework.noarch 0:4.3-10.el7                             
  avalon-logkit.noarch 0:2.1-14.el7                                
  jakarta-taglibs-standard.noarch 0:1.1.2-14.el7_1                 
  tomcat-el-2.2-api.noarch 0:7.0.76-3.el7_4                        
  tomcat-jsp-2.2-api.noarch 0:7.0.76-3.el7_4                       
  tomcat-lib.noarch 0:7.0.76-3.el7_4                               
  tomcat-servlet-3.0-api.noarch 0:7.0.76-3.el7_4                   
 
Complete!
```

#### 使用二进制包安装Tomcat

- 下载Tomcat,文章使用9.0版本

  [官网地址](https://tomcat.apache.org/download-90.cgi)

  ```bash
  curl -O http://mirrors.shu.edu.cn/apache/tomcat/tomcat-9/v9.0.6/bin/apache-tomcat-9.0.6.tar.gz
  ```

- 解压源码包放在/usr/local目录下

```bash
# tar xvf apache-tomcat-9.0.6.tar.gz -C /usr/local/
```

- 创建链接文件

```bash
ln -sv /usr/local/apache-tomcat-9.0.6 /usr/local/tomcat
```

- 安装JDK

  Tomcat9需要JDK8及以上版本

  [JDK下载地址](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)

  ```bash
  # wget --no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u162-b12/0da788060d494f5095bf8624735fa2f1/jdk-8u162-linux-x64.tar.gz
  ```

- 配置CentOS7使用JDK1.8版本

```bash
# mkdir -p /usr/local/jdk-oracal
# tar xvf jdk-8u162-linux-x64.tar.gz -C /usr/local/jdk-oracal/
# cd /etc/profile.d/
# vi java1.8.sh
export JAVA_HOME=/usr/local/jdk-oracal/jdk1.8.0_162
export PATH=$JAVA_HOME/bin:$PATH
# source java1.8.sh
# java -version
java version "1.8.0_162"
Java(TM) SE Runtime Environment (build 1.8.0_162-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.162-b12, mixed mode)
```

- 配置Tomcat9启动环境

  ```bash
  # vi /etc/profile.d/tomcat9.sh
  export CATALINA_HOME=/usr/local/tomcat9
  export PATH=$TOMCAT_HOME/bin:$PATH
  # source /etc/profile.d/tomcat9.sh
  ```

  官方建议在Tomcat安装目录的bin目录下建一个setenv.sh,将JAVA_HOME,JRE_HOME等环境变量信息指定。示例如下：

  ```bash
  # cat bin/setenv.sh 
  CATALINA_HOME=/usr/local/tomcat9
  CATALINA_BASE=/usr/local/tomcat9
  JAVA_HOME=/usr/local/jdk-oracal/jdk1.8.0_162
  JRE_HOME=/usr/local/jdk-oracal/jdk1.8.0_162/jre/
  CATALINA_PID=/usr/local/tomcat9/tomcat9.pid
  ```

- 启动Tomcat9

```bash
# catalina.sh start
Using CATALINA_BASE:   /usr/local/tomcat9
Using CATALINA_HOME:   /usr/local/tomcat9
Using CATALINA_TMPDIR: /usr/local/tomcat9/temp
Using JRE_HOME:        /usr/local/jdk-oracal/jdk1.8.0_162
Using CLASSPATH:       /usr/local/tomcat9/bin/bootstrap.jar:/usr/local/tomcat9/bin/tomcat-juli.jar
Tomcat started.
# ss -tnpl |grep 8080
LISTEN     0      100         :::8080                    :::*                   users:(("java",pid=4177,fd=49))
```

- 使用systemd管理Tomcat9服务

```bash
# cat /usr/lib/systemd/system/tomcat9.service 
[Unit]
Description=Apache Tomcat 9
After=syslog.target network.target remote-fs.target nss-lookup.target
 
[Service]
Type=forking
PIDFile=/usr/local/tomcat9/tomcat9.pid
ExecStart=/usr/local/tomcat9/bin/catalina.sh start -DEFOREGRAND
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
RemainAfterExit=yes
 
[Install]
WantedBy=multi-user.target
```

- 配置访问Tomcat9管理界面

  修改conf/tomat-users.xml，添加以下内容

  ```bash
  <!--  
    <role rolename="tomcat"/>
    <role rolename="role1"/>
    <user username="tomcat" password="tomcat" roles="tomcat"/>
    <user username="both" password="tomcat" roles="tomcat,role1"/>
    <user username="role1" password="tomcat" roles="role1"/>
  -->
    <role rolename="manager-gui"/>
    <role rolename="manager-script" />
    <user username="admin" password="tomcat" roles="manager-gui,manager-script" />
  </tomcat-users>
  ```

  修改webapps/manager/META-INF目录下的content.xml，在allow行的末尾加上|\d+.\d+.\d+.\d+.表示允许所有主机访问。

  ```bash
  <Context antiResourceLocking="false" privileged="true" >
    <Valve className="org.apache.catalina.valves.RemoteAddrValve"
           allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1|\d+\.\d+\.\d+\.\d+" />
    <Manager sessionAttributeValueClassNameFilter="java\.lang\.(?:Boolean|Integer|Long|Number|String)|org\.apache\.catalina\.filters\.CsrfPreventionFilter\$LruCache(?:\$1)?|java\.util\.(?:Linked)?HashMap"/>
  </Context>
  ```

  重启Tomcat9生效
