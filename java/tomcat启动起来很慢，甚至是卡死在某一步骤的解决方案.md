我们在服务器上启动tomcat的时候，偶尔会碰到tomcat启动起来特别慢，甚至是卡死在某一步的情况，下面记录了我一次在CentOS上启动tomcat，使用**./bin/startup.sh**命令启动后，通过命令 **tail -f logs/catalina.out** 查看tomcat日志
```
[root@localhost ~]# cd tomcat/
[root@localhost tomcat]# ./bin/startup.sh 
Using CATALINA_BASE:   /root/tomcat
Using CATALINA_HOME:   /root/tomcat
Using CATALINA_TMPDIR: /root/tomcat/temp
Using JRE_HOME:        /usr/local/java
Using CLASSPATH:       /root/tomcat/bin/bootstrap.jar:/root/tomcat/bin/tomcat-juli.jar
Tomcat started.
[root@localhost tomcat]# tail -f logs/catalina.out 
01-Jun-2019 18:02:38.514 信息 [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Djava.io.tmpdir=/root/tomcat/temp
01-Jun-2019 18:02:38.514 信息 [main] org.apache.catalina.core.AprLifecycleListener.lifecycleEvent The APR based Apache Tomcat Native library which allows optimal performance in production environments was not found on the java.library.path: [/usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib]
01-Jun-2019 18:02:38.701 信息 [main] org.apache.coyote.AbstractProtocol.init Initializing ProtocolHandler ["http-nio-8080"]
01-Jun-2019 18:02:38.723 信息 [main] org.apache.tomcat.util.net.NioSelectorPool.getSharedSelector Using a shared selector for servlet write/read
01-Jun-2019 18:02:38.741 信息 [main] org.apache.coyote.AbstractProtocol.init Initializing ProtocolHandler ["ajp-nio-8009"]
01-Jun-2019 18:02:38.742 信息 [main] org.apache.tomcat.util.net.NioSelectorPool.getSharedSelector Using a shared selector for servlet write/read
01-Jun-2019 18:02:38.754 信息 [main] org.apache.catalina.startup.Catalina.load Initialization processed in 774 ms
01-Jun-2019 18:02:38.788 信息 [main] org.apache.catalina.core.StandardService.startInternal Starting service [Catalina]
01-Jun-2019 18:02:38.788 信息 [main] org.apache.catalina.core.StandardEngine.startInternal Starting Servlet Engine: Apache Tomcat/8.5.29
01-Jun-2019 18:02:38.821 信息 [localhost-startStop-1] org.apache.catalina.startup.HostConfig.deployDirectory Deploying web application directory [/root/tomcat/webapps/ROOT]
```
由上面日志可以看出，tomcat没有启动成功，但是也没有任何报错信息，但是过去了十几二十分钟之后，依然没有启动成功，始终卡死在 **org.apache.catalina.startup.HostConfig.deployDirectory Deploying web application directory** 这一句。通过在网上搜索了半天终于解决了这个问题，在这里对这个问题做一下记录，免得以后碰到又再去搜索
这个问题的起因是tomcat启动的时候，需要通过Linux生成随机数的方式/dev/random生成随机数，但是如果生成随机数失败的话，就会一直阻塞中，这也就是tomcat启动为什么一直卡死某一行日志的原因，经过测试，可以通过配置/dev/urandom来取代/dev/random来解决，我们可以在两个地方进行配置，都可以解决这个问题，取其一即可
##### 1. 在jdk安装目录配置
编辑$JAVA_HOME/jre/lib/security/Java.security文件，将**securerandom.source=file:/dev/random** 换成 **securerandom.source=file:/dev/urandom** 即可

##### 2. 在tomcat配置文件catalina.sh中修改
tomcat的catalina.sh文件位于tomcat目录下的bin目录下，通过在catalina.sh文件添加 **-Djava.security.egd=file:/dev/urandom**，如下所示
![](https://upload-images.jianshu.io/upload_images/8782952-c43e59674451a490.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
##### 3. 以上两种解决方式，只需要配置一种即可，在jdk目录中进行配置，可以一劳永逸，不需要每个tomcat中进行配置，但是一般情况下我们对生产环境下的jdk没有操作权限，这时候可以使用在tomcat中配置来进行解决
配置完之后，再次启动tomcat，可以发现tomcat很快就启动成功了
```
[root@localhost tomcat]# tail -f logs/catalina.out 
01-Jun-2019 18:27:46.100 信息 [localhost-startStop-1] org.apache.catalina.startup.HostConfig.deployDirectory Deployment of web application directory [/root/tomcat/webapps/docs] has finished in [27] ms
01-Jun-2019 18:27:46.100 信息 [localhost-startStop-1] org.apache.catalina.startup.HostConfig.deployDirectory Deploying web application directory [/root/tomcat/webapps/examples]
01-Jun-2019 18:27:46.406 信息 [localhost-startStop-1] org.apache.catalina.startup.HostConfig.deployDirectory Deployment of web application directory [/root/tomcat/webapps/examples] has finished in [306] ms
01-Jun-2019 18:27:46.409 信息 [localhost-startStop-1] org.apache.catalina.startup.HostConfig.deployDirectory Deploying web application directory [/root/tomcat/webapps/host-manager]
01-Jun-2019 18:27:46.436 信息 [localhost-startStop-1] org.apache.catalina.startup.HostConfig.deployDirectory Deployment of web application directory [/root/tomcat/webapps/host-manager] has finished in [28] ms
01-Jun-2019 18:27:46.450 信息 [localhost-startStop-1] org.apache.catalina.startup.HostConfig.deployDirectory Deploying web application directory [/root/tomcat/webapps/manager]
01-Jun-2019 18:27:46.477 信息 [localhost-startStop-1] org.apache.catalina.startup.HostConfig.deployDirectory Deployment of web application directory [/root/tomcat/webapps/manager] has finished in [27] ms
01-Jun-2019 18:27:46.494 信息 [main] org.apache.coyote.AbstractProtocol.start Starting ProtocolHandler ["http-nio-8080"]
01-Jun-2019 18:27:46.516 信息 [main] org.apache.coyote.AbstractProtocol.start Starting ProtocolHandler ["ajp-nio-8009"]
01-Jun-2019 18:27:46.521 信息 [main] org.apache.catalina.startup.Catalina.start Server startup in 924 ms
```
