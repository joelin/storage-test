# netapp产品学习

## 产品介绍

### 硬件产品

#### FAS产品组合

![fas](netapp/fas.jpg)


入门级：FAS2520-2554

高端：FAS9000

数据说明:

- 172PB：支持的最大存储容量(HDD/SSD)
- 16TB：支持的最大缓存容量

#### EF系列——纯闪存（可全部支持SSD）

![](netapp/478092351.jpg)


![](netapp/ef-usage.jpg)

EF系列：将FAS硬盘存储全换成SSD，即AFF


#### E系列存储系统——高性能存储（专为SAN环境，性价比）

![](netapp/e-usage.jpg)


#### FlexPod 融合基础架构

![](netapp/cluster.jpg)

运行集群模式 Data ONTAP 的 NetApp FAS
* 统一存储
* 智能数据管理

应用场景:需超长的应用程序正常运行时间；5个9的可用性条件下，达到高可用；保证存储的可以灵活拓展三大特点：
* 简单-安装使用
* 无缝-对接
* 简化-适应性各种环境，提高运行效率，降低电耗、密度和散热成本


### 软件产品

#### Data ONTAP

netapp的存储数据管理系统基于伯克利的BSD unix和其它一些操作系统技术，原始的ontap 只支持NFS，　后面陆续加入了SMB,iSCSI和FC(Fibre Channel)的支持。2006年６月１６日，netapp公司发布了data ontap系统的两个版本,Data ONTAP 7G 和 基于从Spinnaker Networks的网格技术完全重写的Data ONTAP GX.在2010年,这些特性又被合并在一个操作系统(Data ONTAP 8)中.Data ONTAP 7G 被合并进GX的集群平台中.所以Data ONTAP 8 有两个不同的操作模式, 7-mode 和 cluster-mode.


### manila支持


## reference

- http://netapp.github.io/openstack-deploy-ops-guide/liberty/content/ch_executive-summary.html

- https://en.wikipedia.org/wiki/NetApp
