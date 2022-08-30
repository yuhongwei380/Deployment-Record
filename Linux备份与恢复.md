## Linux备份与恢复

尝试过的几种方法

1.直接在pve 层去lvreduce ，会导致系统直接故障。【Failed】

2.利用clonezilla 网络备份到新虚拟机（40GB），发现clonezilla 不支持大硬盘 到小硬盘的迁移【Failed】

3.只迁移数据，不迁移系统【系统环境复杂，重建比较麻烦，暂不考虑】

4.xfsdump与dump



### 一、XFS文件系统

测试环境：

| 系统版本   | Centos7.9 |
| ---------- | --------- |
| 文件系统   | XFS       |
| 根目录设备 | /dev/sda1 |
| 磁盘容量   | 80G       |
| 实际占用   | 27G       |

Promox中新加磁盘：/dev/sdb1  [xfs文件系统 容量40G]

#### 1.1 备份原系统

通过nfs挂载磁盘并利用xfsdump 备份原系统到NAS、xfsrestore还原系统到新磁盘。

##### 1.1.1 挂载NFS

```
mkdir /nfs
mount -t nfs  ip://volume1/[path]   /nfs
```

##### 1.1.2 添加新磁盘

添加新磁盘为40G，并在系统内格式化成XFS文件系统，并挂载；

```
fdisk /dev/sdb
***
mkfs.xfs -f /dev/sdb1
mkdir /test
mount /dev/sdb1   /test
```

##### 1.1.3 XFSdump备份原系统到NAS

执行xfsdump后会要求输入一些名字作为还原的标识，此步，就默认按照命令提示的即可。可以选择一致，这样保证不出错。

```
xfsdump -f /nfs/sda.dump   /dev/sda1
```

#### 1.2 XFSrestore还原系统到新磁盘

##### 1.2.1 记录新磁盘信息

```
cd /dev/disk/by-uuid  && ls -l 
记录下新磁盘的UUID
```

##### 1.2.2 还原

```
xfsrestore -f /nfs/sda.dump   /test
```

##### 1.2.3 修改系统启动文件

修改`/test/etc/fstab`   ;把磁盘UUID修改为`sdb1`的UUID

```
UUID="[sdb1 's UUID]"     /     xfs  defaults 0 0
```

##### 1.2.4 关机

#### 1.3 PVE中分离`/dev/sda1`

#### 1.4 Rescue模式

进入`Centos7`的Rescue模式。

##### 1.4.1 重建引导分区

`lsblk` 可以看到`/mnt/sysimage`  ，进入`/dev/sda1`  删除原有 `Boot` 分区的内容（非必选，可直接到重建grub 的步骤）

如果是UEFI分区安装的，有/boot/efi，/boot 请更改为/boot/efi/EFI/centos/

```
cd /mnt/sysimage/boot/
pwd #确保路径是/mnt/sysimage/boot/
rm -rf *
```

##### 1.4.2 安装内核（非必选，可直接到重建grub 的步骤）

```
rpm -ivh /run/install/repo/Packages/kernel-3.10.0-1160.el7.x86_64.rpm --root=/mnt/sysimage/ --force
```

##### 1.4.3 重建grub

```
chroot /mnt/sysimage
grub2-install /dev/sda
grub2-mkconfig -o /boot/grub2/grub.cfg
```

`exit` 重启，`PVE WEBUI`页面中移除该虚拟机的`centos7.iso` ，关机。再到启动项中选择 新盘作为系统启动盘。

重新开机，可以正常进入系统。第一次进入后会涉及到一次重启，重启后 登录，利用df -h可以观察到新盘已经是作为系统盘了，且原来/home下的文件也没有丢失。



------

### 二、EXT4文件系统

步骤和上述类似；仅仅将`xfsdump` 和`xfsrestore` 更改为`dump` 和`restore `  即可。