Bootarmor 用户文档
==================

:版本: 0.1
:主页: |Homepage|
:联系方式: jondy.zhao@gmail.com
:作者: 赵俊德

Bootarmor 是以提供绝对安全为目标的一种操作系统，基于 Linux，结合不同 CPU 的特性，
从系统引导开始接管所有内存管理，从而为用户提供一个安全运行环境。

用户通过命令行命令，可以将自己的应用程序转换成为安全应用，安全应用可以运行在安全
系统 Bootarmor 中。运行在 Bootarmor 系统中的安全应用的代码和数据，操作系统中任何
权限的用户（包括 root 在内）也无法读取和访问，无论是静态反编译，还是各种内核调试
器和应用层调试器，都无法获取安全应用的代码和数据。

本文档适用于使用 Bootarmor 来保护自己应用程序的用户。

内容:

.. toctree::
   :maxdepth: 2

   introduction
   tutorial
   btarmor-cli
   handbook
   appendix

.. include:: _common_definitions.txt
