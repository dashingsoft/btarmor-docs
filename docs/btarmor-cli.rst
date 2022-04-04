==================
 btarmor 命令手册
==================

命令行工具 `btarmor`_ 的用户手册，适用于使用 `btarmor`_ 命令，需要了解
其更多高级功能的用户。

.. _btarmor:

btarmor
=======

Bootarmor 提供了一个额外的命令行工具 `btarmor`_ ，用于转换应用程序为安全应用。

`btarmor` 的语法格式::

    btarmor <command> [options]

常用的命令包括::

    make    创建安全应用
    sys     保护系统包

可以运行 `btarmor <command> -h` 查看各个命令的详细使用方法。

.. _btmake:

btmake
======

用于将可执行文件，动态库和数据文件等，转换成为 btarmor 保护的文件，转换后生成的
文件是经过加密的，无法在普通系统中运行，只能在 btarmor-os 系统上运行。

**语法**::

    btarmor make <options> PATH...

    btmake <options> PATH...

**选项**

-i, --inplace			生成的文件直接覆盖原来的文件
-O, --output PATH		生成的文件保存在另外一个路径，默认是 `dist`
--share				使用共享模式进行保护，一般用于保护系统动态库和允许内核访问的文件
-ss, --safe-stack               保护运行栈（局部变量）

**描述**

make 子命令用于将命令行列出的一个或者多个文件转换成为安全文件，如果列出的是目录，
那么会递归处理目录下面的所有文件。

最常用的方式是在执行命令之前，把需要处理的可执行文件，动态库，需要加密的数据文件
拷贝到一个目录，然后直接处理该目录。

默认情况下生成的新文件会保存在另外一个输出目录 ``dist`` ，使用选项 ``--output``
可以指定输出目录。

如果指定了选项 ``--inplace`` ，那么输出文件会直接覆盖原来的文件。

选项 ``--share`` 一般用于加密系统动态库，使用该选项加密的文件允许内核访问。

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

.. _btsys:

btsys
=====

检查系统当前所有安装包，并转换所有系统包为安全应用

**语法**::

    btarmor sys <options>

    btsys <options>

**选项**

-l, --list			列出系统所有安装的包的状态，是否被保护

**描述**

在安装安全系统内核 `btarmor-kernel` 之后，需要运行一次下面的命令，将系
统所有的包转换成为安全应用::

    btarmor sys

查看当前系统安装包是否已经转换成为安全应用，可以使用下面的命令::

    btarmor sys -l

.. include:: _common_definitions.txt
