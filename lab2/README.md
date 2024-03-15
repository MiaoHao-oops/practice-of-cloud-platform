# lab2 虚拟机内存Balloon机制实验

## Balloon 机制

在使用虚拟机的过程中，往往会出现 A 机早上内存负载大，B 机中午内存负载大，C 机晚上内存负载大这种情况。如果 3 台虚拟机内存负载最大时要求的最小内存为 2GB，那么如果虚拟机的内存大小无法动态分配，则要求物理机至少有 2GB $\times$ 3 = 6GB 的可用物理内存。但实际上，当一台虚拟机内存负载达到最大时，另外两台虚拟机总共可能只需要很小的内存，因此对物理机的内存要求可以更小，从而运行更多的虚拟机。

Balloon 机制允许动态地压缩/放大正在运行的虚拟机的可用物理内存大小。对于上述场景，可以在早上、中午和晚上分别对 3 台虚拟机的可用物理内存进行调整：早上给 A 机 2GB 的内存，B、C 机均给 1GB 内存，以此类推。这样，运行这 3 台虚拟机最多只需要物理机有 4GB 的可用物理内存就够了。

## 实验步骤

1. 在虚拟机的XML中添加balloon设备，添加如下配置项：

```xml
<devices>
    <memballoon model='virtio'>
        <address type='pci' domain='0x0000' bus='0x00' slot='0x02' function='0x0'/>
    </memballoon>
</devices>
```

> balloon 机制是基于 virtio 实现的，相当于在客户机中增加了一个虚拟外设，用于宿主机和客户机之间进行通信，调整内存大小

2. 在guest中查看virtio_balloon模块是否已经加载：

```sh
[root@localhost ~]# lsmod | grep virtio
virtio_balloon         18015  0 
virtio_pci             22985  0 
virtio_ring            22991  2 virtio_pci,virtio_balloon
virtio                 14959  2 virtio_pci,virtio_balloon
```

其中 `virio_balloon` 即为虚拟 balloon 设备驱动

3. 查看此时虚拟机的可用内存：

```sh
[root@localhost ~]# free -h
              total        used        free      shared  buff/cache   available
Mem:           1.7G        183M        1.4G        8.6M        103M        1.4G
Swap:            0B          0B          0B
```

虚拟机此时的可用总内存为 1.7GB，对应 xml 文件中设置的 `<currentMemory unit='KiB'>2097152</currentMemory>`

4. 在宿主机的 virsh 中修改虚拟机的可用内存（无需关闭虚拟机，注意不要超过 `<memory unit='KiB'>4194304</memory>` 中设置的最大内存）：

```virsh
virsh # setmem centos 4GB
```

5. 再次查看虚拟机的可用内存：

```sh
[root@localhost ~]# free -h
              total        used        free      shared  buff/cache   available
Mem:           3.4G        183M        3.1G        8.6M        104M        3.1G
Swap:            0B          0B          0B
```

可用内存已经变为上面设置的 4GB

6. 在关闭虚拟机后，可以通过 `virsh # setmem <domain> <size>` 修改虚拟机的最大内存
