.. _下载和安装:

下载和安装
==========

在 Linux 下安装 Bootarmor OS 到树莓派
-------------------------------------

下载 Bootarmor OS

https://bootarmor.dashingsoft.com/downloads/bootarmor-os-0.1.1.img

把 SD 卡插入读写器，然后把读写器插入到 USB 接口，查看设备信息

.. code:: bash

    $ sudo lsblk -l
    NAME                              MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
      ...
    sdb                                 8:16   1  14.9G  0 disk
    ├─sdb1                              8:17   1   256M  0 part
    └─sdb2                              8:18   1  14.6G  0 part

这里可以看到 SD 卡对应的设备名称为 `/dev/sdb` ，那么使用 `dd` 把下载的映像文件写入

.. code:: bash
    $ sudo dd if=bootarmor-os-0.1.1.img of=/dev/sdb bs=4M conv=fsync

把写好的 SD 卡取出，并插入到树莓派，连接电源，启动树莓派，进入到 Linux 命令行控制台。

.. include:: _common_definitions.txt
