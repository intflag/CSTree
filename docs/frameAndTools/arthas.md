# Java诊断利器-Arthas生产实战
## 生产代码热更新
> 背景：在某次生产环境上出现了Bug，但不能重启服务，只能对生产代码进行热更新。
### 准备工作
- 首先保证开发环境代码与生产一致
- 提前安装阿里巴巴开源的Java诊断工具Arthas，如果生产环境不能访问公网可以采用离线全量安装方式。具体参考：https://alibaba.github.io/arthas/install-detail.html#id2

### 测试环境验证
- 首先找到导致Bug的原因，并且可以在测试环境验证通过。

### 热更新操作（建议先在测试环境操作一次）
- 启动Arthas工具
```bash
java -jar arthas-boot.jar --target-ip 0.0.0.0
```
![](http://images.intflag.com/arthas01-001.jpg)
- 启动成功后将会列出当前运行的所有java进程，`输入准备修改的程序编号`。
![](http://images.intflag.com/arthas01-002.jpg)
- 然后会进入到该程序的Arthas操作命令界面中，然后使用sc命令查询将更新的类的classLoaderHash值，并且`记录一下这个值，之后会用到`。
```bash
sc com.example.demo.arthas.user.UserController
```
![](http://images.intflag.com/arthas01-003.jpg)
- 从你的开发环境将这个类编译后的class文件上传到某个目录下（`可以在当前项目的target\classes下找到编译后的class文件`）
![](http://images.intflag.com/arthas01-004.jpg)
- 使用redefine命令重新加载新的UserController.class文件，同时`指定前面记录的classLoaderHash值`。
```bash
redefine -c 5674cd4d /tmp/UserController.class
```
![](http://images.intflag.com/arthas01-005.jpg)
- 当看到`redefine success`时代表类已经热更新成功，此时就可以对业务进行验证了！

> 由于我的操作不便展示，所以采用了Arthas官方的demo，同时官方特别贴心的为我们搭建了线上练习环境，可以访问这个地址：https://alibaba.github.io/arthas/arthas-tutorials?language=cn
<br><br>其实还有一种方案，可以先使用jad命令将目标类反编译到某个目录下，接着用vim修改，然后用mc命令在线编译，编译后使用redefine命令热更新，但是我在实际操作中遇到了编译失败的问题，所以就采用了第二种方案，
<br><br>最后，一定要详细阅读官方的操作手册
