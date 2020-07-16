# APM监控工具-Pinpoint实践
## Pinpoint技术架构
> Pinpoint是一个开源的 APM (Application Performance Management/应用性能管理)工具，用于基于java的大规模分布式系统。在使用上力图简单高效，通过在启动时安装agent，不需要修改哪怕一行代码，最小化性能损失(3%)。
<br><br>同时，Pinpoint也是一款全链路分析工具，提供了无侵入式的调用链监控、方法执行详情查看、应用状态信息监控等功能。基于GoogleDapper论文进行的实现，与另一款开源的全链路分析工具Zipkin类似，但相比Zipkin提供了无侵入式、代码维度的监控等更多的特性。 
### 功能
Pinpoint支持的功能比较丰富，可以支持如下几种功能：
- 服务拓扑图：对整个系统中应用的调用关系进行了可视化的展示，单击某个服务节点，可以显示该节点的详细信息，比如当前节点状态、请求数量等。
- 实时活跃线程图：监控应用内活跃线程的执行情况，对应用的线程执行性能可以有比较直观的了解。
- 请求响应散点图：以时间维度进行请求计数和响应时间的展示，拖过拖动图表可以选择对应的请求查看执行的详细情况。
![](http://images.intflag.com/pinpoint01-002.png)
- 请求调用栈查看：对分布式环境中每个请求提供了代码维度的可见性，可以在页面中查看请求针对到代码维度的执行详情，帮助查找请求的瓶颈和故障原因。
![](http://images.intflag.com/pinpoint01-003.png)
- 应用状态、机器状态检查：通过这个功能可以查看相关应用程序的其他的一些详细信息，比如CPU使用情况，内存状态、垃圾收集状态，TPS和JVM信息等参数。
![](http://images.intflag.com/pinpoint01-004.png)

### 架构组成
![](http://images.intflag.com/pinpoint01-001.jpg)
Pinpoint 主要由 3 个组件外加 Hbase 数据库组成，三个组件分别为：Agent、Collector 和 Web UI。
- Agent组件：用于收集应用端监控数据，无侵入式，只需要在启动命令中加入部分参数即可
- Collector组件：数据收集模块，接收Agent发送过来的监控数据，并存储到HBase
- WebUI：监控展示模块，展示系统调用关系、调用详情、应用状态等，并支持报警等功能

## Pinpoint-1.7.3源码编译
### 参考资料
- github：https://github.com/naver/pinpoint
- 官方文档：https://naver.github.io/pinpoint/index.html
- 博客：https://www.cnblogs.com/yyhh/p/6106472.html
- 博客：https://www.jianshu.com/p/a8482f01af4a（推荐）

### 下载源码
- 进入github上的releases页面：https://github.com/naver/pinpoint/releases
- 找到准备编译的版本，本次我们选择1.7.3，（注意：如果想直接部署，可以下载已经编译好的三个包：pinpoint-agent-1.7.3.tar.gz、pinpoint-collector-1.7.3.war、pinpoint-web-1.7.3.war直接部署即可，部署过程可以参考下一小节）
![](http://images.intflag.com/pinpoint01-002.jpg)

### 配置编译环境
- 安装JDK6、7、8，然后配置环境变量，同时你的默认环境变量JAVA_HOME必须是JDK7以上的（注意：如果版本很新，也需要安装jdk9并配置环境变量）
- 安装maven-3.6.3并配置环境变量，（注意：之前使用3.3.9编译时会报错，后更换3.6.3问题解决）
![](http://images.intflag.com/pinpoint01-003.jpg)

### 编译Pinpoint-1.7.3源码
- 将下载的源码解压到非中文目录下，然后进入windows命令行窗口
![](http://images.intflag.com/pinpoint01-004.jpg)
- 执行命令：`mvn install -Dmaven.test.skip=true`进行编译
![](http://images.intflag.com/pinpoint01-005.jpg)
但是，在我本地却卡在了一个地方，如图
![](http://images.intflag.com/pinpoint01-006.jpg)
出现这个问题的原因是因为，maven要执行`npm install`去下载依赖，但是没有自动执行，所以需要我们手动执行一下这个命令，然后再执行一遍`mvn install -Dmaven.test.skip=true`进行编译，或者先去`web/target`目录下执行`npm install`命令，然后回到项目根目录执行编译。
![](http://images.intflag.com/pinpoint01-007.jpg)
- 当出现`BUILD SUCCESS`即表示编译成功，初次编译会下载很多依赖，时间特别长，至少需要30分钟。
![](http://images.intflag.com/pinpoint01-008.jpg) 

## 使用idea运行Pinpoint项目
### 配置Pinpoint运行环境
- 下载Hbase，解压到非中文目录下
- 配置`hbase-1.4.12\conf\hbase-env.cmd`，设置JAVA_HOME环境变量，如果是linux环境请修改hbase-env.sh文件。
![](http://images.intflag.com/pinpoint01-009.jpg) 
- 配置`hbase-1.4.12\conf\hbase-site.xml`，配置hbase和zookeeper数据文件地址，配置后在Hbase启动时会自动创建。
```
<configuration>
    <property>
        <name>hbase.rootdir</name>
        <value>file:///D:\010-WorkingSpace\099-AppData\hbase_data</value>
    </property>
    <property>
        <name>hbase.zookeeper.property.dataDir</name>
        <value>D:\010-WorkingSpace\099-AppData\zookeeper_data</value>
    </property>
</configuration>
```
- 如果此时启动Hbase可能会报错，建议安装Hadoop。
- 下载Hadoop，解压到非中文目录下，配置环境变量
![](http://images.intflag.com/pinpoint01-010.jpg) 
- 启动HBase，进入bin目录，运行start-hbase.cmd文件；此时使用的是hbase自带的zookeeper。
- 使用jps命令查看hbase进程是否启动
```
D:\java\hbase\hbase-1.4.2\bin>jps
14048 Program
15256 HMaster
20056
8952 Jps
```
- 初始化Pinpoint需要的数据表，数据表脚本在pinpoint源码目录\hbase\scripts\hbase-create.hbase文件中
```
hbase shell E:\workspace\git\pinpoint-master\hbase\scripts\hbase-create.hbase
```
- 查看导入的数据表，hbase shell进入hbase交互环境后，执行list命令查看表
```
D:\java\hbase\hbase-1.4.2\bin>hbase shell
hbase(main):001:0> list
TABLE                                                                                                                   
AgentEvent                                                                                                              
AgentInfo                                                                                                               
AgentLifeCycle                                                                                                          
AgentStatV2                                                                                                             
ApiMetaData                                                                                                             
ApplicationIndex                                                                                                        
ApplicationMapStatisticsCallee_Ver2                                                                                     
ApplicationMapStatisticsCaller_Ver2                                                                                     
ApplicationMapStatisticsSelf_Ver2                                                                                       
ApplicationStatAggre                                                                                                    
ApplicationTraceIndex                                                                                                   
HostApplicationMap_Ver2                                                                                                 
SqlMetaData_Ver2                                                                                                        
StringMetaData                                                                                                          
TraceV2                                                                                                                 
15 row(s) in 0.2060 seconds
```
### idea运行Pinpoint
- 使用idea打开Pinpoint源码
- 配置Tomcat，现在项目，点击菜单栏的`Run`按钮，然后选择`Edit configurations`。
![](http://images.intflag.com/pinpoint01-011.jpg) 
- 新增一个Tomcat，这里使用`apache-tomcat-8.5.31`。
![](http://images.intflag.com/pinpoint01-012.jpg) 
- 点击`Development`，然后点击加号，选择web和collector的war包。
![](http://images.intflag.com/pinpoint01-013.jpg) 
- 启动项目，注意端口占用情况。
![](http://images.intflag.com/pinpoint01-014.jpg) 
- 启动成功后，idea会自动打开Pinpoint的web界面。
![](http://images.intflag.com/pinpoint01-015.jpg) 

## Centos7单点方式部署Pinpoint
### 部署环境准备
- jdk-1.8
- hbase-1.5.0-bin.tar.gz
- apache-tomcat-8.0.36.tar.gz
- pinpoint-web-1.7.3.war
- pinpoint-collector-1.7.3.war
- pinpoint-agent-1.7.3.tar.gz
### 安装jdk
- 查看已安装的jdk
```
[root@localhost jdk1.8.0_171]# rpm -qa | grep java
tzdata-java-2017b-1.el7.noarch
python-javapackages-3.4.1-11.el7.noarch
java-1.8.0-openjdk-1.8.0.131-11.b12.el7.x86_64
java-1.8.0-openjdk-headless-1.8.0.131-11.b12.el7.x86_64
javapackages-tools-3.4.1-11.el7.noarch
```
- 删除默认的openjdk
```
[root@localhost jdk1.8.0_171]# rpm -e --nodeps java-1.8.0-openjdk-1.8.0.131-11.b12.el7.x86_64 java-1.8.0-openjdk-headless-1.8.0.131-11.b12.el7.x86_64
[root@localhost jdk1.8.0_171]# rpm -qa | grep java
tzdata-java-2017b-1.el7.noarch
python-javapackages-3.4.1-11.el7.noarch
javapackages-tools-3.4.1-11.el7.noarch
[root@localhost jdk1.8.0_171]# 
```
- 解压jdk安装包到非中文目录
```
tar -zxf jdk-8u171-linux-x64.tar.gz -C ../module/
```
- 复制jdk目录：/opt/module/jdk1.8.0_171
- 配置JDK环境变量
```
sudo vi /etc/profile
按shift + G 到文件最后一行
输入下面配置
#JAVA_HOME
export JAVA_HOME=/opt/module/jdk1.8.0_171
export PATH=$PATH:$JAVA_HOME/bin
```
- 刷新配置
```
source /etc/profile
```
### 安装Hbase
- 解压Hbase安装包到非中文目录
```
tar -zxf hbase-1.5.0-bin.tar.gz -C ../module/
```
- 配置`conf\hbase-env.sh`，设置JAVA_HOME环境变量。
- 配置`conf\hbase-site.xml`，配置hbase数据文件地址，配置后在Hbase启动时会自动创建。
```
<configuration>
    <property>
      <name>hbase.rootdir</name>
      <value>file:///opt/module/hbase_data</value>
   </property>
</configuration>
```
- 进入`bin\`，执行`./start-hbase.sh`启动Hbase，启动后使用`jps`命令查看是否有`HMaster`进程。
```
[root@localhost bin]# jps
6001 Jps
5947 HMaster
```
- 初始化Pinpoint需要的数据表，数据表脚本在pinpoint源码目录\hbase\scripts\hbase-create.hbase文件中
```
./hbase shell /opt/software/hbase-create.hbase
```
![](http://images.intflag.com/pinpoint01-018.jpg)
### 部署pinpoint监控程序
- 安装tomcat，将tomcat安装包解压到非中文目录
```
tar -zxf apache-tomcat-8.0.36.tar.gz -C ../module/
```
- 然后分别将`pinpoint-collector`和`pinpoint-web`的war包放到刚才解压的tomcat的`webapps`目录中
```
cp pinpoint-web-1.7.3.war pinpoint-collector-1.7.3.war ../module/apache-tomcat-8.0.36/webapps/
```
- 进入tomcat`bin`目录，执行`./startup.sh`，启动tomcat，启动时可以查看日志`tail -f ../logs/catalina.out`。
- 启动后访问`http://主机:8080/pinpoint-web-1.7.3/`，查看是否能访问`Pinpoint Web`界面。
![](http://images.intflag.com/pinpoint01-016.jpg) 
- 如果懒得输项目名的话，可以配置tomcat的`conf/server.xml`文件进行修改。
```
<Host name="localhost"  appBase="webapps" unpackWARs="true" autoDeploy="true">

    <!--要在Host标签中添加下面这行配置，指定默认访问的目录-->
    <Context docBase="/opt/module/apache-tomcat-8.0.36/webapps/pinpoint-web-1.7.3" path="" debug="0"  reloadable="true"/>

    <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs" prefix="localhost_access_log" suffix=".txt" pattern="%h %l %u %t &quot;%r&quot; %s %b" />
</Host>
```
![](http://images.intflag.com/pinpoint01-017.jpg)

## Centos7集群方式部署Pinpoint
`TODO`
- 可以参考：https://blog.csdn.net/huixueyi/article/details/81117155 或 https://blog.csdn.net/weixin_40592911/article/details/91041747

## SpringBoot程序接入Pinpoint监控
- 将Pinpoint-agent程序解压到准备监控的程序所在主机上
```
tar -zxvf pinpoint-agent-1.7.3.tar.gz -C ../module/pinpoint-agent-1.7.3
```
- 编写程序启动脚本
```
nohup java -javaagent:/opt/module/pinpoint-agent-1.7.3/pinpoint-bootstrap-1.7.3.jar -Dpinpoint.applicationName=pinpointTest -Dpinpoint.agentId=pinpointTest -jar pinpoint-test-a-0.0.1-SNAPSHOT.jar >run.log 2>&1 &
```
- 对脚本授权
```
chmod 777 start.sh
```
- 启动脚本，然后随便访问几个程序的接口
```
./start.sh
```
- 进入Pinpoint Web界面查看
![](http://images.intflag.com/pinpoint01-019.jpg)

## Tomcat程序接入Pinpoint监控
- 将Pinpoint-agent程序部署到准备监控的程序所在的主机上
- 修改`bin/catalina.sh`文件，在最上方（但要位于`#!/bin/sh`下）加入下面的配置
```
CATALINA_OPTS="$CATALINA_OPTS -javaagent:/opt/module/pinpoint-agent-1.7.3/pinpoint-bootstrap-1.7.3.jar"
CATALINA_OPTS="$CATALINA_OPTS -Dpinpoint.agentId=pinpointWebTest"
CATALINA_OPTS="$CATALINA_OPTS -Dpinpoint.applicationName=pinpointWebTest"
```
- 将要部署war包上传到`webapps`目录下
- 在bin目录下执行`./start.sh`启动tomcat
- 访问测试程序的接口，然后进入Pinpoint Web界面查看
![](http://images.intflag.com/pinpoint01-020.jpg)

## Pinpoint问题总结
- 为什么Pinpoint-Web界面没有被监控的程序？
    - 1、可能是启动参数写错了，重新检查一遍启动参数
    - 2、Pinpoint客户端和服务端不在同一台服务器，那么需要指定服务端的主机地址，配置`pinpoint-agent根目录/pinpoint.config`文件中的`profiler.collector.ip`属性值即可。
    ```
    profiler.collector.ip=127.0.0.1
    ```
- 为什么访问了好多次接口，在Web界面上只显示1次？
    - Pinpoint-Agent可以设置采样率，默认采样率是20次采集1次，也就是5%，配置`pinpoint-agent根目录/pinpoint.config`文件中的`profiler.sampling.rate`属性值即可控制采样率，当值为1时表示采样率为100%，即每次访问都会采集。
    ```
    # 1 out of n transactions will be sampled where n is the rate. (20: 5%)
    profiler.sampling.rate=20
    ```

