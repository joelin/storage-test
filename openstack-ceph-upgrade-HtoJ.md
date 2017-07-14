# openstack后端存储ceph的升级
openstack后端存储 ceph 的升级是一个比较危险的过程，我们基于 openstack L 版部署，对接后端存储为 ceph 的 Hammer .目前想把 ceph 升级到 Jewel 版本，为将来使用更新版本做准备。

## 环境介绍
openstack L 版本，后端的 glance cinder 都使用 ceph 对接，ceph 集群为 3 节点。


no need run as ceph
chown folder 
chown disk



ceph osd set sortbitwise



ceph osd set require_jewel_osds

ceph osd crush tunables optimal


潜在问题： 
1、单节点停机状态更新软件
2、不停机更新后低版本软件会删除，怎么停机？