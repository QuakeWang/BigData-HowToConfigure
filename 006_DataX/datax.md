# DataX Configuration

1）下载 DataX 安装包并上传到 hadoop102 的 /opt/software

下载地址：http://datax-opensource.oss-cn-hangzhou.aliyuncs.com/datax.tar.gz

2）解压 datax.tar.gz 到 /opt/module

`[quakewang@hadoop102 software]$ tar -zxvf datax.tar.gz -C /opt/module/`

3）自检，执行如下命令

`[quakewang@hadoop102 ~]# python /opt/module/datax/bin/datax.py /opt/module/datax/job/job.json`

如出现如下内容，则表明安装成功

```shell
2022-05-04 20:58:08.928 [job-0] INFO  JobContainer -
任务启动时刻                    : 2022-05-04 20:57:58
任务结束时刻                    : 2022-05-04 20:58:08
任务总计耗时                    :                 10s
任务平均流量                    :          253.91KB/s
记录写入速度                    :          10000rec/s
读出记录总数                    :              100000
读写失败总数                    :                   0
```

