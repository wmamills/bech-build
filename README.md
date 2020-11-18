################################################
Standalone firmware and Linux kernel environment
################################################

.. contents::

This project is a helper project to be able to create a development enviroment
for QEMU consisting of Linux kernel, U-Boot, Buildroot and QEMU itself.

Installation
************

Prerequisites
=============
TBD

Setup
=====
Clone this git project

.. code-block:: bash

	$ make -j2 toolchains
3. (see TFTP instructions further down)

Compile
=======
1. $ make

Run
===
1. $ make run-netboot

Configure netboot
=================
At the U-boot prompt (change IP to the IP of your computer where you are doing this)
=> setenv nbr "dhcp; setenv serverip 192.168.1.110; tftp ${kernel_addr_r} uImage; tftp ${ramdisk_addr_r} rootfs.cpio.uboot; bootm ${kernel_addr_r} ${ramdisk_addr_r} ${fdt_addr}"
=> saveenv
=> run nbr

TFTP
****
Setup the tftp server
=====================
Credits to the author of [this](https://developer.ridgerun.com/wiki/index.php?title=Setting_Up_A_Tftp_Service)
guide.
```
sudo apt install xinetd tftpd tftp
```

```
$ sudo vim /etc/xinetd.d/tftp
```
and paste

```
service tftp
{
    protocol        = udp
    port            = 69
    socket_type     = dgram
    wait            = yes
    user            = nobody
    server          = /usr/sbin/in.tftpd
    server_args     = /srv/tftp
    disable         = no
}
```
Save the file and exit.

Create the directory
```
$ sudo mkdir /srv/tftp
$ sudo chmod -R 777 /srv/tftp
$ sudo chown -R nobody /srv/tftp
```

Start tftpd through xinetd

```
sudo /etc/init.d/xinetd restart
```

## Symlink kernel and dtb
```
$ cd /srv/tftp
$ ln -s <project_path>/imx8mqevk/linux/arch/arm64/boot/Image .
$ ln -s <project_path>/imx8mqevk/linux/arch/arm64/boot/dts/freescale/imx8mq-evk.dtb fsl-imx8mq-evk.dtb
```

## Boot up
Make sure you have an SD-card with at least the bootloader on it (minimum
`compile` and `flash bootloader only`). Plug in the Ethernet cable to the
IMX8MQ device, then turn on the device and halt U-Boot when it is counting
down, then run:
```
u-boot=> run netboot
```

// Joakim Bech
2020-11-18

