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


.. include:: _common_definitions.txt
