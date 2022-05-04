# ZOOKEEPER

Zookeeper 所选用版本为 zookeeper-3.5.7.tar.gz。

## 一、集群部署规划

|           | hadoop102 | hadoop103 | hadoop104 |
| --------- | --------- | --------- | --------- |
| Zookeeper | Zookeeper | Zookeeper | Zookeeper |

## 二、准备工作——解压安装

#### （1）解压 Zookeeper 安装包到 `/opt/module/` 目录下。

`tar -zxvf zookeeper-3.5.7.tar.gz -C /opt/module/`

#### （2）修改 /opt/module/apache-zookeeper-3.5.7-bin 名称为zookeeper-3.5.7

`mv apache-zookeeper-3.5.7-bin/ zookeeper-3.5.7`

#### (3)同步/opt/module/zookeeper-3.5.7目录内容到hadoop103、hadoop104

`xsync zookeeper-3.5.7/`

## 三、配置集群

### 1、配置服务器编号

#### （1）在/opt/module/zookeeper-3.5.7/目录下创建zkData

`mkdir zkData`

#### （2）在 /opt/module/zookeeper-3.5.7/zkData 目录下创建一个 myid 的文件

`vim myid`

注：添加 myid 文件，注意一定要在linux里面创建，在notepad++ 里面很可能乱码

在文件汇总添加与 server 对应的编号：

`2`

#### （3）拷贝配置好的 zookeeper 到其他机器上

`[quakewang@hadoop102 zkData]$ xsync myid`

并分别在 hadoop103 和 hadoop104 上修改 myid 文件中的内容为 3 和 4。

### 2、配置 zoo.cfg 文件

#### （1）重命名 /opt/module/zookeeper-3.5.7/conf 这个目录下的 zoo_sample.cfg 为 zoo.cfg

`[quakewang@hadoop102 conf]$ mv zoo_sample.cfg zoo.cfg`

#### （2）编辑 zoo.cfg 文件

`vim zoo.cfg`

修改数据存储路径配置

`dataDir=/opt/module/zookeeper-3.5.7/zkData`

增加如下配置：

```cfg
#######################cluster##########################
server.2=hadoop102:2888:3888
server.3=hadoop103:2888:3888
server.4=hadoop104:2888:3888
```

#### （3）同步 zoo.cfg 配置文件

`[quakewang@hadoop102 conf]$ xsync zoo.cfg`

#### （4）配置参数解读

`server.A=B:C:D`

**A** 是一个数字，表示这个是第几号服务器；

集群模式下配置一个文件 myid，这个文件在 dataDir 目录下，这个文件里面有一个数据就是 A 的值，Zookeeper 启动时读取此文件，拿到里面的数据与 zoo.cfg 里面的配置信息比较从而判断到底是哪个 server。

**B **是这个服务器的地址；

**C **是这个服务器 Follower 与集群中的 Leader 服务器交换信息的端口；

**D** 是万一集群中的 Leader 服务器挂了，需要一个端口来重新进行选举，选出一个新的 Leader，而这个端口就是用来执行选举时服务器相互通信的端口。

## 四、集群操作

（1）分别启动 Zookeeper

```bash
[quakewang@hadoop102 zookeeper-3.5.7]$ bin/zkServer.sh start
[quakewang@hadoop103 zookeeper-3.5.7]$ bin/zkServer.sh start
[quakewang@hadoop104 zookeeper-3.5.7]$ bin/zkServer.sh start
```

（2）查看状态

```bash
[quakewang@hadoop102 zookeeper-3.5.7]# bin/zkServer.sh status
JMX enabled by default
Using config: /opt/module/zookeeper-3.5.7/bin/../conf/zoo.cfg
Mode: follower
[quakewang@hadoop103 zookeeper-3.5.7]# bin/zkServer.sh status
JMX enabled by default
Using config: /opt/module/zookeeper-3.5.7/bin/../conf/zoo.cfg
Mode: leader
[quakewang@hadoop104 zookeeper-3.4.5]# bin/zkServer.sh status
JMX enabled by default
Using config: /opt/module/zookeeper-3.5.7/bin/../conf/zoo.cfg
Mode: follower
```

## 五、客服端命令行操作

启动客户端

`[quakewang@hadoop103 zookeeper-3.5.7]$ bin/zkCli.sh`

| 命令基本语法 | 功能描述                                                     |
| ------------ | ------------------------------------------------------------ |
| help         | 显示㞋操作命令                                               |
| ls path      | 使用 ls 命令来查看当前znode的子节点<br />-w 监听子节点变化  <br />-s  附加次级信息 |
| create       | 普通创建<br />-s 含有序列<br />-e 临时（重启或者超时消失）   |
| get path     | 获得节点的值<br />-w 监听节点内容变化<br />-s 附加次级信息   |
| set          | 设置节点的具体值                                             |
| status       | 查看节点状态                                                 |
| delete       | 删除节点                                                     |
| deleteall    | 递归删除节点                                                 |

## 六、Zookeeper 集群启动脚本

```shell
#!/bin/bash

case $1 in
"start"){
	for i in hadoop102 hadoop103 hadoop104
	do
        echo ---------- zookeeper $i 启动 ------------
		ssh $i "/opt/module/zookeeper-3.5.7/bin/zkServer.sh start"
	done
};;
"stop"){
	for i in hadoop102 hadoop103 hadoop104
	do
        echo ---------- zookeeper $i 停止 ------------    
		ssh $i "/opt/module/zookeeper-3.5.7/bin/zkServer.sh stop"
	done
};;
"status"){
	for i in hadoop102 hadoop103 hadoop104
	do
        echo ---------- zookeeper $i 状态 ------------    
		ssh $i "/opt/module/zookeeper-3.5.7/bin/zkServer.sh status"
	done
};;
esac
```

增加脚本执行权限

`chmod u+x zk.sh`

Zookeeper 集群启动脚本

`zk.sh start`

Zookeeper 集群停止脚本

`zk.sh stop`