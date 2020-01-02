# APM监控工具-Pinpoint实战
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
- https://www.cnblogs.com/yyhh/p/6106472.html
- https://www.jianshu.com/p/a8482f01af4a（推荐）
