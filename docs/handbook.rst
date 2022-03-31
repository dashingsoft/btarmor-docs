==============
 用户使用手册
==============

本章节主要包含下列内容

* 命令行工具 `btarmor`_ 的用户手册，适用于使用 `btarmor`_ 命令，需要了解其更多高
  级功能的用户。

* C 程序开发保护手册，适用于 C 开发人员，详细了解如何使用 Bootarmor 保护的 C 应
  用程序的各个组成部分。

.. _btarmor:

btarmor
=======

Bootarmor 提供了一个额外的命令行工具 `btarmor`_ ，用于转换应用程序为安全应用。

`btarmor` 的语法格式::

    btarmor <command> [options]

常用的命令包括::

    make    创建 btarmor 保护的安全文件
    get     下载 btarmor-os 的系统映像

可以运行 `btarmor <command> -h` 查看各个命令的详细使用方法。

.. _make:

make
~~~~

用于将可执行文件，动态库和数据文件等，转换成为 btarmor 保护的文件，转换后生成的
文件是经过加密的，无法在普通系统中运行，只能在 btarmor-os 系统上运行。

**语法**::

    btarmor make <options> PATH...

    btmake <options> PATH...

.. _make 命令选项:

**选项**

-i, --inplace			生成的文件直接覆盖原来的文件
-O, --output PATH		生成的文件保存在另外一个路径，默认是 `dist`
-sys, --share			使用共享模式进行保护，一般用于保护系统动态库和允许内核访问的文件
-nd, --no-data			不保护运行时刻的数据段，默认情况下数据段是被保护的
-sr, --safe-rodata		保护只读数据段，默认情况下只读数据段是没有被保护的
-ss, --safe-stack		保护运行栈（局部变量），默认情况下运行栈是没有被保护的

**描述**

make 子命令用于将命令行列出的一个或者多个文件转换成为安全文件，如果列出的是目录，
那么会递归处理目录下面的所有文件。

最常用的方式是在执行命令之前，把需要处理的可执行文件，动态库，需要加密的数据文件
拷贝到一个目录，然后直接处理该目录。

默认情况下生成的新文件会保存在另外一个输出目录 ``dist`` ，使用选项 ``--output``
可以指定输出目录。

如果指定了选项 ``--inplace`` ，那么输出文件会直接覆盖原来的文件。

选项 ``--share`` 一般用于加密系统动态库，使用该选项加密的文件允许内核访问。

选项 ``--no-data`` 一般用于数据段需要被系统内核访问的情况，如果使用默认选项加密
的可执行文件运行的时候出现保护错误，可以尝试使用该选项。如果确认是数据段保护问题，
那么需要修改代码，将被保护的全局变量声明为共享属性（使用宏定义 `BTSHARE_DATA`)。

选项 ``--safe-rodata`` 用于保护只读的数据段，默认情况下只读数据段是允许内核访问
的。如果应用程序确认没有只读数据被内核访问，可以使用该选项提高安全性。

选项 ``--safe-stack`` 用于保护运行栈，也就是各种局部变量。默认情况下局部变量是允
许内核访问的。如果确认程序没有任何局部变量被内核访问，可以使用该选项提高安全性。

**示例**

* 转换目录 ``foo`` 下面的所有文件，保存生成的安全文件到到默认输出目录 ``dist``::

    btarmor make foo/

* 只转换两个动态库文件，转换之后直接覆盖原来的文件::

    btarmor make -i dist/foo.so dist/too.so

* 没有使用任何内核服务的程序，使用最高安全选项进行加密::

    btarmor make -sr -ss myapp

* 有很多全局变量需要内核访问，并且也没有敏感数据，所以不保护数据::

    btarmor make -nd myapp

.. _get:

get
~~~

下载 btarmor 的系统映像，下载的映像可以被写入嵌入式系统的。

**语法**::

    btarmor get <options> NAME

    btget <options> NAME

.. _get 命令选项:

**选项**

-l, --list			列出所有可用的系统映像名称
-t, --timeout SECONDS		如果下载的时候出现超时错误，设置该选项为较大的值

**描述**

查看并下载可用的系统映像，下载的映像文件默认保存在用户目录的子目录 ``.btarmor/images``

