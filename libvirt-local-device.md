# libvirt虚拟机使用本地裸存储设备

libvirt可以使用本地的磁盘和卷两种裸设备。其它的非本地裸设备的文件、iscsi等这里就不介绍了。


# 使用本地物理磁盘
libvirt 会先定义一个存储池；对于物理磁盘而言，是一个物理磁盘池。对于裸设备`/dev/sdb` 定义的文件如下池文件`sdb_pool.xml`

```
<pool type="disk">
  <name>sdb_pool</name>
  <source>
    <device path='/dev/sdb'/>   
  </source>
  <target>
    <path>/dev</path>
  </target>
</pool>
```

使用virsh定义存储池
> virsh pool-define sdb_pool.xml

创建存储池

> virsh pool-build sdb_pool
> virsh pool-autostart sdb_pool

查看存储池的状态
> virsh pool-list --all

```
Name                 State      Autostart
-------------------------------------------
default              active     yes       
dirpool              active     yes       
Downloads            active     yes       
linxuhua             active     yes              
root                 active     yes       
sdb_pool             active     yes
```

在存储池上创建卷  `sdb1`,最大容量为2G，大小为1G

> virsh vol-create-as --pool sdb_pool --name sdb1 --allocation 1G --capacity 2G

给虚拟机挂载卷,在虚拟机里定义设备名为sda

> virsh attach-disk --domain centos7.0 --source /dev/sdb1 --target sda

总结

libvirt使用本地物理磁盘时，只是把磁盘的分区过程进行封装，最终还是通过qemu使用设备

# 使用本地逻辑卷
libvirt 会先定义一个存储池；对于逻辑卷而言，是一个卷组。对于裸设备`/dev/sdb /dev/sde` 定义的文件如下池文件`lvm_pool.xml`

```
<pool type="logical">
  <name>lvm_pool</name>
  <source>
    <device path='/dev/sdb'/>  
    <device path='/dev/sde'/>  
  </source>
  <target>
    <path>/lvm_pool</path>
  </target>
</pool>
```

使用virsh定义存储池
> virsh pool-define lvm_pool.xml

创建存储池

> virsh pool-build lvm_pool
> virsh pool-autostart lvm_pool

查看存储池的状态
> virsh pool-list --all

```
Name                 State      Autostart
-------------------------------------------
default              active     yes       
dirpool              active     yes       
Downloads            active     yes
lvm_pool             active     yes
linxuhua             active     yes              
root                 active     yes       
sdb_pool             active     yes
```
同时在物理机器上能够看到卷组信息
> vgdisplay
```
VG Name lvm_pool
System ID
format lvm2
```

在存储池上创建卷  `lvm_vol1`,最大容量为2G，大小为1G

> virsh vol-create-as --pool lvm_pool --name lvm_vol1 --allocation 1G --capacity 2G

给虚拟机挂载卷,在虚拟机里定义设备名为`sda`

> virsh attach-disk --domain centos7.0 --source /dev/lvm_pool/lvm_vol1 --target sda

总结

libvirt使用逻辑卷时，只是把lvm的管理封装起来，它可以从裸设备开始，创建pv vg lv；也可以直接从vg开始，最终还是通过qemu使用设备
