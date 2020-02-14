# 工作中常用的Linux命令
## 常用Linux命令一览表
|功能|命令|说明|
|:----|:----|:----|
|时间逆序查看|ls -lrt||
|推送文件到目标主机|scp 文件路径 主机用户名@IP地址:存放路径|例：scp /opt/soft/nginx-0.5.38.tar.gz root@10.10.10.10:/opt/soft/|
|从目标主机拉取文件|scp 主机用户名@IP地址:文件路径 存放路径|例：scp root@10.10.10.10:/opt/soft/test.xml ./|
|查看进程|ps -ef \| grep 程序名称|例：ps -ef \| grep HelloApplication|
|查看端口占用情况|netstat -nltp|可以在后面使用grep命令进行过滤|
|查看与目标主机端口连通情况|telnet 主机IP 端口|例：telnet 127.0.0.1 8080|
|解压tar包|tar -zxvf xxx.tar.gz -C 保存路径|.tar文件不用加z命令|
|建立多级目录|mkdir -p /opt/m1/m2/m3|当目录不存在时会建立| 
|查找文件|find 查找路径 文件名称|例：find . ".jar" \| xargs grep "CodeManager" 查找某个类在哪个jar包中|

## 实用小技巧
### 1、快速修改tomcat端口
```
sed -i 's/port="8005"/port="18005"/g' server.xml
sed -i 's/port="8080"/port="18080"/g' server.xml
sed -i 's/port="8443"/port="18443"/g' server.xml
sed -i 's/port="8009"/port="18009"/g' server.xml
sed -i 's/redirectPort="8443"/redirectPort="18443"/g' server.xml
```
### 2、Linux中没有`ll`命令
```
sudo vi ~/.bashrc
加入
alias ll='ls -lrt'
然后刷新配置
source  ~/.bashrc
```
### 3、Linux下批量杀掉筛选进程
```
例如批量删掉flume指定了myconf/.*properties下配置文件的进程
ps -ef|grep flume|grep -E "myconf/.*properties"|awk '{print $2}'|xargs kill -9
```
### 4、设置DNS
```
vi /etc/resolv.conf
加入或修改
nameserver 114.114.114.114
```
### 5、Linux下使用timedatectl命令操作时间时区
```
# 显示当前时间信息
timedatectl status
# 显示所有可用时区
timedatectl list-timezones
# 设置时区
timedatectl set-timezone "Asia/Shanghai"
# 设置日期和时间
timedatectl set-time '2020-02-14 12:13:24'
```
### 6、JAVA程序快速启动停止脚本
```
# 启动
#!/bin/bash
nohup java -classpath './lib/*:./conf/' com.intflag.TestApplication >run.log 2>&1 &

# 停止
#!/bin/bash
ID=`ps -ef | grep TestApplication | grep java | awk '{print $2}'`
echo ${ID}
kill -9 ${ID}

# 重启
#!/bin/bash
ID=`ps -ef | grep TestApplication | grep java | awk '{print $2}'`
echo ${ID}
kill -9 ${ID}
nohup java -classpath './lib/*:./conf/' com.intflag.TestApplication >run.log 2>&1 &
```