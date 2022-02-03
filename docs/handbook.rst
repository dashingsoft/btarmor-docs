用户使用手册
============

Bootarmor 提供了一个额外的命令行工具 `btarmor`_ ，用于转换应用程序为安全应用。

.. _btarmor:

btarmor
-------

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

对于使用 C 语言开发的应用程序来说，可以使用 btarmor 提供的预定义宏代码来实现下列
功能:

* 定义安全数据段
* 请求保护数据内存

安全段和受保护内存里只允许应用程序本身访问，不允许任何外部访问，包括 Linux 内核，
所以只能把那些不需要系统调用服务的数据放入到其中，从而最大限度的保护数据安全。

使用方法是把 btarmor 的头文件包含进行，然后使用宏代码声明变量和申请内存，例如

.. code:: c

   #include "btarmor.h"

   int password __BTA_SECTION;

   int main(int argc, char *argv[])
   {
       char *p = (char*)BTA_MALLOC(PAGE_SIZE);
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

申请保护的内存必须是页面对齐的

.. include:: _common_definitions.txt
