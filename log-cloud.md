# 云上的日志处理

## 虚拟机写日志过程

云平台存储现状为虚拟机使用外部存储提供云盘功能，目前为nfs对接方式，虚拟机读写需要通过虚拟化层落宿主物理机文件系统，然后nfs客户端写入远端nas。应用日志使用云盘进行写入，日志的收集使用elk在虚拟机内部署agent实现。基于此现状，一份日志数据从产生到归集需要经历如下过程。

1、产生日志并落地
   虚拟机内部应用产生日志通过写入虚拟机数据盘的途径，走虚拟化落在物理机的远端nfs设备上，流量需经过物理机网卡

2、日志采集
   elk在虚拟机内部读取数据盘，实际会传导到虚拟化层，并表现在物理机的nfs客户端从远端nfs设备读数据，流量需经过物理机网卡

3、日志归集
   elk采集的数据上送给elk的服务端，需要通过虚拟网络上送到远端elk服务端，流量需经过物理机网卡

整个过程中，针对同一份数据来说有很多翻倍效应。
1 3倍的物理机网络流程
2 3倍的磁盘操作(2写和1读)
3 2倍的虚拟机数据处理逻辑

之前信总提出使用远端的fs(cephfs 或者nfs、s3fs)来替换掉数据盘写日志的方式，科技结合现有的平台分如下

cephfs 可行，大规模还需验证测试，而且生产稳定性需要厂商加强
nfs ganesha nfs 目前没有大规模的验证，不保证能够满足大规模的日志写入
s3fs 测试下来无法追加写，不能满足要求，日志的写入是不断追加写的过程，对s3fs来说，就是不断覆盖文件。

cephfs 集中写日志需要在虚拟机内有挂载的过程，可以把2、3步骤从虚拟机内部剥离，使用外部服务器从cephfs里读取数据往elk里灌，此方案还需定制开发。
科技建议直接使用业界通用的方案，让日志直接写入远端的elk。业界方案如下

1、产生日志并落地
   虚拟机内部应用产生日志，通过日志组件直接写入远端的消息队列

2、日志采集
   消息处理，分析及转换，直接存入elk服务端存储

此方案在java体系中已支持，如果使用组件室的日志组件对应用是透明的，应用只需升级日志组件即可达到此目的(此点已于组件室确认)；如果没有使用组件室的日志组件，且不能做到无缝升级到log4j2的需要修改代码。此外需要elk系统支持大规模的直接写入即可。






