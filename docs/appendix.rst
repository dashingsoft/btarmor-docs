======
 附录
======

.. _btarmor-os:

btarmor-os
==========

`btarmor-os`_ 是一个基于 Debian Linux 安全操作系统。

`btarmor-os`_ 目前提供两个不同的版本:

* `Btarmor 社区版`_
* `Btarmor 企业版`_

Btarmor 社区版
~~~~~~~~~~~~~~

默认安装的版本是社区版，任何人可以自由下载使用。

Btarmor 企业版
~~~~~~~~~~~~~~

企业版需要注册之后得到授权文件，把授权文件部署在社区版上，就可以升级成
为企业版。

社区版和企业版的区别在于

* 社区版使用共享的相同的 Key
* 企业版使用私有的 Key
* 企业版的安全内核经过加壳来保证安全，社区版的安全内核没有经过加壳

Btarmor 内核模式
~~~~~~~~~~~~~~~~

* 高级模式
* 专用模式

高级模式的主要特征:

* 高级模式不允许 Linux 内核直接访问安全应用的内存。如果需要访问，必须
  得到安全应用的授权。

默认情况下，Linux 内核是可以访问安全应用的内存，高级模式能够提供更高的
安全性，但是也需要更多的了解操作系统的内存管理，ELF文件的组成，以及编
译器的工作原理等相关知识。否则，安全应用可能无法正常运行，或者即便运行，
也降低了安全性。

专用模式的主要特征:

* 只有经过加密的安全应用才可以运行

默认情况下，普通的可执行文件和动态库还可以继续使用，安全应用也可以正常
运行。但是在专用模式下面，普通的可执行文件和动态库都无法使用，必须转换
成为相应的安全文件才可以继续使用。

这两个模式可以同时启用，并不互斥。

安全应用
~~~~~~~~

使用 :ref:`btmake` 命令转换之后的应用程序称之为安全应用。

安全应用是经过加密后的应用程序。

安全应用的代码，数据和堆栈都是可以设置成为不允许内核访问的。

安全应用无法运行普通的 Linux 系统中，只能运行在 `btarmor-os`_ 中的安全层。

在同一个系统中，每一个安全应用使用自己独有的Key被加密。

认证应用
~~~~~~~~

使用 :ref:`btmake` 命令的同时指定选项 ``--sys`` ，这样转换之后的可执行
文件和动态库称之为认证应用和认证动态库。

认证应用无法运行普通的 Linux 系统，只能运行在 `btarmor-os`_ 中。

认证应用运行在应用层，而不是运行在安全层。

认证应用的代码和数据是能够被内核直接访问的。

认证动态库被安全应用调用的时候运行在安全层，被认证应用调用的时候运行在应用层。

其它用户无法修改认证应用。

在同一个系统中，所有的认证应用使用相同的Key被加密。

安全层
~~~~~~

运行安全应用的最高权限层。

内核层
~~~~~~

运行操作系统内核的次高权限层。

应用层
~~~~~~

运行一般应用程序的最低权限层。

Debian Packages
===============

btarmor 提供了下列 Debian 包

开发环境的功能包，目前支持 amd64 和 arm64 两种架构

* btarmor-cli 提供了命令行工具 btarmor ，大部分的开发相关的工作都可以使用这个命令完成
* btarmor-toolkit 提供了头文件和示例源文件，用于帮助生成具备高级保护功能的 c 应用程序

运行环境的包，主要包括两个

一是用于替换原来的 Linux 内核包:

* raspberrypi-btarmor (适用于 raspberry os，替换原来的 raspberrypi-kernel)
* linux-image-bt-arm64 (适用于 debian 系统，替换原来的 linux-image-arm64)
* linux-btarmor (适用于 ubuntu 系统，替换原来的 linux-generic)

二是用于装载安全应用和动态库的包，主要是替换原来的 ld.so 的功能，支持安全库的装载

* btarmor-runtime

.. include:: _common_definitions.txt
