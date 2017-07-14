# openstack后端存储ceph的升级
openstack后端存储 ceph 的升级是一个比较危险的过程，我们基于 openstack L 版部署，对接后端存储为 ceph 的 Hammer .目前想把 ceph 升级到 Jewel 版本，为将来使用更新版本做准备。此升级方案只包含块存储服务的升级，不包含对象和文件系统的升级操作。

## 1.单机集群升级

### 1.1更新源到 J 版

change repo to j

### 1.2停止一个节点

> /etc/init.d/ceph stop

### 1.3更新软件

> yum install ceph

### 1.4更改目录权限

>chown -R ceph:ceph /var/lib/ceph

>chown -R ceph:ceph /var/log/ceph

>chown -R ceph:ceph /var/run/ceph

### 1.5更改分区权限

需要把ceph所使用的所有分区(包含journal和data分区)的属主调整为ceph用户，从J开始，ceph服务使用ceph 用户启动，并且使用systemd 来管理服务。

>chown -R ceph:ceph /dev/sdb1


### 1.6启动服务

> systemctl start ceph-mon@hostname

> systemctl start ceph-osd@num


### 1.7调整为新版本的ceph配置
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


## 2多机集群升级
openstack L 版本，后端的 glance cinder 都使用 ceph 对接，ceph 集群为 3 节点。参考单节点的验证步骤，我们逐个节点来升级，以下步骤逐个节点操作，以保证集群的正常服务水平。

### 2.1单节点升级操作

#### 2.1.1准备工作
更新源指向J版

#### 2.1.2停止

> /etc/init.d/ceph stop

#### 2.1.3更新软件

> yum udpate ceph

如果没法正常更新，比如自编译的特殊版本，需要载旧的软件包，由于H版为特殊版本，没法正常升级到J 版。使用以下步骤，必须保证所有的ceph包都卸载干净。卸载前备份配置文件。
> mv /etc/ceph /etc/ceph.bk
> rpm -e ceph

> yum install ceph

#### 2.1.4更改目录权限

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

### 2.2 调整为新版本的ceph配置

整个集群升级完成后需要调整集群的参数到新版本，否则会出现集群的异常状态。参考单节点的此步骤描述

>ceph osd set sortbitwise

>ceph osd set require_jewel_osds

>ceph osd crush tunables optimal

### 2.3 客户端升级

### 2.3.1 cinder对接升级

### 2.3.2 glance对接升级

### 2.3.3 compute对接升级