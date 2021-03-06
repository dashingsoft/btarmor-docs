.. _btarmor:

btarmor
=======

安全操作系统 :ref:`btarmor-os` 的命令行工具。

语法
----

.. code-block:: console

    btarmor <command> [options]

描述
----

`btarmor`_ 命令行工具主要的功能包括

* 将原来的 Debian 系统升级成为安全操作系统 :ref:`btarmor-os`
* 将应用程序/动态库/数据文件转换成为加密保护的安全文件

常用的子命令包括::

    boot    升级当前系统为安全系统
    make    生成各种安全文件
    deploy  发布安全系统

运行 ``btarmor <command> -h`` 可以查看各个命令的详细使用方法。

.. _btarmor boot:

btarmor boot
------------

子命令 ``boot`` 用于安装安全操作系统 :ref:`btarmor-os`

直接运行下面的命令会安装新的安全内核，这个命令需要使用 ``root`` 权限，一般用于初
始化安全操作系统::

    sudo btarmor boot

安装完成之后需要重启系统。重新启动之后，内核就升级成为安全操作系统 :ref:`btarmor-os`

下面的命令则显示安全系统的相关信息::

    btarmor boot --status

.. _btarmor make:

btarmor make
------------

子命令 ``make`` 用于将可执行文件，动态库和数据文件等，转换成为 Bootarmor 保护的
安全文件，转换后生成的文件是经过加密处理的，只能在 :ref:`btarmor-os` 系统上运行。

`btarmor make`_ 可以运行在 :ref:`btarmor-os` 系统，直接将应用程序转换成为安全应
用。

`btarmor make`_ 也可以运行在普通的 Linux 系统，将应用程序转换成为安全应用，然后
再把安全应用发布到 :ref:`btarmor-os` 系统中。

语法
~~~~

.. code-block:: console

    btarmor make <options> <PATTERN>

选项
~~~~

-i, --inplace			生成的文件直接覆盖原来的文件
-O, --output PATH		生成的文件保存在另外一个路径，默认是 `dist`
-sys, --share			使用共享模式进行保护，一般用于保护系统动态库
-sh, --safe-heap		不允许内核访问应用程序申请的堆空间
-ss, --safe-stack               不允许内核访问运行栈（局部变量）
-f FILE				从文件里面读取需要处理的文件名称

描述
~~~~

子命令 make 用于将命令行列出的一个或者多个文件转换成为安全文件，可以使用通配符。

例如::

    cd /home/jondy/myapp
    btarmor make foo *.so config.txt

还可以把所有需要处理的文件名称都存放到一个文件里面，例如 ``files.list`` ::

    foo
    libtoo.so
    config.txt

然后在命令行使用 ``-f`` 指定这个文件，例如::

    cd /home/jondy/myapp
    btarmor make -f files.list

默认情况下生成的新文件会保存在另外一个输出目录 ``dist`` ，使用选项 ``--output``
可以指定输出目录。

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

示例
~~~~

* 转换目录 ``foo`` 下面的所有文件，保存生成的安全文件到到默认输出目录 ``dist``::

    btarmor make foo/*

* 只转换两个动态库文件，转换之后直接覆盖原来的文件::

    btarmor make -i dist/*.so

* 没有使用任何内核服务的程序，对内存堆和运行栈也进行保护::

    btarmor make -sh -ss myapp

* 加密保护数据文件到 ``dist/data.txt``::

    btarmor make data.txt

* 使用共享模式加密系统动态库::

    sudo btarmor make -i --share /usr/lib/*.so

* 加密系统命令 ``ls``::

    sudo btarmor make -i -sys /usr/bin/ls

.. _btarmor deploy:

btarmor deploy
--------------

用于产品开发完成之后，发布运行安全应用的系统到最终用户。

这个命令仅适用于企业版，社区版没有提供此功能。

语法
~~~~

.. code-block:: console

    btarmor deploy <options>

选项
~~~~

-s, --senior		安全内核使用高级模式
-e, --exclusive		安全内核使用专用模式

描述
~~~~

在产品准备交付给用户之前，需要使用该命令把内核功能进行加壳固化，清理开发辅助的文
件等。

选项 ``--senior`` 可以启用安全内核的高级模式。启用高级模式之后，Linux内核将不能
访问安全应用的内存，如果需要访问，必须得到安全应用的授权。这样能提高安全性，但是
也需要更多的了解操作系统的内存管理，ELF文件的组成，以及编译器的工作原理等相关知
识。否则，安全应用可能无法正常运行。

选项 ``--exclusive`` 可以启用安全内核的专用模式。启用专用模式之后，则普通可执行
文件，包括系统命令都无法执行，只有加密后的安全应用可以运行，这样可以进一步提高安
全行。

.. _btarmor patch:

btarmor patch
-------------

下载安全内核的源代码补丁，用于定制自己的安全内核。

语法
~~~~

.. code-block:: console

    btarmor patch <options> [VERSION]

选项
~~~~
-O, --output PATH		保存到指定目录，默认是 patches
-l, --list			列出所有可用版本的补丁名称

描述
~~~~

直接下载和当前系统内核版本相同的补丁，保存到目录 `patches/`::

  btarmor patch

下载应用于内核版本 `5.10.73` 的补丁::

  btarmor patch 5.10.73

查看所有支持的内核版本的补丁::

  btarmor patch --list

也可以从下面的链接查看和下载对应版本的 :ref:`btarmor-os` 补丁

    https://btarmor.dashingsoft.com/kernel/patches/

.. include:: _common_definitions.txt
