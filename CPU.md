# 物理机CPU学习

## CPU  

* 物理CPU

真实CPU，接口插上去的(几路几路)

* 核心数

多核概念，也就是说的一个物理CPU集成了几个核心

* 超线程(hyper-threading)

开启超线程后，一个核心可以当2个逻辑CPU使用，也就是有2个processor，计算机的逻辑计算单元就是这里的processor概念

* numa

NUMA通过提供分离的存储器给各个处理器，避免当多个处理器访问同一个存储器产生的性能损失来试图解决这个问题。对于涉及到分散的数据的应用（在服务器和类似于服务器的应用中很常见），NUMA可以通过一个共享的存储器提高性能至n倍,而n大约是处理器（或者分离的存储器）的个数。
当然，不是所有数据都局限于一个任务，所以多个处理器可能需要同一个数据。为了处理这种情况，NUMA系统包含了附加的软件或者硬件来移动不同存储器的数据。这个操作降低了对应于这些存储器的处理器的性能，所以总体的速度提升受制于运行任务的特点。

* refer

https://en.wikipedia.org/wiki/Central_processing_unit
https://en.wikipedia.org/wiki/Non-uniform_memory_access
https://www.ibm.com/developerworks/cn/linux/l-numa/index.html

## libvirt

* vcpu pin

https://libvirt.org/formatdomain.html#elementsCPUTuning

## openstack

* specs

https://specs.openstack.org/openstack/nova-specs/specs/juno/approved/virt-driver-cpu-pinning.html

* blueprints

https://blueprints.launchpad.net/nova/+spec/virt-driver-numa-placement

