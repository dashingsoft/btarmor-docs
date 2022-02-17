用户使用手册
============

本章节主要包含下列内容

* 命令行工具 `btarmor`_ 的用户手册，适用于使用 `btarmor`_ 命令，需要了解其更多高
  级功能的用户。

* C 程序开发保护手册，适用于 C 开发人员，详细了解如何使用 Bootarmor 保护的 C 应
  用程序的各个组成部分。

.. _btarmor:

btarmor
-------

Bootarmor 提供了一个额外的命令行工具 `btarmor`_ ，用于转换应用程序为安全应用。

命令行工具，用于转换应用程序为安全应用。

**语法**::

    btarmor <options> FILENAME...

**选项**

-i, --overwrite                 将转换后的安全应用直接覆盖原来的输入文件
-o, --output                    输出的安全应用名称
-k, --key HEXSTRING             指定用于加密应用的键

**描述**


C 程序开发保护手册
------------------

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

如果需要对数据文件的进行保护，那么需要对源代码进行相应的修改，不能使用 `malloc`
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
