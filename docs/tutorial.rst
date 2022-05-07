基本使用教程
============

Bootarmor 支持在原来的 Debian Linux （例如 Ubutnu 或者 RaspberryOS）中直接安装新
的安全内核，重新启动之后就可以将原来的系统升级成为可以运行安全应用的操作系统。

Bootarmor 提供了命令行工具 :ref:`btarmor` ，可以帮助用户完成大部分的功能。

支持平台
--------

目前 Bootarmor 内核只支持 arm64 架构的 Debian 11 (bullseye) 和 RaspberryOS ，其
它系统和版本的支持会在以后逐渐加入。

命令行工具 :ref:`btarmor` 可以跨平台运行，目前支持 arm64 和 x86_64 两种架构。

安装命令行工具 btarmor
----------------------

可以根据使用习惯使用下面的任意一种方式安装

1. 使用 apt 安装

启动任意 Debian Linux 系统之后，使用下面的方式增加安装源之后，就可以使用常用的管
理命令 `apt` 命令安装 Bootarmor 提供的各种包::

    wget -O /etc/apt/trusted.gpg.d/btarmor-archive-keyring.gpg \
         https://btarmor.dashingsoft.com/apt/debian/btarmor-archive-keyring.gpg

    sudo echo "deb https://btarmor.dashingsoft.com/apt/debian bullseye contrib" \
              > /etc/apt/sources.list.d/btarmor.list

    sudo apt update

命令行工具 :ref:`btarmor` 在包 `btarmor-cli` 中，使用下面的命令直接安装::

    sudo apt install btarmor-cli

查看命令 :ref:`btarmor` 是否安装成功::

    btarmor -v

安装成功的话，会显示相关的版本信息。

使用下面的命令清除 btarmor 相关的安装源配置::

    sudo rm /etc/apt/trusted.gpg.d/btarmor-archive-keyring.gpg \
            /etc/apt/sources.list.d/btarmor.list

2. 使用 pip 安装

如果对 Python 比较熟悉，可以直接使用下面的命令安装::

    pip install btarmor-cli

安装完成之后同样会创建命令 :ref:`btarmor`

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

关于转换应用程序的更多方式，请参考命令手册 :ref:`btarmor make`

对于使用 C 编写的应用程序，可以在源代码级别提供更多的安全性选项，具体请参考
:ref:`C 用户使用手册`

一般来说，转换成为安全应用之后应用程序的代码就无法使用逆向工程进行反编译，就可以
发布给客户。

但是对于企业版用户来说，可以使用下面的发布功能来进一步加强内核的安全性。

发布安全应用
------------

.. note::

   发布功能仅适用于企业版，社区版不提供这个功能。

发布功能会对内核进行加壳处理，从而提供更高的安全性，使用下面的命令转换安全内核为发布模式::

  btarmor deploy

更多详细的使用方式，请参考命令手册 :ref:`btarmor deploy`

.. include:: _common_definitions.txt
