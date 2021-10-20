Bootarmor 用户文档
==================

:版本: 0.1
:主页: |Homepage|
:联系方式: jondy.zhao@gmail.com
:作者: 赵俊德

Bootarmor 是以提供绝对安全为目标的一种操作系统，基于 Linux，结合不同 CPU 的特性，
从系统引导开始接管所有内存管理，从而为用户提供一个安全运行环境。

用户只需要使用一条简单的命令，就可以将自己的应用程序转换成为安全应用，并运行在安
全环境 Bootarmor 系统中。运行在 Bootarmor 环境中的安全应用的代码和数据，任何权限
的用户（包括 root 在内）也无法读取。不管是静态反编译，还是动态调试，都无法获取安
全应用的代码和数据。

本文档适用于使用 Bootarmor 来保护自己应用程序的用户。

内容:

.. toctree::
   :maxdepth: 2

   introduction
   installation
   tutorial
   handbook
   appendix

.. include:: _common_definitions.txt
