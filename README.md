## Why To Do?

### 1、我为什么要搞这个，能做什么

```
	家里的旧手机实在是不知道要做些什么用处了，对于有一颗折腾心理的人来说，不干点啥说不过去。让他运行起来为我工作吧，毕竟废品才50块，真心不值当卖掉
```

## How To Do?

### 1、先决条件

确保安装**qemu-system-aarch64**和**qemu-image**，推荐4.0以上版本，安装过程参见[QEMU官网](https://www.qemu.org/download/ "下载安装文件")

```
qemu-system-aarch64 -version

#正常情况下，你将看到返回下面这些狗东西，说明qemu-system-aarch64你已经正确安装了
QEMU emulator version 4.2.0 (v4.2.0-11797-g2890edc853-dirty)
Copyright (c) 2003-2019 Fabrice Bellard and the QEMU Project developers

qemu-img -V

#正常情况下，你将看到返回下面这些狗东西，说明qemu-img你已经正确安装了
qemu-img version 4.2.0 (v4.2.0-11797-g2890edc853-dirty)
Copyright (c) 2003-2019 Fabrice Bellard and the QEMU Project developers
```

### 2、下载系统镜像和所需UEFI引导文件

1. 下载Ubuntu或Debian的arm64(aarch64)的iso文件，建议下载最新版的系统，不多赘述，这一步要是都搞不定，就别往下看了

2. 从[Linux应用中心](https://pkgs.org/search/?q=qemu-efi-aarch64 "qemu-efi-aarch64")搜索**qemu-efi-aarch64**，下载对应发行版对应版本对的deb文件，用7zip等文件解压，拷贝出**QEMU_EFI.fd**文件

### 3、创建磁盘并开始安装

```s
#创建格式为qcow2的文件名为debian.qcow2容量为15GB的磁盘
qemu-img create -f qcow2 debian.qcow2 15G

#on Windows cmd
qemu-system-aarch64 ^
    -M virt -m 3G -cpu cortex-a72 -smp 2 ^
    -bios QEMU_EFI.fd ^
    -drive id=cdrom,media=cdrom,if=none,file=debian-10.4.0-arm64-xfce.iso ^
    -device virtio-scsi-device -device scsi-cd,drive=cdrom ^
    -drive id=hd0,media=disk,if=none,file=debian.qcow2 ^
    -device virtio-scsi-pci -device scsi-hd,drive=hd0 ^
    -nic user,model=virtio-net-pci,hostfwd=tcp::2222-:22,hostfwd=tcp::6900-:5900 ^
    -monitor stdio
    
 #on Linux terminal
 qemu-system-aarch64 \
    -M virt -m 3G -cpu cortex-a72 -smp 2 \
    -bios QEMU_EFI.fd \
    -drive id=cdrom,media=cdrom,if=none,file=debian-10.4.0-arm64-xfce.iso \
    -device virtio-scsi-device -device scsi-cd,drive=cdrom \
    -drive id=hd0,media=disk,if=none,file=debian.qcow2 \
    -device virtio-scsi-pci -device scsi-hd,drive=hd0 \
    -nic user,model=virtio-net-pci,hostfwd=tcp::2222-:22,hostfwd=tcp::6900-:5900 \
    -nographic
    
#注意事项
-monitor stdio是默认使用serial0控制台，在windows下请使用此项,如在linux下，请替换为-nographic，因为在Linux中存在键盘方向键无法使用的问题，十分不便

如需要安装ubuntu-20.04-server-live-arm64，只需替换镜像文件和磁盘文件路径即可，另外，ubuntu 20.04及更新版本在Windows下安装会出现问题，建议在Linux下进行
    
```

参数说明

| 参数                                                         | 值                                            | 含义                                                         |
| ------------------------------------------------------------ | --------------------------------------------- | ------------------------------------------------------------ |
| -M                                                           | virt                                          | 模拟的机器类型                                               |
| -m                                                           | 3G                                            | 3GB RAM，还可以写3072或3072M                                 |
| -cpu                                                         | cortex-a72                                    | 模拟的CPU类型，cortex-a72是ARMv8架构比较新的，性能比较强，也可以使用A57或A53 |
| -smp                                                         | 2                                             | 模拟的CPU核心数                                              |
| -bios                                                        | QEMU_EFI.fd                                   | uefi引导文件，目录可以改，文件名也可以改                     |
| -drive id=cdrom,media=cdrom,if=none,file=ubuntuordebian.iso    -device virtio-scsi-device -device scsi-cd,drive=cdrom \ | debian-10.4.0-arm64-xfce.iso                  | 启动关盘镜像                                                 |
| -drive id=hd0,media=disk,if=none,file=debian.qcow2   -device virtio-scsi-pci -device scsi-hd,drive=hd0 | debian.qcow2                                  | 磁盘，系统将会安装到这个磁盘中                               |
| -nic user,model=virtio-net-pci,hostfwd=tcp::2222-:22,hostfwd=tcp::6900-:5900 | hostfwd=tcp::2222-:22,hostfwd=tcp::6900-:5900 | 端口转发，宿主机2222端口映射虚拟机22端口，6900对应5900，可按需更改 |
|                                                              | -nographic                                    | 不使用qemu gui窗口                                           |
|                                                              | -monitor stdio                                | 默认hi用qemu的serial0窗口                                    |
|                                                              | -device virtio-gpu-pci                        | 用于显示带gui的操作系统，安装时不建议使用该选项，因为在Window和Linux下键盘鼠标都无法使用，安装完成后运行时可使用（ARM下官方推荐的两个显示设备之一） |
|                                                              | -device ramfb                                 | 用于显示带gui的操作系统，安装时不建议使用该选项，因为在Window和Linux下键盘鼠标都无法使用，安装完成后运行时可使用（ARM下官方推荐的两个显示设备之一） |



### 4、运行并添加UEFI启动项

注意，真正的难题出现了，安装完成后，如果使用-bios QEMU_EFI.fd启动，登陆时会按默认的UEFI启动项顺序启动，会出现` Start PXE over IPv4`等信息，无法启动进入系统。关于这个问题，Debian官网有描述，参见[Debian官网Wiki](https://wiki.debian.org/UEFI#Booting_from_removable_media"debian uefi")，出现的原因，是因为原有的UEFI引导项没有从grubaa64.efi启动的，那我们需要新建一个引导项从grubaa64.efi启动。但是，由于-bios QEMU_EFI.fd是只读的，我们即便本次新建了引导项，等关机后再次启动时，新增的UEFI引导项会丢失，官方提供了pflash来解决这问题，它是可读写的。我这里提供了几个Demo文件，可以让你直接启动到系统，但是仅适用于Debian，如果是ubuntu，你还需要自己动手。

首先，我们将QEMU_EFI.fd文件打包到img文件中，给64MB大小即可，在Linux下执行以下命令

```
dd if=/dev/zero of=flash0.img bs=1M count=64
dd if=QEMU_EFI.fd of=flash0.img conv=notrunc
dd if=/dev/zero of=flash1.img bs=1M count=64
```

在上一步中，会生成两个文件`flash0.img`和`flash1.img`，我们用pflash来启动

```
#on Windows cmd
qemu-system-aarch64 ^
    -M virt -m 3G -cpu cortex-a72 -smp 2 ^
    -drive file=flash0.img,format=raw,if=pflash ^
    -drive file=flash1.img,format=raw,if=pflash ^
    -drive id=hd0,media=disk,if=none,file=debian.qcow2 ^
    -device virtio-scsi-pci -device scsi-hd,drive=hd0 ^
    -nic user,model=virtio-net-pci,hostfwd=tcp::2222-:22,hostfwd=tcp::6900-:5900 ^
    -monitor stdio
    
 #on Linux terminal
 qemu-system-aarch64 \
    -M virt -m 3G -cpu cortex-a72 -smp 2 \
    -drive file=flash0.img,format=raw,if=pflash \
    -drive file=flash1.img,format=raw,if=pflash \
    -drive id=hd0,media=disk,if=none,file=debian.qcow2 \
    -device virtio-scsi-pci -device scsi-hd,drive=hd0 \
    -nic user,model=virtio-net-pci,hostfwd=tcp::2222-:22,hostfwd=tcp::6900-:5900 \
    -nographic
    
#注意事项
-monitor stdio是默认使用serial0控制台，在windows下请使用此项,如在linux下，请替换为-nographic，因为在Linux中存在键盘方向键无法使用的问题，十分不便

如果安装了桌面环境，无论Windows还是Linux，请替换为-device virtio-gpu-pci 或 -device ramfb
```

注意，因为我们还没有添加引导项，所以本次登陆时仍会出现` Start PXE over IPv4`等类息，此时我们在控制台输入以下内容(一个标点符号都不要少)，然后Enter，即可正常进入系统

```
#for debian
\EFI\debian\grubaa64.efi

#ubuntu
\EFI\ubuntu\grubaa64.efi
```

成功进入系统后，我们新增一个uefi启动选项，并设置为启动首选项，通过以下命令即可解决问题

```
#for debian
 efibootmgr -c -d /dev/sda -p 1 -L debian -l \EFI\debian\grubx64.efi

#ubuntu
 efibootmgr -c -d /dev/sda -p 1 -L debian -l \EFI\ubuntu\grubx64.efi

#执行成功的情况下，可以看到如下响应
#大致意思是，列出来现有的启动项，并且新增了序号为Boot0007*，备注为debian的启动项，而且在启动顺序BootOrder中，我们新增的007已经排列到第一位了，关机后下次再进入系统，就无需再手动输入uefi引导文件的位置了

BootCurrent: 0006
Timeout: 3 seconds
BootOrder: 0007,0000,0001,0002,0003,0004,0005,0006
Boot0000* UiApp
Boot0001* UEFI Misc Device
Boot0002* UEFI Misc Device 2
Boot0003* UEFI QEMU QEMU HARDDISK 
Boot0004* UEFI PXEv4 (MAC:525400123456)
Boot0005* UEFI HTTPv4 (MAC:525400123456)
Boot0006* EFI Internal Shell
Boot0007* debian
```

### 5、写在结尾

```
Good Luck，感谢你看到这里，希望能给你一些帮助。不出意外，你已经成功了。这里的一些问题，曾经一度困扰我很久，中间断断续续了很多次，占用了我许多上下班时间，但是还是坚持下来了，给自己的点个赞。办法总比困难多，仔细看文档，许多问题官方文档都有写。如果一直搞不定，想必我会一直没心思认真搞别的，哈哈，明天开始终于可以放下心去做更重要的事情了。
```

### 6、搜索关键字

`qemu-system-aarch64` `debian-buster-arm64-xfce` `debian-10.4.0-arm64-xfce` `ubuntu-focal-live-server-arm64` `ubuntu-20.04-live-server-arm64` `qemu arm64 uefi` `qemu aarch64 uefi`
