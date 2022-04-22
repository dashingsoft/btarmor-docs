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

用于安装安全操作系统 `btarmor-os`_ ，并且能够禁用或者启用相关的功能特性。

**语法**::

    btarmor boot [status]
    btarmor <enable | disable> <exclusive | senior>

**描述**

直接运行下面的命令会安装新的内核，一般用于初始化安全操作系统::

    sudo btarmor boot

这个命令会安装新的内核，所以需要使用 `sudo` 获得 root 权限

重新启动系统之后，其内核就已经是安全操作系统 `btarmor-os`_

下面的命令则显示安全系统的相关信息::

    btarmor boot status

默认情况下， Linux 内核是可以访问安全应用的内存。如果启用高级模式，那么不允许内
核直接访问安全应用的内存。如果需要访问，必须得到安全应用的授权。这样能提高安全性，
但是也需要更多的了解操作系统的内存管理，ELF文件的组成，以及编译器的工作原理等相
关知识。否则，安全应用可能无法正常运行，或者即便运行，也降低了安全性。

使用下面的命令启用高级模式::

    sudo btarmor enable senior

这个命令同样会修改内核，所以需要 root 用户权限，并且需要重启系统才能生效。

使用下面的命令禁用高级模式（需要重启系统才能生效）::

    sudo btarmor disable senior

默认情况下，安全内核中普通程序可以正常运行，如果启用内核的专用模式则只有加密后的
安全应用可以运行，普通程序在安全系统中无法运行。

使用下面的命令启用专用模式::

    sudo btarmor enable exclusive

这个命令同样会修改内核，所以需要 root 用户权限，并且需要重启系统才能生效。

在切换到专用模式之前一般需要使用 `btmake`_ 将系统命令转换成为被保护的认证应用，
否则一旦启用专用模式，所有系统命令也将无法运行。

使用下面的命令禁用专用模式（需要重启系统才能生效）::

    sudo btarmor disable exclusive

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
--exact, --no-deps		不自动处理ELF文件的依赖库
-f FILE				从文件里面读取需要处理的文件名称

**描述**

make 子命令用于将命令行列出的一个或者多个文件转换成为安全文件，如果列出的是目录，
那么会递归处理目录下面的所有可执行文件和动态库。

例如::

    btarmor make /home/jondy/myapp

如果文件是一个压缩包，那么压缩包会作为解压后的一个目录进行处理。

这个命令会递归加密 myapp 目录下面的所有可执行文件和动态库，对于可执行文件所依赖
的系统动态库，也会使用共享模式进行加密处理。如果不需要处理系统动态库，使用选项
``--no-deps`` ，例如::

    btarmor make --no-deps /home/jondy/myapp

如果需要处理数据文件，可以在命令行直接列出，例如::

    btarmor make --no-deps /home/jondy/myapp /home/jondy/myapp/config.txt

还可以把所有需要处理的文件名称都存放到一个文件里面，例如 ``files.list`` ::

    /home/jondy/myapp/foo
    /home/jondy/myapp/libtoo.so
    /home/jondy/myapp/config.txt

然后在命令行使用 ``-f`` 指定这个文件，例如::

    btarmor make -f files.list

默认情况下生成的新文件会保存在另外一个输出目录 ``dist`` ，使用选项 ``--output``
可以指定输出目录。

``-f`` 也可以指定压缩包里面的路径，来选择压缩包里面的文件。例如::

    btarmor make -f tarfile.list foo-1.0.tar.gz

``btamake`` 也可以直接处理一个未签名的 Debian 包。例如::

    btarmor make foo_2.31-2_aarch64.deb

如果指定了选项 ``--inplace`` ，那么输出文件会直接覆盖原来的文件。

选项 ``-sys`` 一般用于加密系统动态库，主要是为了保护系统命令不能被修改，并且在高
级模式下面能够执行系统命令。因为专用模式下面，没有加密的任何可执行文件都无法运行。
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

用于产品开发完成之后，发布运行安全应用的系统到最终用户。这个命令主要对内核进行加
壳，加壳之后内核的功能特性就不能被改变， `enable` 和 `disable` 命令会失效。

**语法**::

    btarmor deploy <options>

**选项**

-ns, --no-senior		安全内核不使用高级模式
-ne, --no-exclusive		安全内核不使用专用模式

**描述**

在产品准备交付给用户之前，需要使用该命令把内核功能进行加壳固化，清理开发辅助的文
件。加壳之后内核的功能特性就不能被改变， `enable` 和 `disable` 命令会失效。

默认情况下安全内核运行于高级模式，任何低权限级的内存访问都会导致保护异常。如果这
样导致某些应用无法正常运行，可以使用选项 ``--no-senior`` 禁用高级模式，这样就允
许低权限级别应用访问安全内存。

默认情况下安全内核处于专用模式下，只有安全应用和认证应用才可以运行。如果需要运行
非安全应用，可以使用选项 ``--no-exclusive`` 禁用专用模式，这样，普通的应用也可以
正常运行。

.. _patch:

patch
=====

下载安全内核的源代码补丁，用于定制自己的安全内核。

**语法**::

    btarmor patch <options> VERSION

**选项**

-O, --output PATH		保存到指定目录，默认是 patches
-l, --list			列出所有可用版本的补丁名称

**描述**

直接下载和当前系统内核版本相同的补丁，保存到目录 `patches/`::

  btarmor patch

下载指定内核版本 `5.10.73`::

  btarmor patch 5.10.73

查看所有支持的内核版本的补丁::

  btarmor patch --list

也可以从下面的链接查看和下载对应版本的 `btarmor-os`_ 补丁

    https://btarmor.dashingsoft.com/kernel/patches/


.. include:: _common_definitions.txt
