# lab0-1 安装 qemu-kvm/libvirt 虚拟化软件 & 虚拟机基本操作

## lab 0

1. 查看CPU是否支持虚拟化：

```sh
LC_ALL=C lscpu | grep Virtualization
```

> `LC_ALL=C` 统一设置 `LC_*` 环境变量 for Computer，保证命令输出格式统一

运行结果：

```sh
root@server-hyjmc2qr:~# LC_ALL=C lscpu | grep Virtualization
Virtualization:                  VT-x
Virtualization type:             full
```

以上输出说明服务器支持 VT-x 虚拟化扩展。

2. 安装相关软件：

```sh
apt install qemu qemu-kvm virt-manager bridge-utils -y
```

3. 启动libvirt服务：

```sh
systemctl enable libvirtd.service && sudo systemctl start libvirtd.service
```

4. 测试安装是否成功：

    - 命令行执行 `virsh` 命令，如果出现“`virsh #`”提示符，说明 libvirt 安装成功；
    - 命令行执行 `which qemu-system-x86_64` 命令，如果出现“`/usr/bin/qemu-system-x86_64`”说明 qemu-kvm 虚拟机安装成功。

> `virsh` 是一个交互式的虚拟机管理程序，我们可以通过 `.xml` 文件描述一个 domain，然后将其通过 `virsh define` 定义，并通过 `virsh start` 启动虚拟机。