# lab0-1 安装 qemu-kvm/libvirt 虚拟化软件 & 虚拟机基本操作

## lab 0 安装 qemu-kvm/libvirt 虚拟化软件

1. 查看CPU是否支持虚拟化：

    ```sh
    root@server-hyjmc2qr:~# LC_ALL=C lscpu | grep Virtualization
    Virtualization:                  VT-x
    Virtualization type:             full
    ```

    > `LC_ALL=C` 统一设置 `LC_*` 环境变量 for Computer，保证命令输出格式统一

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

## lab 1 虚拟机基本操作

1. 基于已有的 [cirros 镜像](./cirros-0.3.6-x86_64-disk.img)，创建 `.xml` 文件 [cirros.xml](./cirros.xml)

    > `.xml` 文件采用标签化语法描述，在 `.xml` 文件中，可以指定 domain 的名称，虚拟机架构、CPU、内存，模拟器配置等信息。

2. 通过 `.xml` 文件定义 domain：

    ```sh
    root@server-hyjmc2qr:~# virsh define ./cirros.xml 
    Domain cirros defined from ./cirros.xml
    ```

3. 通过 `list` 命令查看已经定义的 domain：

    ```sh
    root@server-hyjmc2qr:~# virsh list --all
    Id   Name     State
    -------------------------
    -    cirros   shut off
    ```

4. 启动定义的 domain：

    ```sh
    root@server-hyjmc2qr:~# virsh start cirros
    Domain cirros started
    ```

5. 连接启动虚拟机的串口：

    ```sh
    root@server-hyjmc2qr:~# virsh console cirros
    Connected to domain cirros
    Escape character is ^]
    ...
      ____               ____  ____
     / __/ __ ____ ____ / __ \/ __/
    / /__ / // __// __// /_/ /\ \ 
    \___//_//_/  /_/   \____/___/ 
    http://cirros-cloud.net


    login as 'cirros' user. default password: 'cubswin:)'. use 'sudo' for root.
    cirros login: 
    ```

    可以通过快捷键 `^[` 退出控制台。

> 注：镜像文件不要置于任何所属用户/组为 root 的文件夹下，否则在 `virsh start cirros` 时将出现权限错误。
