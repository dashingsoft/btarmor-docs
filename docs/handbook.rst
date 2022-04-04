================
 C 用户使用手册
================

本章详细说明了如何保护 C 语言开发的应用程序，适用于 C 开发人员，详细了
解如何使用 :ref:`btarmor` 保护的 C 应用程序的各个组成部分。

默认保护模式
============

对于 C 开发的应用程序来说，基本的保护包括

* 代码段
* 数据段，存放全局变量和函数体内使用 ``static`` 声明的变量
* 字符串常量
* 内存堆，程序申请的内存
* 内存栈，存放局部变量
* 数据文件

受保护内存只允许应用程序本身访问，不允许任何外部访问，包括 Linux 内核，所以提供
了最大限度的安全性。当然这样也意味在调用系统服务的时候，如果这些数据需要被内核访
问，就需要进行一些额外的设置和处理。

默认情况下，代码段，数据段，字符串常量和内存堆是被保护的，而内存栈和数据文件是
没有被保护的。

例如，在默认编译链接选项下面

.. code:: c

   #include <fcntl.h>
   #include <stdio.h>
   #include <stdlib.h>

   // 全局变量存放被保护
   int ga;
   int gb = 1;
   static int sa;
   static int sb = 2;

   // 字符串常量 "abc" 也受到保护
   char *gc = "abc";

   int main(int argc, char *argv[])
   {
       // 局部变量存放在内存栈中，没有被保护
       int a;

       // 字符串常量 "efg" 受到保护
       char *b = "efg";

       // 使用 static 声明的变量受到保护
       static int d;
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

如果需要使用非默认的保护模式，则需要对源代码进行一定的修改，这时候需要导入的头文
件 ``btapp.h`` ，并在源代码中使用相应的宏，来实现需要的功能。

共享字符串和全局变量
====================

当内核需要访问字符串或者全局变量的时候，因为默认保护模式下它们受到保护，直接访问
会出错。例如系统调用 ``open`` 需要把文件名称传递到内核，下面这个示例中默认模式下
是无法正常运行的，因为字符串常量默认是处于保护模式，无法被内核访问。

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

这时候需要修改源代码，可以在函数中使用 ``BTS`` 声明字符串常量。这种字符串会被存
放到当前函数的运行栈中，而运行栈默认情况是允许内核访问的，从而避免内核无法访问字
符串的问题。下面的例子就可以解决上面的问题

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

宏 ``BTS`` 可以应用于字符串常量，字符串数组，结构体等，但是不能用于字符串指针

.. code:: c

    char arr[10];
    struct {
        int a;
        int b;
    } stv;

    BTS("hello");
    BTS(arr);
    BTS(stv);

    char *s = "hello";
    BTS(s);               // 错误

宏 ``BTS`` 只支持内核读取数据，并不支持内核回写数据。如果需要内核写回数据到共享
变量，那么需要使用一组宏 ``BTPS`` 和 ``BTPE`` 。在需要共享的时候调用 ``BTPS`` ，
结束共享调用 ``BTPE`` 。

例如，

.. code:: c

    // 全局变量 gt 受到保护，内核无法直接访问
    struct T {
        int a;
        int b;
    } gt = { 1, 2 };

    // 20 号系统调用需要修改 gt 里面的数据，原来的调用方式
    // syscall(20, &gt);

    // 使用 BTPS/BTPE 向内核共享可写的全局变量
    BTPS(&gt);
    syscall(20, &gt);
    BTPE(&gt);

.. note::

   宏 ``BTS`` 和 ``BTPS/BTPE`` 都使用通过把数据拷贝到运行栈来实现和内核共享，如
   果应用程序使用 ``--safe-stack`` 对运行栈进行了保护，那么这些宏是无法共享数据
   到内核的。在这种情况下，可以参考下面的 :ref:`共享内存堆` 的方式，把变量所在的
   内存区域作为堆空间来进行共享。

共享内存堆
==========

如果内核需要访问内存堆中的数据，那么需要使用宏定义 ``BTPUB`` 实现对内核的共享。
如果需要重新对内存堆进行保护，那么可以在使用完成之后调用宏 ``BTPRI`` 重新保护指
定的内存堆空间。

例如，

.. code:: c

   #include <fcntl.h>
   #include <stdio.h>
   #include <stdlib.h>

   #include "btapp.h"

   int main(int argc, char *argv[])
   {
       int fd;
       ssize_t len = 1024;

       char *ps = (char*)malloc(len);
       strncpy(ps, "data.txt", len);

       // 下面的语句中 open 会调用系统功能，Linux 内核会访问 *ps
       // 而 *ps 是受到保护的， 不允许的内核访问，直接访问会出错

       BTPUB(ps, len);
       fd = open(ps, O_RDONLY);
       BTPRI(ps, len);

       if (fd > 0)
           close(fd);

       return 0;
   }

.. note::

   使用 ``BTPUB`` 会以页面单位进行共享，起始地址分别会和页面对齐，所有地址范围之
   内的页面都会被共享，所以可能会有额外的堆空间被共享。

保护内存栈
==========

默认情况下内存栈没有进行额外保护，如果需要保护内存栈，可以在生成安全应用的时候使
用选项 ``--safe-stack``::

    btarmor make --safe-stack myapp

保护数据文件
============

如果需要对数据文件的进行保护，那么需要对源代码进行相应的修改，必须使用 ``mmap``
的方式来读写文件，而不能使用 ``fread`` 和 ``fwrite`` 等方式读写文件。

首先对数据文件进行加密，下面使用命令 :ref:`make` 对数据文件 ``data.txt`` 进行加
密，加密后的文件保存为 ``dist/data.txt``::

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
