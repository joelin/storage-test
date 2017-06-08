# 磁盘格式化工具

根据文件系统的材料，我们知道磁盘分区可以分为 GPT(GUID Partition Table)、MBR(Master Boot Record).我们通常使用分区工具把其规则数据写入磁盘对磁盘进行分区。这里主要介绍几种分区工具并作为备忘。

* 传统的fdsik工具

   命令行工具，交互式的操作，包含(`fdisk`,`cfdisk`,`sfdisk`)，可以处理 MBR 和 其它特殊分区表，但是不能处理 GPT

* GNU Parted(libparted)

   它提供一个库 (libparted) 和命令行工具 (parted)来进行分区操作，可以基于这个库开发图形界面。

* GPT fdisk 
   
   GPT 工具根据 fdisk 进行建模，它可以处理 GPT 分区的磁盘，主要包含下面三个工具

   * gdisk  
     
     与 `fdisk` 类似的一种交互式命令行操作工具

   * cgdisk
     
     纯命令行工具，可以编写在脚本中使用，不需要交互式的操作
   * sgdisk
     
     与 `cfdisk` 类似的一种交互式命令行操作工具

本文不介绍交互式工具，着重介绍命令行工具 `sgdisk`,因为它使用方便灵活。

## sgdisk

### 命令行参数

`-a,--set-alignment=value`

设置扇区对齐倍数, GPT 将分区的开始与此值的倍数的扇区对齐, 这在新格式化的磁盘上默认为2048。此对齐值对于获得最佳性能, 需要使用西数高级格式和具有较大物理逻辑扇区的磁盘, 做 raid 的或者 SSD 磁盘。

`-A, --attributes=list|[partnum:show|or|nand|xor|=|set|clear|toggle|get[:bitnum|hexbitmask]]`

查看或设置分区属性

`-b,--backup=file`

导出分区表到文件

`-c, --change-name=partnum:name`

改分区名字

`-C, --recompute-chs`

在保护或混合 MBR 分区中重新计算 CHS(磁盘柱面扇区信息) 值

`-d, --delete=partnum`

删除分区，后面为分区号

`-D, --display-alignment`

查看分区扇区对齐倍数信息，即使用 `-a` 设置的参数

`e, --move-second-header`

将备份 gpt 数据结构移动到磁盘的末尾.

`-E, --end-of-largest`

显示磁盘结束为止的最大可用扇区的扇区编号


`-f, --first-in-largest`

显示磁盘起始位置的最大可用扇区的扇区编号

`-F, --first-aligned-in-largest`

类似于 `-f (-first-in-largest)`, 返回正确应用了扇区对齐的扇区编号。

`-g, --mbrtogpt`

把 MBR 的磁盘分区 转换为 GPT 的磁盘分区

`-m, --gpttombr`

把 GPT 的磁盘分区转换为 MBR 

`-T, --transform-bsd=partnum`

把 BSD 的磁盘分区 转换为 GPT 的磁盘分区

`-G, --randomize-guids`

随机分配磁盘和所有分区的唯一 guid (但不是它们的分区类型代码 guid)

`-h, --hybrid` 

创建一个混合的 MBR 分区磁盘

`-i, --info=partnum`

查看分区的详细信息

`-l, --load-backup=file`

从备份的分区表数据恢复分区信息

`-L, --list-types`

查看所有分区类型，与 `fdisk` 里面的 `l` 操作一样

`-n, --new=partnum:start:end`

新建分区。这里可以灵活指定自己的分区分割，start 和 end  可以是扇区编号，也可以是磁盘大小。end 可以省掉,省掉就代表到最后，以下给几个示例并进行解释。注意，这里sgdisk 默认有对齐倍数，为2048.所以收分区的起点都为 2048

  * `sgdisk -n 1:0:+5g  -t 1:8300 -p /dev/sdb`
      
      从硬盘起点开始创建一个 5G 的1号分区，分区类型编码为 8300
  
  * `sgdisk -n 2:10487807:+2g  -t 2:8300 -p /dev/sdb`
     
     从硬盘扇区号 10487807 开始创建一个 2G 的2号分区，分区类型编码为 8300

   * `sgdisk -n 2:10487807  -t 2:8300 -p /dev/sdb`
     
     从硬盘扇区号 10487807 开始,把之后所有容量都分配给心创建的2号分区，分区类型编码为 8300

  * `sgdisk -n 1:0:-2g  -t 1:8300 -p /dev/sdb`
     
     为硬盘保留2G的容量，其它全部分配给新创建的 1 号分区

`-N, --largest-new=num`

这个命令等同于 `sgdisk -n 2:10487807` ,把磁盘剩余未分配的空间全部分配给新创建的分区2.

`-o, --clear`

清空磁盘所有分区的数据

`-p, --print`

打印分区信息，可以在每次操作后加上此参数，输出变更后的分区信息

`-P, --pretend`

对磁盘的操作不写入磁盘，仅保留在内存

`-r, --transpose`

交换两个分区，这个操作没啥意义，前提条件要保证至少有一个分区为空。

`-R, --replicate=second_device_filename`

这个操作相当于把一个磁盘的分区完整的 clone 到另外一个磁盘，包含所有信息，包括 GUID，如果想重新分配GUID，可以加上 `-G`

`-s, --sort`

对分区项进行排序，不常用。

`-t, --typecode=partnum:{hexcode|GUID}`

更改分区的类型代码。您可以使用两个字节的十六进制数 (如前面所述) 或完全指定的 guid 值 (如 EBD0A0A2-B9E5-4433-87C0-68B6B72699C7) 输入类型代码，比如使用uuidgen 生成的值。


`-u, --partition-guid=partnum:guid`

设置分区的唯一GUID。

`-U, --disk-guid=guid`

设置磁盘的唯一GUID。

`-z, --zap`

仅销毁磁盘上 GPT 的数据结构，保留 MBR 等信息。

`-Z, --zap-all`

销毁磁盘上 所有的数据结构，包含 GPT MBR 。


## reference

- https://en.wikipedia.org/wiki/Disk_partitioning
- https://en.wikipedia.org/wiki/Master_boot_record
- https://en.wikipedia.org/wiki/GUID_Partition_Table
- http://www.rodsbooks.com/gdisk/sgdisk-walkthrough.html
- https://linux.die.net/man/8/sgdisk
- https://en.wikipedia.org/wiki/Fdisk






