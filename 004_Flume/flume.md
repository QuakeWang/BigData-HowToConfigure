# Flume

按照采集通道规划，需在hadoop102，hadoop103，hadoop104 三台节点分别部署一个Flume。**可参照以下步骤先在 hadoop102** 安装，然后再进行分发

## 一、准备工作

#### 安装地址

-    Flume官网地址：http://flume.apache.org/

-    文档查看地址：http://flume.apache.org/FlumeUserGuide.html

-    下载地址：http://archive.apache.org/dist/flume/

## 二、安装部署

（1）将 apache-flume-1.9.0-bin.tar.gz 上传到服务器的 /opt/software 目录下

（2）解压 apache-flume-1.9.0-bin.tar.gz 到 /opt/module/ 目录下

`tar -zxf /opt/software/apache-flume-1.9.0-bin.tar.gz -C /opt/module/`

（3）修改 apache-flume-1.9.0-bin 的名称为 flume

`[quakewang@hadoop102 module]$ mv /opt/module/apache-flume-1.9.0-bin /opt/module/flume`

（4）将 lib 文件夹下的 guava-11.0.2.jar 删除以兼容 Hadoop 3.1.3

`rm /opt/module/flume/lib/guava-11.0.2.jar`

**注意：删除guava-11.0.2.jar的服务器节点，一定要配置hadoop环境变量。否则会报如下异常。**

```log
Caused by: java.lang.ClassNotFoundException: com.google.common.collect.Lists
        at java.net.URLClassLoader.findClass(URLClassLoader.java:382)
        at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
        at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:349)
        at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
        ... 1 more
```

（5）修改 conf 目录下的 log4j.properties 配置文件，配置日志文件路径。

```bash
[quakewang@hadoop102 conf]$ vim log4j.properties

flume.log.dir=/opt/module/flume/logs
```

