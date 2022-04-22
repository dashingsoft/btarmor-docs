基本使用教程
============

Bootarmor 支持在原来的 Debian Linux （例如 Ubutnu 或者 RaspberryOS）中直接安装新
的安全内核，重新启动之后就可以将原来的系统升级成为可以运行安全应用的操作系统。

Bootarmor 提供了命令行工具 :ref:`btarmor` ，可以帮助用户完成大部分的功能。

安装命令行工具 btarmor
----------------------

启动任意 Debian Linux 系统之后，使用下面的方式增加安装源之后，就可以使用常用的管
理命令 `apt` 命令安装 Bootarmor 提供的各种包::

    wget -O - https://btarmor.dashingsoft.com/apt/debian/dashingsoft-btarmor.gpg \
         | sudo apt-key add -

    sudo echo "https://btarmor.dashingsoft.com/apt/debian bullseye contrib" \
              > /etc/apt/sources.list.d/btarmor.list

    sudo apt update

命令行工具 :ref:`btarmor` 在包 `btarmor-cli` 中，使用下面的命令直接安装::

    sudo apt install btarmor-cli

查看命令 :ref:`btarmor` 是否安装成功::

    btarmor -v

安装成功的话，会显示相关的版本信息。

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
