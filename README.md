# Bootarmor

一种基于 Linux 的嵌入式操作系统，为用户应用程序的代码提供绝对安全，确
保其不被使用者获取

## man.1

使用下面的命令生成 btarmor.1

    cd docs
    rst2man.py btarmor-cli.rst btarmor.1
    rst2man.py --lang zh_CN btarmor-cli.rst btarmor.1

查看生成的文件，注意看本地文件必须在路径中包含 `/`

    man ./btarmor.1
