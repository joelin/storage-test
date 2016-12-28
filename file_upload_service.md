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
