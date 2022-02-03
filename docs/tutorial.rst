基本使用教程
============

下面以树莓派为例说明用户如何发布安全的应用

首先参考 :ref:`下载和安装` 安装 Bootarmor OS 到树莓派

发布安全的可执行文件
--------------------

用户把一个应用程序 `foo` 发布到安全环境的场景

首先拷贝 `foo` 到树莓派，打开树莓派控制台，运行下面的命令::

  btarmor foo

这样就会转换 `foo` 为安全应用，然后就可以运行这个安全应用::

  mv foo /usr/bin/
  foo

发布安全的动态库
----------------

用户把一个动态库 `foo.so` 发布到安全环境的场景

首先拷贝 `foo.so` 到树莓派，打开树莓派控制台，运行下面的命令::

  btarmor foo.so

这样就会转换 `foo.so` 为安全动态库，然后就可以把这个动态库拷贝到系统库
目录以供使用::

  mv foo.so /usr/lib/

发布安全的安装包
----------------

用户把包含可执行文件/动态库/数据文件的包 `foo.tar.gz` 发布到安全环境的
场景

首先拷贝 `foo.tar.gz` 到树莓派，打开树莓派控制台，运行下面的命令::

  btarmor foo.tar.gz

默认会输出一个 `foo.armored.tar.gz` 安全安装包，然后删除老的包，解压安
全安装包到安装目录::

  rm foo.tar.gz
  mkdir -p /opt/foo
  tar xzf foo.armored.tar.gz /opt/foo

.. include:: _common_definitions.txt
