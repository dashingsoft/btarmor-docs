基本使用教程
============

Bootarmor 支持在 Debian Linux 中直接安装一个新的安全内核，重新启动之后就可以将原
来的系统升级成为可以运行安全应用的操作系统。

安装命令行工具
----------------

1. 修改 `/etc/apt/source.list` ，增加一行::

     deb https://btarmor.dashingsoft.com/btarmor bullseye main

2. 更新安装源::

     sudo apt update

3. 安装命令行工具 :ref:`btarmor`::

     apt get btarmor-cli


安装安全操作系统
----------------

安装好命令行工具 :ref:`btarmor` 之后，直接使用下面的命令安装安全操作系统::

  sudo btarmor boot

然后重新启动系统就进入 `btarmor-os`

创建安全应用
------------

创建安全应用就是把原来的可执行文件，动态库和数据文件使用命令转换成为安全应用。

例如，转换被保护的应用程序 ``/opt/foo`` 为安全应用::

  btamor make -i /opt/foo

该目录下面的所有可执行文件和动态库，以及使用到的所有系统动态库都会被转换成为安全
应用。

发布安全应用
------------

当所有开发完成之后，将产品交付给用户之前，需要把安全操作系统转换成为发布模式，转
换成为发布模式之后只有安全应用可以正常运行，其它可执行文件和动态库都无法使用。

使用下面的命令转换安全内核为发布模式::

  btarmor deploy

现在可以把产品交付给给用户使用。

.. include:: _common_definitions.txt
