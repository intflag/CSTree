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