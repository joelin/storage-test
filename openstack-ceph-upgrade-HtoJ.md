# openstack后端存储ceph的升级
openstack后端存储 ceph 的升级是一个比较危险的过程，我们基于 openstack L 版部署，对接后端存储为 ceph 的 Hammer .目前想把 ceph 升级到 Jewel 版本，为将来使用更新版本做准备。此升级方案只包含块存储服务的升级，不包含对象和文件系统的升级操作。

## 1 单机集群升级

### 1.1 更新源到 J 版

change repo to j

### 1.2 停止一个节点

> /etc/init.d/ceph stop

### 1.3 更新软件

> yum install ceph

### 1.4 更改目录权限

>chown -R ceph:ceph /var/lib/ceph

>chown -R ceph:ceph /var/log/ceph

>chown -R ceph:ceph /var/run/ceph

### 1.5 更改分区权限

需要把ceph所使用的所有分区(包含journal和data分区)的属主调整为ceph用户，从J开始，ceph服务使用ceph 用户启动，并且使用systemd 来管理服务。

>chown -R ceph:ceph /dev/sdb1


### 1.6 启动服务

> systemctl start ceph-mon@hostname

> systemctl start ceph-osd@num


### 1.7 调整为新版本的ceph配置
整个集群升级完成后需要调整集群的参数到新版本，否则会出现集群的异常状态。


```
    cluster 9615d4f2-1e9e-41a0-9b00-f05b1a7aaefb
     health HEALTH_ERR
            20 pgs are stuck inactive for more than 300 seconds
            31 pgs degraded
            21 pgs peering
            20 pgs stale
            20 pgs stuck stale
            52 pgs stuck unclean
            31 pgs undersized
            recovery 1/2 objects degraded (50.000%)
            crush map has legacy tunables (require bobtail, min is firefly)
            no legacy OSD present but 'sortbitwise' flag is not set
            all OSDs are running jewel or later but the 'require_jewel_osds' osdmap flag is not set
     monmap e1: 1 mons at {test1=192.168.56.210:6789/0}
            election epoch 2, quorum 0 test1
     osdmap e33: 3 osds: 3 up, 3 in
      pgmap v49: 72 pgs, 2 pools, 2547 bytes data, 1 objects
            100 MB used, 27514 MB / 27614 MB avail
            1/2 objects degraded (50.000%)
                  31 active+undersized+degraded
                  21 remapped+peering
                  20 stale+active+clean

```


>ceph osd set sortbitwise

>ceph osd set require_jewel_osds

>ceph osd crush tunables optimal


## 2 多机集群升级
openstack L 版本，后端的 glance cinder 都使用 ceph 对接，ceph 集群为 3 节点。参考单节点的验证步骤，我们逐个节点来升级，以下步骤逐个节点操作，以保证集群的正常服务水平。

### 2.1 单节点升级操作

#### 2.1.1 准备工作
更新源指向J版

#### 2.1.2 停止节点服务

> /etc/init.d/ceph stop

#### 2.1.3 更新软件

> yum udpate ceph

如果没法正常更新，比如自编译的特殊版本，需要载旧的软件包，由于H版为特殊版本，没法正常升级到J 版。使用以下步骤，必须保证所有的ceph包都卸载干净。卸载前备份配置文件。
> mv /etc/ceph /etc/ceph.bk
> rpm -e ceph

> yum install ceph

#### 2.1.4 更改目录权限

>chown -R ceph:ceph /var/lib/ceph

>chown -R ceph:ceph /var/log/ceph

如果没有 `/var/run/ceph`目录，需要手工创建

>chown -R ceph:ceph /var/run/ceph


#### 2.1.5 更改分区权限

需要把ceph所使用的所有分区(包含journal和data分区)的属主调整为ceph用户，从J开始，ceph服务使用ceph 用户启动，并且使用systemd 来管理服务。

>chown -R ceph:ceph /dev/sdb1

>chown -R ceph:ceph /dev/sdb2

> ....

#### 2.1.6 启动服务

如果为mon的节点，启动mon服务
> systemctl start ceph-mon@{hostname}

查看状态，正常应该是已加入集群

> systemctl status ceph-mon@{hostname}

如果为osd的节点，启动osd服务
> systemctl start ceph-osd@{osd.num}

查看状态，正常应该是已加入集群
> systemctl status ceph-osd@{osd.num}

#### 2.1.7 确认集群状态

使用 `ceph -s `查看集群状态,此时集群应该会发生recovery，为了保证集群数据的安全性，需要等待recovery结束才能开始下一节点的升级操作。也就是集群的状态为 OK .
> ceph -s 


### 2.2 调整为新版本的ceph配置

整个集群升级完成后需要调整集群的参数到新版本，否则会出现集群的异常状态。参考单节点的此步骤描述

>ceph osd set sortbitwise

>ceph osd set require_jewel_osds

>ceph osd crush tunables optimal

### 2.3 客户端升级
每个对接的服务先升级`ceph`版本，重启其依赖的服务，如果是计算节点，虚拟机必须要重启。

### 2.3.1 cinder对接升级
增加 ceph J 版的repo源
> yum update ceph

如果有特殊原因没法直接更新软件包，需要手工卸载软件包，并安装新的版本。因为librbd会被kvm依赖，所以要忽略依赖强制卸载。

> mv /etc/ceph /etc/ceph.bk

> rpm -e ceph

> yum install ceph

> mv /etc/ceph.bk /etc/ceph

重启 ceph对接的 `cinder-volume` 服务

> systemctl restart openstack-cinder-volume-ceph 


### 2.3.2 glance对接升级
增加 ceph J 版的repo源
> yum update ceph

如果有特殊原因没法直接更新软件包，需要手工卸载软件包，并安装新的版本。因为librbd会被kvm依赖，所以要忽略依赖强制卸载。

> mv /etc/ceph /etc/ceph.bk

> rpm -e ceph

> yum install ceph

> mv /etc/ceph.bk /etc/ceph

重启`glance` 服务

> systemctl restart openstack-glance-api



### 2.3.3 compute对接升级

> yum update ceph

如果有特殊原因没法直接更新软件包，需要手工卸载软件包，并安装新的版本。因为librbd会被kvm依赖，所以要忽略依赖强制卸载。

> mv /etc/ceph /etc/ceph.bk

> rpm -e ceph

> yum install ceph

> mv /etc/ceph.bk /etc/ceph

重启`libvirt` `nova-compute` 服务，使其使用最新版本的`librbd` 模块。

> systemctl restart libvirtd

> systemctl restart openstack-nova-compute

## 3 影响性总结及注意事项

* 1、升级过程不能对 `openstack` 集群有任何的操作，包含虚拟机、云硬盘等
* 2、`ceph` 集群的逐步升级过程不会影响虚拟机内部正在运行不中断读写进程（例如测试FIO应用），但是新打开一个文件则不行，也就是会影响虚拟机的正常运行。
* 3、 升级完成后，重新启动虚拟机则一切恢复正常
* 4、整个过程大概需要30分钟时间，时间具体视集群的大小自行评估


## 4 参考
- http://docs.ceph.com/docs/master/release-notes/#id597