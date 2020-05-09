# Java诊断利器-Arthas生产实践
## 1、生产代码热更新
> 背景：在某次生产环境上出现了 Bug，但不能重启服务，只能对生产代码进行热更新。

1）准备工作
- 首先保证开发环境代码与生产一致
- 提前安装阿里巴巴开源的Java诊断工具Arthas，如果生产环境不能访问公网可以采用离线全量安装方式。具体参考：https://alibaba.github.io/arthas/install-detail.html#id2

2）测试环境验证
- 首先找到导致 Bug 的原因，并且可以在测试环境验证通过。

3）热更新操作（建议先在测试环境操作一次）
- 启动 Arthas 工具

```bash
java -jar arthas-boot.jar --target-ip 0.0.0.0
```

![](http://images.intflag.com/arthas01-001.jpg)

- 启动成功后将会列出当前运行的所有 Java 进程，`输入准备修改的程序编号`。

![](http://images.intflag.com/arthas01-002.jpg)

- 然后会进入到该程序的 Arthas 操作命令界面中，然后使用 sc 命令查询将更新的类的 classLoaderHash 值，并且`记录一下这个值，之后会用到`。

```bash
sc com.example.demo.arthas.user.UserController
```

![](http://images.intflag.com/arthas01-003.jpg)

- 从你的开发环境将这个类编译后的 class 文件上传到某个目录下（`可以在当前项目的target\classes下找到编译后的class文件`）

![](http://images.intflag.com/arthas01-004.jpg)

- 使用 redefine 命令重新加载新的 UserController.class 文件，同时`指定前面记录的classLoaderHash值`。

```bash
redefine -c 5674cd4d /tmp/UserController.class
```

![](http://images.intflag.com/arthas01-005.jpg)

- 当看到`redefine success`时代表类已经热更新成功，此时就可以对业务进行验证了！

> 由于我的操作不便展示，所以采用了 Arthas 官方的 demo，同时官方特别贴心的为我们搭建了线上练习环境，可以访问这个地址：https://alibaba.github.io/arthas/arthas-tutorials?language=cn
<br><br>其实还有一种方案，可以先使用jad命令将目标类反编译到某个目录下，接着用 vim 修改，然后用 mc 命令在线编译，编译后使用 redefine 命令热更新，但是我在实际操作中遇到了编译失败的问题，所以就采用了第二种方案，
<br><br>最后，一定要详细阅读官方的操作手册

## 2、生产方法执行耗时分析
> 背景：生产某程序在执行某个操作的时候，耗时30多分钟，测试环境无法复现，要求定位到具体的操作

1）准备工作
- 首先保证开发环境代码与生产一致
- 提前安装阿里巴巴开源的 Java 诊断工具 Arthas，如果生产环境不能访问公网可以采用离线全量安装方式。具体参考：https://alibaba.github.io/arthas/install-detail.html#id2

2）耗时分析
- 启动Arthas工具

```bash
java -jar arthas-boot.jar
```

![](http://images.intflag.com/arthas02-001.jpg)

- 启动成功后将会列出当前运行的所有 java 进程，`输入准备修改的程序编号`。

![](http://images.intflag.com/arthas02-002.jpg)

- 然后会进入到该程序的 Arthas 操作命令界面中，然后使用`trace`命令查询某个方法内部调用路径，并输出方法路径上的每个节点上耗时

```
trace -E com.test.ClassA|org.test.ClassB method1|method2|method3
```

![](http://images.intflag.com/arthas02-003.jpg)

- 手动或者等待该方法执行，即可得到该方法的内部调用路径和方法路径上的每个节点上耗时

![](http://images.intflag.com/arthas02-004.jpg)

- 根据图片上的分析我们可以得知 `initLog` 方法耗费了大量的时间，然后我们就可以专门分析这个方法了
- PS：后续我仔细的分析了这个方法，发现并不是程序逻辑问题导致时间长，是由于方法里面一个对数据库查询的操作导致的，但是单独在生产执行这个查询的 SQL 语句却非常快，最后经过和运维同事一起排查发现，是由于这个任务执行时数据库压力太大导致。 

> 之前用过 Arthas 的代码热更新功能，非常好用，这次又用它来排查程序耗时过长的问题，所以 Arthas 还是很牛掰的，一些常用的操作最好能够掌握。

## 3、生产方法执行数据观测
> 背景：某次在项目提测后收到测试人员 Bug 反馈，前台提交后进行数据回显出现乱码。

1）问题分析

遇到上述这种情况，我们要先分析程序的调用过程，找到问题发生的根本所在，不能急于修改代码。然后，简单分析了一下我的程序，前端不是直接调用后台接口，而是通过一个 API 调用模块转发了一次，所以，我们要定位乱码到底发生在什么时候，此时，你要在代码中各种地方加入日志打印进行观察吗，不不不，效率太低，应该马上祭出 Arthas 大法，用这个工具对调用的方法进行执行时的数据观测。

2）方法执行数据观测

- 启动 Arthas 工具，找到发生问题的 Java 进程，进入该程序的 Arthas 命令界面中。

具体方法在**生产代码热更新**和**生产方法执行耗时分析**中已经介绍过，就不在赘述了，也可以到 [Arthas 官网](https://alibaba.github.io/arthas/index.html)。

- 使用 `watch` 命令对方法进行观测

```
watch com.xxx.service.XXXService getJobApiData "{params,returnObj}" -x 3 -b -s
```

- 命令解释

![](http://images.intflag.com/arthas01.png)

3）分析结果

![](http://images.intflag.com/arthas02.png)

通过分析执行真正操作数据库的方法发现，该方法在接收到调用时参数就已经乱码了。

![](http://images.intflag.com/arthas03.png)

通过分析转发接口调用的方法发现，该方法在接收到前端调用时参数并没有乱码，所以，我们可以断定，问题出现在接口转发时。



