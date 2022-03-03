基本使用教程
============

下面以树莓派为例说明用户如何发布安全的应用

1. 按照正常方式编译生成一个应用程序，包括可执行文件，动态库以及数据文
   件等，保存在包 ``foo-1.0.tar.gz`` 里面。

2. 参考 :ref:`下载和安装` 在开发机器上安装命令行工具 :ref:`btarmor`

3. 在开发机器上使用命令行工具 :ref:`btarmor` 创建安全应用::

       tar xzf foo-1.0.tar.gz
       btarmor make -i foo-1.0/
       tar czf foo-armor-1.0.tar.gz foo-1.0/

4. 参考 :ref:`下载和安装` 安装 Bootarmor OS 到树莓派并启动树莓派

5. 拷贝安全应用包到树莓派::

       scp foo-armor-1.0.tar.gz pi@raspberrypi.local:/home/pi

6. 在任意终端上执行命令，使用 ssh 方式进入树莓派控制台::

       ssh pi@raspberrypi.local

   输入密码之后进入树莓派控制台，下面的命令均在树莓派控制台运行

5. 在树莓派上发布安全应用到 ``/opt/foo``::

       mkdir -p /opt
       tar xzf /home/pi/foo-armor-1.0.tar.gz /opt

至此安全应用已经按照到 ``/opt/foo-1.0`` 下面，里面的所有文件，包括可执行文件，动态库
以及数据文件都是受到保护的。

.. include:: _common_definitions.txt
