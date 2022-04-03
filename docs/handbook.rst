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
-nr, --no-rodata		不保护只读数据段，默认情况下是没有被保护的
-sb, --safe-bss			保护没有初始化的全局变量，默认情况下动态数据段是没有被保护的
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

选项 ``--no-data`` 一般用于静态数据段需要被系统内核访问的情况，如果使用默认选项
加密的可执行文件运行的时候出现保护错误，可以尝试使用该选项。静态数据段（.data）
主要存放有初始值的全局变量，如果只是个别全局变量需要被内核访问，可以在源代码里面
使用宏修饰符 `BTSHARE` 声明变量。

选项 ``--no-rodata`` 用于保护只读的数据段，默认情况下只读数据段是不允许内核访问
的。只读数据段（.rodata）一般存放字符串常量等，如果内核需要访问很多字符串常量，
可以考虑使用该选项进行加密。如果只是个别字符串常量需要被内核访问，可以在源代码里
面使用宏修饰符 `BTSHARE` 声明常量。

选项 ``--safe-bss`` 用于没有初始值的全局变量。默认情况下没有初始值的全局变量存放
在 .bss 节中，是允许内核访问的。如果确认这些全局变量不需要被内核访问，可以使用该
选项提高安全性。

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
* 静态数据段，有初始值的全局变量，一般存放在 .data 节
* 动态数据段，没有初始值的全局变量，一般存放在 .bss 节
* 字符串常量段，一般存放在 .rodata 节
* 内存栈，存放局部变量
* 内存堆，程序申请的内存
* 数据文件

受保护内存里只允许应用程序本身访问，不允许任何外部访问，包括 Linux 内核，所以提
供了最大限度的安全性。当然这样也意味在调用系统服务的时候，如果这些数据需要被内核
访问，就需要进行一些额外的设置和处理。

默认情况下，代码段，静态数据段，字符串常量段是被保护的，而其它部分是没有被保护的。

例如，在默认编译链接选项下面

.. code:: c

   #include <fcntl.h>
   #include <stdio.h>
   #include <stdlib.h>

   // 没有初始值的全局变量存放在动态数据段 .bss 节，没有被保护
   int ga;
   static int sa;

   // 有初始值的全局变量存放在静态数据段 .data 节，受到保护
   int gb = 1;
   static int sb = 2;

   // 有初始值的全局变量 gc 存放在静态数据段 .data 节，受到保护
   // 字符串常量 "abc" 存放在只读数据段 .rodata 节，受到保护
   char *gc = "abc";

   int main(int argc, char *argv[])
   {
       // 局部变量存放在内存栈中，没有被保护
       int a;

       // 局部变量 b 存放在内存栈中，没有被保护
       // 字符串常量 "efg" 存放在只读数据段 .rodata 节，受到保护
       char *b = "efg";

       // 没有初始值的静态全局变量存放在 .bss 节，没有被保护
       static int d;

       // 有初始值的静态全局变量存放在 .data 节，受到保护
       static int e = 2;

       // 下面的所有可执行代码都受到保护
       d = ga + sa + gb + sb + a + d + e;

       printf("Got result %d\n", d);

       return 0;
   }

对于使用 C 语言开发的可执行文件来说，如果使用默认保护模式，那么不需要修改源代码。

默认保护模式的基本使用步骤

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

如果需要使用非默认的保护模式，则需要对源代码进行一定的修改，下面分别说明如何定制
各种保护模式。

共享字符串
~~~~~~~~~~

因为字符串默认是处于保护模式，无法被内核访问，而系统调用 ``open`` 需要把文件名称
传递到内核，下面这个示例中默认模式下是无法正常运行的

.. code:: c

   #include <fcntl.h>
   #include <stdio.h>
   #include <stdlib.h>

   char *filename1 = "data1.txt";

   int main(int argc, char *argv[])
   {
       int fd;
       char *filename2 = "data2.txt";

       // 出错
       fd = open("data.txt", 0);
       if (fd > 0)
           close(fd);

       // 出错
       fd = open(filename1, 0);
       if (fd > 0)
           close(fd);

       // 出错
       fd = open(filename2, 0);
       if (fd > 0)
           close(fd);

       return 0;
   }