**示例**

* 列出当前所有的可用系统映像::

    btarmor get --list

* 下载树莓派 pi4 的系统映像::

    btarmor get raspi4

C 程序开发保护手册
==================

对于 C 开发的应用程序来说，基本的保护包括

* 代码段
* 数据段，存放全局变量
* 内存栈，存放局部变量
* 内存堆，程序申请的内存
* 数据文件

受保护内存里只允许应用程序本身访问，不允许任何外部访问，包括 Linux 内核，所以提
供了最大限度的安全性。当然这样也意味在调用系统服务的时候，如果这些数据需要被内核
访问，就需要进行一些额外的设置和处理。

保护代码段和数据段
~~~~~~~~~~~~~~~~~~

对于使用 C 语言开发的可执行文件来说，默认情况下数据段和代码段都是受到保护的，不
需要额外的处理。基本使用步骤

1. 正常编译一个可执行文件::

     gcc -o myapp myapp.c

2. 使用 :ref:`btarmor` 转换成为安全应用::

     btarmor make -i myapp

对于动态库，和可执行文件的保护是一样的。下面的例子是一个包含可执行文件和私有动态
库的应用程序的基本的使用步骤

1. 正常编译动态库和可执行文件到发布目录 `dist`::

     mkdir dist
     gcc -o dist/myapp myapp.c
     gcc -shared -o dist/mydll.so mydll.c

2. 使用 :ref:`btarmor` 转换整个目录下面的文件为安全文件::

     btarmor make -i dist/

共享数据段
~~~~~~~~~~

如果全局变量需要被内核访问，那么就需要声明为 `BTSHARE_DATA` ，否则内核会报错。

被声明为共享的全局变量，会被存放到一个特殊的数据段中，这个数据段允许内核访问，从
而避免数据共享问题。

例如，下面这个示例中全局变量 `msg` 是一个共享变量，调用内核服务的时候需要访问

.. code:: c

   #include "btapp.h"

   char *msg BTSHARE_DATA = "It's in Linux kernel";

   int main(int argc, char *argv[])
   {
       printf("%s\n", msg);
       return 0;
   }

保护内存栈
~~~~~~~~~~

默认情况下内存栈没有进行额外保护。

保护内存堆
~~~~~~~~~~

如果需要对内存堆的进行保护，那么需要对源代码进行相应的修改，不能使用 `malloc`
等直接分配内存，而是需要使用 `btapp.h` 提供的宏定义来 `bt_malloc` 等来实现。

使用方法是把 btarmor 的头文件包含进来，然后使用宏代码声明变量和申请内存，例如

.. code:: c

   #include "btapp.h"

   int main(int argc, char *argv[])
   {
       // 使用 BT_MALLOC 申请被保护内存
       char *p = (char*)BT_MALLOC(PAGE_SIZE);
       char c;

       // 下列语句都没有问题
       printf("Address is %p\n", p);
       *p = 'a';
       c = *p;

       // 下面的语句会出现保护错误，因为 printf 会调用系统功能，
       // Linux 内核会访问 *p ，这样就不允许的
       printf("Value is %c\n", *p);

       return 0;
   }

申请保护的内存必须是页面对齐的。

保护数据文件
~~~~~~~~~~~~

如果需要对数据文件的进行保护，那么需要对源代码进行相应的修改，不能使用 `fread`
和 `fwrite` 的方式读写文件，而必须使用 `mmap` 的方式，这些可以通过 `btapp.h`
中提供的宏定义来实现。

例如

.. code:: c

   #include "btapp.h"

   int main(int argc, char *argv[])
   {
       char *fmap;
       char buf[200];
       int size = 100;
       int offset = 0;

       int fd = open("data.txt", 0);
       if (fd < 0) {
           perror("open file");
           return -1;
       }

       fmap = mmap(0, size, PROT_READ, MAP_PRIVATE | MAP_BTARMOR, fd, offset);
       if (fmap == MAP_FAILED) {
           perror("map file");
           close(fd);
           return -1;
       }
       memcpy(buf, fmap, size);
       buf[size] = 0;

       printf("file %d bytes: %s\n", size, buf);

       close(fd);
       return 0;
   }

.. include:: _common_definitions.txt
