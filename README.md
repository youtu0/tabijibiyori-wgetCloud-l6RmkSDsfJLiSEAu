
Jave Web是java面向web开发的相关技术，他是相关技术的统称，并不是指某一个单一的技术。在我之前的博客中（Java网络编程\-\-\-\-通过实现简易聊天工具来聊聊BIO模型 https://github.com/jilodream/p/17405923\.htm），就已经写到过java可以作为一个服务器（如TCP/UDP），接收外部的请求。如使用TCP监听端口，然后直接用web页面请求该端口，那么服务器就会接收到相关的响应，接着做好业务处理，返回响应的请求即可。但是整个的业务流程太繁琐了。我们不但要处理业务流程，还要控制请求会话，还要控制各种业务分支的处理，显然这不是我们想要的。于是聪明的开发者很快想到了\-\-\-\-\-解耦，业务人员只要编写相关的业务即可，不需要关心繁琐的网络细节处理，因此就诞生了servlet。开发人员只要实现servlet，而servlet和不同的路径绑定。web请求后，由系统直接转发到各自的Servlet，并由Servlet来处理相关业务即可。那什么是servlet呢？servlet 是**Server Applet**（服务器应用程序）的简写，从名字我们就可以看出它是专门用来编写服务器端的应用程序。我们通常用它来处理服务器中http请求的处理和响应。它是一项很古老的技术，随java诞生之初就已经问世，很多人甚至都不知Servlet是做什么。那么问题来了，我们为什么还要学习和掌握Servlet呢？这主要是由于Servlet是javaEE的重要组成，是java web开发的重要基石。我们现在项目中常用到的Jsp、Springmvc、Springboot等框架，在处理网络请求的核心技术，仍然是Servlet。我们虽然不需要再继续直面Servlet或更底层的技术进行开发，但是Servlet究竟是什么样的，如何执行，以及再新技术中承担什么样的角色，这个却是我们想要熟悉底层原理所必须要掌握的。


话不多说，想要使用Servlet，我们需要做两步：


1、编写Servlet相关业务代码2、将业务代码打包放置在Tomcat中，由Tomcat来加载这些Servlet


**第一步，试着来编写一个Servlet**我们首先通过IDEA 新建一个web项目，此处我们选择采用maven部署，搭建好之后项目整体的结构就如下面的文件层级树一样了




```
E:.
├─.idea
├─.smarttomcat
│  └─PureJaveServlet
│      └─conf
└─src
    └─main
        ├─java
        │  └─org
        │      └─example
        ├─resources
        └─webapp
```


其中：


