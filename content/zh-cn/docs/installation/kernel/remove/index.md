---
title: "删除内核"
linkTitle: "删除内核"
weight: 30
description: >
  删除不再需要的 debian 核心
---

多次升级之后，系统内就会累计有多个内核版本，可以考虑删除旧的不用的内核。

参考：

https://askubuntu.com/questions/1253347/how-to-easily-remove-old-kernels-in-ubuntu-20-04-lts

```bash
mkdir -p ~/work/soft/debian
cd ~/work/soft/debian

vi remove_old_kernels.sh
```

新建一个文件内容如下：

```bash
#!/bin/bash
# Run this script without any param for a dry run
# Run the script with root and with exec param for removing old kernels after checking
# the list printed in the dry run

uname -a
IN_USE=$(uname -a | awk '{ print $3 }')
if [[ $IN_USE == *-generic ]]
then
  IN_USE=${IN_USE::-8}
fi
echo "Your in use kernel is $IN_USE"

OLD_KERNELS=$(
    dpkg --list |
        grep -v "$IN_USE" |
        grep -v "linux-headers-generic" |
        grep -v "linux-image-generic"  |
        grep -Ei 'linux-image|linux-headers|linux-modules' |
        awk '{ print $2 }'
)
echo "Old Kernels to be removed:"
echo "$OLD_KERNELS"

if [ "$1" == "exec" ]; then
    for PACKAGE in $OLD_KERNELS; do
        yes | apt purge "$PACKAGE"
    done
fi
```

执行

```bash
bash ./remove_old_kernels.sh
```

看查看到要删除的内核版本和相关的包，确认没有问题之后再通过

```bash
sudo bash ./remove_old_kernels.sh exec
```

进行实际删除。

之后重启，再检查现有的内核是否符合预期。


