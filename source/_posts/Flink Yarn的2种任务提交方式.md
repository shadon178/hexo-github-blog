---
title: Flink Yarn的2种任务提交方式
tags: flink yarn
categories: flink
---

# Flink Yarn的2种任务提交方式

## Pre-Job模式介绍
每次使用flink run运行任务的时候，Yarn都会重新申请Flink集群资源（JobManager和TaskManager），任务执行完成之后，所申请的Flink集群资源就会释放，所申请的Yarn资源是独享的，不与其他任务分享资源。

### 运行命令
``` shell
./bin/flink run -m yarn-cluster -yn 3 -ys 12 -p 4 -yjm 1024m -ytm 4096m ./examples/batch/WordCount.jar
```
参数解读：
	-p 并行度
	-yn Task Managers数量
	-ys 每个TaskManager的Slot数量
	-yjm 每个JobManager内存 (default: MB)
	-ytm 每个TaskManager内存 (default: MB)


## Session模式介绍
需要先在yarn上先分配一个flink集群，后续所有任务都共享这个Flink集群上的资源，该Flink不会因为任务的结束而终止。

### 先向Yarn申请Flink所需资源
flink客户端目录下，执行如下命令：
``` shell
bin/yarn-session.sh -jm 1024m -tm 4096m -n 4 -s 8 -na hdq-yarn
```
参数含义：
	-jm jobmanager的内存大小
	-tm taskManager的内存大小
	-n taskManager个个数
	-s 每个taskManager中slot的个数

执行完成之后会输出如下日志：
``` 
Flink JobManager is now running on 172-16-122-56:9101 with leader id 00000000-0000-0000-0000-000000000000.
JobManager Web Interface: http://172-16-122-56:9101
```
运行完成后，Yarn的集群上会有一个常驻任务。
此时，Flink集群的资源都已经申请完毕。
这里需要记住JobManager的ip和端口：172-16-122-56:9101，等会运行Flink任务的时候需要修改这里的配置。

### 运行Flink程序
运行Flink任务之前需要修改Flink客户端下的配置文件：conf/flink-conf.yaml
分别修改jobmanager.rpc.address和rest.port，对应第二步中的172-16-122-56和9101。
``` 
jobmanager.rpc.address: 172-16-122-56
rest.port: 9101
```
修改完成之后即可运行Flink任务：
``` 
/flink/bin/flink run  
-C file:/plugins/oraclereader/flinkx-oracle-reader.jar 
-C file:/plugins/mysqlwriter/flinkx-mysql-writer.jar 
-C file:/plugins/common/flinkx-rdb.jar 
-C file:/plugins/common/flinkx-rdb-2.0.0.jar 
-C file:/plugins/common/flink-table_2.11-1.7.2.jar /plugins/flinkx.jar  
-job fx_2065.json 
-pluginRoot /plugins 
-jobid 2065
```
此时，flink会自动将任务提交到我们申请的Flink集群上进行运行。

## 注意事项
如果程序依赖第三方jar，通过-C传参的方式进行依赖，那么整个Yarn集群都要有jar文件。
其中-C所指定的所有jar文件，在整个Yarn集群的机器上都必须存在，否则运行会失败。不支持hdfs共享存储，支持ftp等其他协议。


## 总结
- Pre-Job模式: 运行时需要会自动申请Yarn资源，申请完成后才能运行任务，并且所申请的资源是该任务独享的，运行完成后资源会自动释放；适合资源消耗比较大的情况。
- Session模式: 运行之前需要在Yarn上先申请好资源才能提交任务，所有任务会共享资源，适合小任务运行。