resources 一般是我们填写的资源信息，如图片，业务配置文件等，webapp则会放置html、css等渲染文件，还会放一些web组件（Servlet、Filter）的配置信息。我们这里只要知道作用即可。src/main/java则是我们的业务代码。值得注意的是，我们要引入Servlet网络编程的相关依赖包，才能进行相关的web开发。默认的java SE是不包含这些开发包的。在Servlet4\.0之前，我们只使用javax相关的包，并且未来对接的是Tomcat 9\.x及以下版本。(防盗连接：本文首发自https://github.com/jilodream/ )在Servlet5\.0之后，我们只使用javax相关的包，并且未来对接的是Tomcat 10\.x及以后版本。这里我们使用高版本来学习，即jakarta版本，未来tomcat也需要使用高版本。


接着编写POM文件：




```
 1 </spanxml version="1.0" encoding="UTF-8"?>
 2 <project xmlns="http://maven.apache.org/POM/4.0.0"
 3          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 4          xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
 5 <modelVersion>4.0.0modelVersion>
 6 
 7 <groupId>org.examplegroupId>
 8 <artifactId>PureJaveServletartifactId>
 9 <version>1.0-SNAPSHOTversion>
10 <name>PureJaveServletname>
11 <packaging>warpackaging>
12 
13 <properties>
14     <project.build.sourceEncoding>UTF-8project.build.sourceEncoding>
15     <maven.compiler.target>11maven.compiler.target>
16     <maven.compiler.source>11maven.compiler.source>
17 properties>
18 
19 <dependencies>
20     <dependency>
21         <groupId>jakarta.servletgroupId>
22         <artifactId>jakarta.servlet-apiartifactId>
23         <version>5.0.0version>
24         <scope>providedscope>
25     dependency>
26 
27     <dependency>
28         <groupId>org.projectlombokgroupId>
29         <artifactId>lombokartifactId>
30         <version>1.18.30version>
31     dependency>
32     <dependency>
33         <groupId>org.apache.commonsgroupId>
34         <artifactId>commons-lang3artifactId>
35         <version>3.13.0version>
36     dependency>
37     
38     
39     <dependency>
40         <groupId>com.alibabagroupId>
41         <artifactId>fastjsonartifactId>
42         <version>1.2.83version>
43     dependency>
44 
45 
46     <dependency>
47         <groupId>com.fasterxml.jackson.coregroupId>
48         <artifactId>jackson-databindartifactId>
49         <version>2.10.0version>
50     dependency>
51     <dependency>
52         <groupId>io.pebbletemplatesgroupId>
53         <artifactId>pebbleartifactId>
54         <version>3.1.6version>
55     dependency>
56     <dependency>
57         <groupId>org.apache.maven.pluginsgroupId>
58         <artifactId>maven-compiler-pluginartifactId>
59         <version>3.10.1version>
60     dependency>
61 
62 dependencies>
63 
64 <build>
65     <plugins>
66         <plugin>
67             <groupId>org.apache.maven.pluginsgroupId>
68             <artifactId>maven-war-pluginartifactId>
69             <version>3.3.2version>
70         plugin>
71         <plugin>
72             <groupId>org.apache.maven.pluginsgroupId>
73             <artifactId>maven-compiler-pluginartifactId>
74             <configuration>
75                 <compilerArgs>
76                     <arg>-parametersarg>
77                 compilerArgs>
78             configuration>
79         plugin>
80     plugins>
81 build>
82 project>
```


 


Servlet类




```
 1 package org.example;
 2 
 3 
 4 import jakarta.servlet.ServletException;
 5 import jakarta.servlet.annotation.WebServlet;
 6 import jakarta.servlet.http.HttpServlet;
 7 import jakarta.servlet.http.HttpServletRequest;
 8 import jakarta.servlet.http.HttpServletResponse;
 9 
10 import java.io.IOException;
11 import java.io.PrintWriter;
12 
13 
14 @WebServlet(urlPatterns = "/nihao")
15 public class HiServlet extends HttpServlet {
16     @Override
17     protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws  IOException {
18         String name = req.getParameter("name");
19         resp.setContentType("text/html");
20         PrintWriter out = resp.getWriter();
21         out.println("");
22         out.println(String.format("# Hello, %s

", name));
23         out.println("");
24         out.flush();
25     }
26 }
```


 




```
 1 package org.example;
 2 
 3 import jakarta.servlet.annotation.WebServlet;
 4 import jakarta.servlet.http.HttpServlet;
 5 import jakarta.servlet.http.HttpServletRequest;
 6 import jakarta.servlet.http.HttpServletResponse;
 7 
 8 import java.io.IOException;
 9 import java.io.PrintWriter;
10 
11 /**
12  * @discription
13  */
14 @WebServlet(urlPatterns = "/bye")
15 public class ByeServlet extends HttpServlet {
16     @Override
17     protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws IOException {
18         String name = req.getParameter("name");
19         resp.setContentType("text/html");
20         PrintWriter out = resp.getWriter();
21         out.println("");
22         out.println(String.format("# bye bye, %s

", name));
23         out.println("");
24         out.flush();
25     }
26 }
```


代码部分就结束了，我们观察代码可以发现两点：1、所有的Sevlet都要继承自HttpServlet。HttpServlet是一个抽象类，我们需要复写抽象类中的抽象方法，以保证未来Tomcat等web服务器在加载Servlet时，可以按照统一的规范查找，执行。我们在Servlet中重写了doGet() 方法，表示处理该servlet路径下的get请求，同理还可以重写doPost doDelete doPut等方法，来处理对应的请求类型。这里我们很简单，直接返回一段响应的html。2、我们并没有写main方法，而是在Pom中标记我们的工程需要打包成一个war包。**第二步，配置tomcat**没有main方法，我们如何启动我们的java程序呢？我们通常是将其配置到tomcat的指定路径中，启动tomcat后，tomcat会加载war包中servlet的相关类，进行处理。因此我们会将tomcat这样的web服务器称之为Servlet容器。我们首先从tomcat官网上（https://tomcat.apache.org/whichversion.html）下载一个与我们对应的servlet版本匹配的tomcat版本。


下载到本地之后解压即可。接着我们为了方便在IDEA中下载一个smart tomcat的组件，将该组件关联好servlet代码和tomcat服务器即可。关键配置如下：Tomcat server: 选择我们下载好的tomcat服务器，如果没有下拉选项就选择"Congure..."手动加一下。Deployment dirctory：部署文件夹，(防盗连接：本文首发自https://github.com/jilodream/ )该配置指向前文项目结构树种的webapp。Use classpath of module：选择当前项目Context path：选择上下文路径（其实就是url的前缀路径），按照url规则随便填，我这里叫填的是/biubiubiuserver port：Servlet的服务器端口，默认填8080admin port：tomcat的管理端口 默认填8005，一般就是用于停掉tomcat（其实一般也不用）。之后我们通过IDEA：拉起tomcat，加载servlet相关类和资源。


命令行输入如下：




```
....15-Nov-2024 10:34:38.099 信息 [main] org.apache.coyote.AbstractProtocol.start 开始协议处理句柄["http-nio-8080"]
15-Nov-2024 10:34:39.831 信息 [main] org.apache.catalina.startup.Catalina.start [4858]毫秒后服务器启动
http://localhost:8080/biubiubiu
```


执行效果如下：


![](https://img2024.cnblogs.com/blog/704073/202411/704073-20241115103826703-669433307.png)


有人会觉得通过下载并配置tomcat有点麻烦，我们如果想debug代码的话，就更麻烦了，有没有简单点的办法：其实除了下载tomcat，我们还可以通过代码的形式直接拉起tomcat。思路如下：首先通过maven加载对应tomcat依赖，然后在main方法中创建tomcat实例，并且指定tomcat所需要的配置信息，如资源和class路径。然后通过start()方法启动tomcat实例就可以了。代码如下：


新搭建一个java web项目，Servlet类和工程结构我们保持不变还是和原来一样


POM文件




```
 1 </spanxml version="1.0" encoding="UTF-8"?>
 2 <project xmlns="http://maven.apache.org/POM/4.0.0"
 3          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 4          xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
 5     <modelVersion>4.0.0modelVersion>
 6 
 7     <groupId>com.examplegroupId>
 8     <artifactId>demotomartifactId>
 9     <version>1.0-SNAPSHOTversion>
10     <name>demotomname>
11     <packaging>warpackaging>
12 
13     <properties>
14         <project.build.sourceEncoding>UTF-8project.build.sourceEncoding>
15         <maven.compiler.target>11maven.compiler.target>
16         <maven.compiler.source>11maven.compiler.source>
17         <tomcat.version>10.0.0tomcat.version>
18     properties>
19 
20     <dependencies>
21 
22 
23         <dependency>
24             <groupId>org.apache.tomcat.embedgroupId>
25             <artifactId>tomcat-embed-coreartifactId>
26             <version>${tomcat.version}version>
27             <scope>providedscope>
28         dependency>
29         <dependency>
30             <groupId>org.apache.tomcat.embedgroupId>
31             <artifactId>tomcat-embed-jasperartifactId>
32             <version>${tomcat.version}version>
33             <scope>providedscope>
34         dependency>
35     dependencies>
36 
37     <build>
38         <plugins>
39             <plugin>
40                 <groupId>org.apache.maven.pluginsgroupId>
41                 <artifactId>maven-war-pluginartifactId>
42                 <version>3.3.2version>
43             plugin>
44         plugins>
45     build>
46 project>
```


主类：




```
 1 package com.example.demotom;
 2 
 3 import org.apache.catalina.Context;
 4 import org.apache.catalina.LifecycleException;
 5 import org.apache.catalina.WebResourceRoot;
 6 import org.apache.catalina.startup.Tomcat;
 7 import org.apache.catalina.webresources.DirResourceSet;
 8 import org.apache.catalina.webresources.StandardRoot;
 9 
10 import java.io.File;
11 
12 /**
13  * @discription
14  */
15 public class TomMain {
16     public static void main(String[] args) throws LifecycleException {
17         Tomcat tomcat = new Tomcat();
18         tomcat.setPort(Integer.getInteger("port", 8080));
19         tomcat.getConnector();
20         String docBase = new File("src/main/webapp").getAbsolutePath();
21         Context ctx = tomcat.addContext("", docBase);
22         WebResourceRoot resources = new StandardRoot(ctx);
23         String base = new File("target/classes").getAbsolutePath();
24         resources.addJarResources(new DirResourceSet(resources, "/WEB-INF/classes", base, "/"));
25         ctx.setResources(resources);
26         tomcat.start();
27         tomcat.getServer().await();
28     }
29 }
```


启动后控制台输出如下，我们可以看到8080端口已经被监听：




```
"C:\Program Files\Java\jdk-11\bin\java.exe" -agentlib:jdwp=transport=dt_socket,address=127.0.0.1:51854,suspend=y,server=n -Dfile.encoding=UTF-8 -classpath ....
Connected to the target VM, address: '127.0.0.1:51854', transport: 'socket'
11月 15, 2024 10:47:54 上午 org.apache.coyote.AbstractProtocol init
信息: Initializing ProtocolHandler ["http-nio-8080"]
11月 15, 2024 10:47:54 上午 org.apache.catalina.core.StandardService startInternal
信息: Starting service [Tomcat]
11月 15, 2024 10:47:54 上午 org.apache.catalina.core.StandardEngine startInternal
信息: Starting Servlet engine: [Apache Tomcat/10.0.0]
11月 15, 2024 10:47:56 上午 org.apache.catalina.util.SessionIdGeneratorBase createSecureRandom
警告: Creation of SecureRandom instance for session ID generation using [SHA1PRNG] took [1,753] milliseconds.
11月 15, 2024 10:47:56 上午 org.apache.coyote.AbstractProtocol start
信息: Starting ProtocolHandler ["http-nio-8080"]
```


 


 本博客参考[FlowerCloud机场](https://hushicha.org)。转载请注明出处！
