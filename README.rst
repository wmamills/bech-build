################################################
Standalone firmware and Linux kernel environment
################################################

.. contents::

This project is a helper project to be able to create a development environment
for QEMU consisting of Linux kernel, U-Boot, Buildroot and QEMU itself.

Prerequisites
=============
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

Configure netboot
=================
At the U-boot prompt (change IP to the IP of your computer where you are doing
this). The lines below setup the environment variables to load directly into
memory as well as loading a FIT image.

.. code-block:: bash

    => setenv nbr "dhcp; setenv serverip 192.168.1.110; tftp ${kernel_addr_r} uImage; tftp ${ramdisk_addr_r} rootfs.cpio.uboot; bootm ${kernel_addr_r} ${ramdisk_addr_r} ${fdt_addr}"
    => setenv bootfit "dhcp; setenv serverip 192.168.1.110; tftp 0x48000000 kernel.itb; bootm"
    => saveenv

Run
===

Netboot using raw files
-----------------------
This will load files directly from TFTP (must be configured, see further down)
into the correct memory location and then it will will boot using ``bootm``.

.. code-block:: bash

    $ make run-netboot
    => run nbr

Netboot using FIT image
-----------------------
This will load a fit image directly from TFTP (must be configured, see further
down) into memory. The FIT image consist of the kernel image, the rootfs
(Buildroot) and the DTB coming from QEMU. The DTB is in memory by default in
QEMU, but since we want to emulate a real flow, we dump the DTB to a file, then
use that when creating the FIT-image and then load it to the same address where
QEMU would have put it initially. When you have made changes, then you need to
update the FIT-image.

.. code-block:: bash

    $ make fit

Once the FIT-image has been updated you can boot up QEMU and then the below that
will load the FIT-image image and bootm the content of it.

.. code-block:: bash

    $ make run-netboot
    => run bootfit

If everything goes well, you images shall be fetched from the TFTD server and
you should end up with Linux kernel booting and you get a Buildroot prompt.

TFTP
****
Setup the tftp server
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


// Joakim Bech
2020-11-18

