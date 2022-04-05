基本使用教程
============

下面以树莓派为例说明用户如何使用 Bootarmor 发布和保护应用程序

1. 启动树莓派，然后修改 `/etc/apt/source.list` ，增加一行::

     deb http://btarmor.dashingsoft.com/btarmor bullseye main

2. 更新安装源::

     sudo apt update

3. 安装命令行工具 :ref:`btarmor`::

     apt get btarmor-cli

4. 创建安全内核::

     btarmor sys

5. 重新启动树莓派，现在已经是安全操作系统。

6. 转换被保护的应用程序 ``/opt/foo`` 为安全应用::

     btamor make -i /opt/foo

至此，安全应用 ``/opt/foo`` 已经受到保护，可以把树莓派发布给用户使用。

.. include:: _common_definitions.txt
