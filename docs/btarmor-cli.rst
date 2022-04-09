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

    boot    安装安全系统
    make    创建安全应用
    deploy  发布安全应用
    patch   下载安全内核的源代码补丁

可以运行 `btarmor <command> -h` 查看各个命令的详细使用方法。

.. _boot:

boot
====

用于安装安全操作系统 `btarmor-os`

**语法**::

    btarmor boot [IMAGE]

**描述**

如果没有对内核进行定制，那么直接运行下面的命令安装 `btarmor-os`::

  btarmor boot

如果是对内核进行了定制，那么需要下载相应内核版本的补丁::

  btarmor patch -O btarmor-os.patch

在源文件目录应用该补丁文件，重新生成 Linux 内核，在命令行中指定新的内核文件名称::

  sudo btarmor boot /PATH/TO/IMAGE

重新启动系统之后，其内核就已经是安全操作系统 `btarmor-os`_

如果已经是安全内核，那么运行该命令则显示安全系统的相关信息。

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
-sys, --share			使用共享模式进行保护，一般用于保护系统动态库
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

选项 ``-sys`` 一般用于加密系统动态库，主要是为了保护系统命令不能被修改，并且在高
级模式下面能够执行系统命令。因为高级模式下面，没有加密的任何可执行文件都无法运行。
使用该选项加密的文件允许内核访问，同时这个可执行文件也只能运行在应用程序层，而没
有运行在安全应用层。

选项 ``--safe-heap`` 用于保护内存堆，这里面一般是程序使用 ``malloc`` 申请的内存
空间，默认情况下是允许内核访问的。如果这些数据不需要被内核访问，使用该选项可以提
高安全性。

选项 ``--safe-stack`` 用于保护运行栈，这里面一般是程序的调用框架和局部变量，默认
情况下是允许内核访问的。如果这些数据不需要被内核访问，使用该选项可以提高安全性。

**示例**

* 转换目录 ``foo`` 下面的所有文件，保存生成的安全文件到到默认输出目录 ``dist``::

    btarmor make foo/

* 只转换两个动态库文件，转换之后直接覆盖原来的文件::

    btarmor make -i dist/foo.so dist/too.so

* 没有使用任何内核服务的程序，对内存堆和运行栈也进行保护::

    btarmor make -sh -ss myapp

* 加密保护数据文件到 ``dist/data.txt``::

    btarmor make data.txt

* 使用共享模式加密系统动态库::

    sudo btarmor make -i --share /usr/lib/*.so

* 加密系统命令 ``ls``::

    sudo btarmor make -i -sys /usr/bin/ls

.. _deploy:

deploy
======

用于产品开发完成之后，发布运行安全应用的系统到最终用户。

**语法**::

    btarmor deploy

**描述**

在产品准备交付给用户之前，在 `btarmor-os` 上面把应用程序和相关的系统库转换称为安
全应用，然后运行这个命令切换到发布模式。

.. _patch:

patch
=====

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


.. include:: _common_definitions.txt
