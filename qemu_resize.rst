修改虚拟机镜像大小(qcow2/raw resize)
##########################################################################################################################################
:date: 2011-08-31 05:38:41
:category: 系统运维
:tags: kvm, 虚拟化, 虚拟机
:slug: qemu_resize

不多说，直接看

创建一个镜像文件，大小1G，格式是qcow2

.. code-block:: bash

    muxueqz@muxueqz /tmp $ qemu-img create -f qcow2 t.qcow2 1G
    Formatting 't.qcow2', fmt=qcow2 size=1073741824 encryption=off
    cluster\_size=0

查看镜像文件实际占用空间

.. code-block:: bash

    muxueqz@muxueqz /tmp $ ls -alh t.qcow2
     -rw-r--r-- 1 muxueqz muxueqz 193K 8月 31 13:18 t.qcow2

查看qcow2信息

.. code-block:: bash

    muxueqz@muxueqz /tmp $ qemu-img info t.qcow2
     image: t.qcow2
     file format: qcow2
     virtual size: 1.0G (1073741824 bytes)
     disk size: 136K
     cluster\_size: 65536

加1G(resize +)

.. code-block:: bash

    muxueqz@muxueqz /tmp $ qemu-img resize t.qcow2 +1G
     Image resized.

再查看qcow2信息

.. code-block:: bash

    muxueqz@muxueqz /tmp $ qemu-img info t.qcow2
     image: t.qcow2
     file format: qcow2
     virtual size: 2.0G (2147483648 bytes)
     disk size: 140K
     cluster\_size: 65536

减1G(resize -)

.. code-block:: bash

    muxueqz@muxueqz /tmp $ qemu-img resize t.qcow2 -- 2G
     qemu-img: This image format does not support resize

**!!!qcow2只能加不能减!**
再查看qcow2信息

.. code-block:: bash

    muxueqz@muxueqz /tmp $ qemu-img info t.qcow2
     image: t.qcow2
     file format: qcow2
     virtual size: 2.0G (2147483648 bytes)
     disk size: 140K
     cluster\_size: 65536

果然没变

再试试改变raw格式的大小(resize raw)

同样先创建1G大小的raw

.. code-block:: bash

    muxueqz@muxueqz /tmp $ qemu-img create -f raw t.raw 1G
     Formatting 't.raw', fmt=raw size=1073741824
     muxueqz@muxueqz /tmp $ qemu-img info t.raw
     image: t.raw
     file format: raw
     virtual size: 1.0G (1073741824 bytes)
     disk size: 1.0M

可以看出raw要比qcow2多占一些空间。

加1G

.. code-block:: bash

    muxueqz@muxueqz /tmp $ qemu-img resize t.raw -- +1G
     Image resized.
     muxueqz@muxueqz /tmp $ qemu-img info t.raw
     image: t.raw
     file format: raw
     virtual size: 2.0G (2147483648 bytes)
     disk size: 2.0M

减1G(resize -)

.. code-block:: bash

    muxueqz@muxueqz /tmp $ qemu-img resize t.raw -- -1G
     Image resized.
     muxueqz@muxueqz /tmp $ qemu-img info t.raw
     image: t.raw
     file format: raw
     virtual size: 1.0G (1073741824 bytes)
     disk size: 1.0M


