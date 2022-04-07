==================
 btarmor 命令手册
==================

命令行工具 `btarmor`_ 的用户手册，适用于使用 `btarmor`_ 命令，需要了解
其更多高级功能的用户。

.. _btarmor:

btarmor
=======

Bootarmor 提供了一个命令行工具 `btarmor`_ ，用于转换内核和应用程序为安全应用。

`btarmor` 的语法格式::

    btarmor <command> [options]

常用的命令包括::

    boot    创建安全系统（开发模式）
    make    创建安全应用
    deploy  发布安全系统（发布模式）
    patch   下载安全内核的源代码补丁

可以运行 `btarmor <command> -h` 查看各个命令的详细使用方法。

btarmor boot
------------

用于安装开发模式的安全内核。

**语法**::

    btarmor boot [IMAGE]

**描述**

直接运行下面的命令安装开发模式的安全内核::

    btarmor boot

重新启动系统之后，其内核就已经是安全操作系统 `btarmor-os`_

如果已经是安全内核，那么运行该命令则显示安全系统的相关信息。

如果是定制的内核，那么需要在命令中指定内核映像，例如::

    btarmor boot /path/to/Image

btarmor deploy
--------------

用于安装发布模式的安全内核，该命令必须运行在开发模式的安全内核。

**语法**::

    btarmor deploy

**描述**

在产品准备交付给用户之前，在开发模式的安全内核上面运行该命令生成发布模式的安全内
核。

btarmor patch
--------------

下载安全内核的源代码补丁，用于定制自己的安全内核。

**语法**::

    btarmor patch <options> VERSION

**选项**

-O, --output PATH		保存到指定的文件，默认是标准输出
-l, --list			列出所有可用版本的补丁名称

**描述**

直接下载和当前系统内核版本相同的补丁，保存到 `btarmor.patch`::

  btarmor patch -O btarmor.patch

下载指定内核版本 `5.10.73`::

  btarmor patch -O btarmor.patch 5.10.73

查看所有支持的内核版本的补丁::

  btarmor patch --list

也可以从下面的链接查看和下载对应版本的 `btarmor-os`_ 补丁

    https://btarmor.dashingsoft.com/kernel/patches/

.. _btmake:

btmake
======

用于将可执行文件，动态库和数据文件等，转换成为 Bootarmor 保护的文件，转换后生成
的文件是经过加密的，无法在普通系统中运行，只能在 `btarmor-os`_ 系统上运行。

`btmake`_ 可以运行在 `btarmor-os`_ 系统，直接将应用程序转换成为安全应用。

`btmake`_ 也可以运行在其它 Linux 系统，先将应用程序转换成为安全应用，然后再把安
全应用发布到 `btarmor-os`_ 系统中。

**语法**::

    btarmor make <options> PATH...

    btmake <options> PATH...

**选项**

-i, --inplace			生成的文件直接覆盖原来的文件
-O, --output PATH		生成的文件保存在另外一个路径，默认是 `dist`
-sys, --share			使用共享模式进行保护，一般用于保护系统动态库和允许内核访问的文件
-sh, --safe-heap		不允许内核访问应用程序申请的堆空间
-ss, --safe-stack               不允许内核访问运行栈（局部变量）

**描述**

make 子命令用于将命令行列出的一个或者多个文件转换成为安全文件，如果列出的是目录，
那么会递归处理目录下面的所有文件。

最常用的方式是在执行命令之前，把需要处理的可执行文件，动态库，需要加密的数据文件
拷贝到一个目录，然后直接处理该目录。

默认情况下生成的新文件会保存在另外一个输出目录 ``dist`` ，使用选项 ``--output``
可以指定输出目录。

如果指定了选项 ``--inplace`` ，那么输出文件会直接覆盖原来的文件。

选项 ``-sys`` 一般用于加密系统动态库，使用该选项加密的文件允许内核访问。如果使用
该选项加密可执行文件，那么这个可执行文件也只能运行在应用程序层。主要是为了让一些
系统命令可以在发布模式下面执行。

选项 ``--safe-stack`` 用于保护运行栈，这里面一般是程序的调用框架和局部变量，默认
情况下是允许内核访问的。如果这些数据不需要被内核访问，使用该选项可以提高安全性。

**示例**

* 转换目录 ``foo`` 下面的所有文件，保存生成的安全文件到到默认输出目录 ``dist``::

    btarmor make foo/

* 只转换两个动态库文件，转换之后直接覆盖原来的文件::

    btarmor make -i dist/foo.so dist/too.so

* 没有使用任何内核服务的程序，对运行栈也进行保护::

    btarmor make -ss myapp

* 加密保护数据文件到 ``dist/data.txt``::

    btarmor make data.txt

* 使用共享模式加密系统动态库::

    sudo btarmor make -i --share /usr/lib/*.so

* 加密系统命令 `ls`::

    sudo btarmor make -i -sys /usr/bin/ls

.. include:: _common_definitions.txt
