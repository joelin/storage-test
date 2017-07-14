# openstack后端存储ceph的升级
openstack后端存储 ceph 的升级是一个比较危险的过程，我们基于 openstack L 版部署，对接后端存储为 ceph 的 Hammer .目前想把 ceph 升级到 Jewel 版本，为将来使用更新版本做准备。

## 环境介绍
openstack L 版本，后端的 glance cinder 都使用 ceph 对接，ceph 集群为 3 节点。

### 更新源到 J 版

change repo to j

### 停止一个节点

> /etc/init.d/ceph stop

### 更新软件

> yum install ceph

### 更改目录权限

>chown -R ceph:ceph /var/lib/ceph

>chown -R ceph:ceph /var/log/ceph

>chown -R ceph:ceph /var/run/ceph

### 更改分区权限

>chown -R ceph:ceph /dev/sdb1


### 启动服务

> systemctl start ceph-mon@hostname

> systemctl start ceph-osd@num


### 调整为新版本的ceph配置

>ceph osd set sortbitwise

>ceph osd set require_jewel_osds

>ceph osd crush tunables optimal


## 潜在问题： 
1、单节点停机状态更新软件
2、不停机更新后低版本软件会删除，怎么停机？