################################################
Standalone firmware and Linux kernel environment
################################################

.. contents::

This project is a helper project to be able to create a development environment
for QEMU consisting of Linux kernel, U-Boot, Buildroot and QEMU itself.

Prerequisites
*************
TBD, but depending on the setup, you should consider setting up TFTP (see TFTP
instructions further down).


Installation
************

Get the source code
===================
Install ``repo`` by following the installation instructions 
`here <https://source.android.com/setup/build/downloading>`_.

Then initialize the tree 

.. code-block:: bash

    $ mkdir -p <path-to-my-project-root>
    $ cd <path-to-my-project-root>
    $ repo init -u https://github.com/jbech-linaro/manifest.git -b dte

Next sync the actual tree

.. code-block:: bash

    $ repo sync -j4

Compile
=======

.. code-block:: bash

    $ make -j2 toolchains
    $ make -j4


**Compiler flags**

 - GDB: **y** enables the GDB stub in QEMU, **n** is disabled (default)
 - SIGN: **y** enables signature verification of FIT images, **n** is disabled (default)


Configure netboot
=================
At the U-boot prompt (change IP to the IP of your computer where you are doing
this). The lines below setup the environment variables to load directly into
memory as well as loading a FIT image.

.. code-block:: bash

    => setenv nbr "dhcp; setenv serverip 192.168.1.110; tftp ${kernel_addr_r} uImage; tftp ${ramdisk_addr_r} rootfs.cpio.uboot; bootm ${kernel_addr_r} ${ramdisk_addr_r} ${fdt_addr}"
    => setenv bootfit "dhcp; setenv serverip 192.168.1.110; tftp 0x48000000 image.fit; bootm"
    => setenv loadfit "dhcp; setenv serverip 192.168.1.110; tftp 0x48000000 image.fit"
    => setenv c1 "bootm 0x48000000#config-1"
    => setenv c2 "bootm 0x48000000#config-2"
    => setenv c3 "bootm 0x48000000#config-3"
    => setenv c4 "bootm 0x48000000#config-4"
    => saveenv

A note about the address ``0x48000000``. That is an address in RAM that should
be on a sufficient offset to not overlap with the kernel, dtb and the rootfs. If
content of these grows, then we might need to move that offset to a higher
address.

Run targets
===========

**Netboot using raw files**

This will load files directly from TFTP (must be configured, see further down)
into the correct memory location and then it will boot using ``bootm``. No
special build commands is required for this target.

.. code-block:: bash

    $ make run-netboot
    => run nbr

**Netboot using FIT image**

This will load a FIT image directly from TFTP (must be configured, see further
down) into memory. The FIT image consist of the kernel image, the rootfs
(Buildroot) and the DTB coming from QEMU. The DTB is in memory by default in
QEMU, but since we want to emulate a real flow, we dump the DTB to a file, then
use that when creating the FIT-image and then load it to the same address where
QEMU would have put it initially. When you have made changes, then you need to
update the FIT-image (i.e., re-run make). No special build commands is required
for this target.

Once the FIT-image has been updated you can boot up QEMU and then the below that
will load the FIT-image image and bootm the content of it.

.. code-block:: bash

    $ make run-netboot
    => run bootfit

**Netboot using a signed FIT image**

Same as for the FIT image above, with the difference the you need to enable
signature support when building. If we'd run the ``bootfit`` target, then the
outcome would be the same as above. Here we're interest in running a signed
kernel. So for that we do it two steps. First we load the FIT image into memory,
then we bootm the ``config-3`` (which is signed kernel) using previously
configured U-Boot environment command (``c3``).

.. code-block:: bash

    $ make SIGN=y
    $ make run-netboot
    => run loadfit
    => run c3


**Boot Linux kernel directly**

If you just need to boot Linux kernel directly without using nor involving
U-Boot, then you can do that running the run target below. Note that in this
case there is no signature verification etc enabled.

.. code-block:: bash

    $ make run-kernel-initrd


TFTP
****
Setup the TFTP server
=====================
Credits to the author of `this <https://developer.ridgerun.com/wiki/index.php?title=Setting_Up_A_Tftp_Service>`_
guide.

.. code-block:: bash

    $ sudo apt install xinetd tftpd tftp
    $ sudo vim /etc/xinetd.d/tftp

and paste

.. code-block:: bash

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

Save the file and exit, then create the directory and fix permissions

.. code-block:: bash

    $ sudo mkdir /srv/tftp
    $ sudo chmod -R 777 /srv/tftp
    $ sudo chown -R nobody /srv/tftp

Start tftpd through xinetd

.. code-block:: bash

    $ sudo /etc/init.d/xinetd restart

Symlink the necessary files
===========================
.. code-block:: bash

    $ cd /srv/tftp
    $ ln -s <project_path>/linux/arch/arm64/boot/Image .
    $ ln -s <project_path>/linux/arch/arm64/boot/Image.gz .
    $ ln -s <project_path>/buildroot/output/images/rootfs.cpio.uboot .
    $ ln -s <project_path>/buildroot/output/images/rootfs.cpio.gz .
    $ ln -s <project_path>/out/qemu-aarch64.dtb .

Or a maybe simpler alternative is to go to the TFTP server directory and symlink
all files in ``<project_path>/out`` in one go. Do this **after** completing the
first build!

.. code-block:: bash

    $ cd /srv/tftp
    $ ln -s <project_path>/out/* .


// Joakim Bech
2021-01-20