1. 共享全部字符串

如果要共享所有的字符串，包括其它所有存放在 .rodata 的数据，那么可以在生成安全应
用的时候使用选项 ``--no-rodata``:

    btarmor make --no-rodata myapp

2. 共享个别字符串

如果只是个别字符串需要被内核访问，可以在函数中使用 ``BTS`` 声明字符串常量。被声
明为共享的字符串，会被存放到一个特殊的数据段中，这个数据段允许内核访问，从而避免
数据共享问题。这种方式是把字符串拷贝到一个共享缓冲区中，因为该缓冲区是全局的，所
以但是这种方式无法应用于多线程中。例如

.. code:: c

   #include <fcntl.h>
   #include <stdio.h>
   #include <stdlib.h>

   #include "btapp.h"

   int main(int argc, char *argv[])
   {
       int fd;
       fd = open(BTS("data.txt"), 0);
       if (fd > 0)
           close(fd);
       return 0;
   }

保护动态数据段
~~~~~~~~~~~~~~

没有进行初始化的全局变量都会被链接器放入到二进制文件的 .bss 节里面，而 .bss 节里
面的内容默认情况下是没有被保护。

如果需要保护单个变量，只需要对其赋任意初值就可以。

如果要保护整个动态数据段，可以在生成安全应用的时候使用选项 ``--safe-bss``:

    btarmor make --safe-bss myapp

也可以通过在主程序调用 ``BTPROT_BSS`` 来实现

.. code:: c

   #include "btapp.h"

   int ga;

   int main(int argc, char *argv[])
   {
       BTPROT_BSS();
       ga = 0;
       return ga;
   }

保护内存栈
~~~~~~~~~~

默认情况下内存栈没有进行额外保护，如果需要保护内存栈，可以在生成安全应用的时候使
用选项 ``--safe-stack``:

    btarmor make --safe-stack myapp

如果需要单独保护某一个函数的堆栈，可以在函数开始的时候调用 ``BTPROT_STACK`` 来实
现

.. code:: c

   #include "btapp.h"

   int foo(void)
   {
       BTPROT_STACK();
       return 0;
   }

   int main(int argc, char *argv[])
   {
       return foo();
   }

.. note::

   目前保护内存栈暂时还没有实现。

保护内存堆
~~~~~~~~~~

如果需要对内存堆的进行保护，那么需要对源代码进行相应的修改，使用 ``btapp.h`` 中
提供的宏定义 ``BTMALLOC`` 等来进行内存管理。例如，

.. code:: c

   #include <fcntl.h>
   #include <stdio.h>
   #include <stdlib.h>

   #include "btapp.h"

   int main(int argc, char *argv[])
   {
       int fd;

       // 使用 BTMALLOC 申请被保护内存
       char *ps = (char*)BTMALLOC(PAGE_SIZE);

       // 把文件名保存在安全内存中
       strncpy(ps, "data.txt", PAGE_SIZE);

       // 下面的语句中 open 会调用系统功能，Linux 内核会访问 *ps
       // 而 *ps 是受到保护的， 不允许的内核访问，所以会出错
       fd = open(ps, O_RDONLY);

       if (fd > 0)
           close(fd);

       free(ps);

       return 0;
   }

申请保护的内存必须是页面对齐的。

保护数据文件
~~~~~~~~~~~~

如果需要对数据文件的进行保护，那么需要对源代码进行相应的修改，必须使用 ``mmap``
的方式来读写文件，而不能使用 ``fread`` 和 ``fwrite`` 等方式读写文件。

首先对数据文件进行加密，下面使用命令 :ref:`make` 对数据文件 ``data.txt`` 进行加
密，加密后的文件保存为 ``dist/data.txt``

    btarmor make data.txt

然后使用 ``mmap`` 分配内存，同时指定标志 ``MAP_BTARMOR`` 。 下面是一个简单的示例

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
