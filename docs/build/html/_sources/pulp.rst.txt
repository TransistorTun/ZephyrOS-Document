Getting Started PULP
++++++++++++++++++++

Cài đặt Pulp Toolchain
======================

Prerequisites
_____________

.. code-block:: bash

    $ sudo apt-get install autoconf automake autotools-dev curl libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo gperf libtool patchutils bc zlib1g-dev

Getting the sources
___________________

.. code-block:: bash

    git clone https://github.com/pulp-platform/pulp-riscv-gnu-toolchain
    cd pulp-riscv-gnu-toolchain
    git submodule update --init --recursive

Installation (Pulp)
___________________

Cài đặt installation directory vào PATH
---------------------------------------

**Bước 1**:

Trong file ``~/.profile``, ở dòng cuối thêm:

.. code-block:: bash

    if [ -d "<INSTALL_DIR>" ] ; then
        PATH="<INSTALL_DIR>:$PATH"
    fi

Trong đó, ``<INSTALL_DIR>`` là nơi để ghi địa chỉ mới của thư
mục chứa installation.

.. warning::
    Nhớ log out hoặc restart lại máy sau khi thêm vào PATH.

**Bước 2**:

Tạo thư mục đó giống với thư mục trong ``~/.profile``. Theo hướng dẫn trên
github của pulp-riscv-gnu-toolchain thì các bạn có thể để vào ``/opt/riscv``,
nhớ tạo folder ``bin`` trong ``/opt/riscv/bin`` là được. Lưu ý, chúng ta chỉ
thêm ``/opt/riscv`` vào ``PATH`` còn folder ``bin`` thì không cho vào.

Chạy make
---------

Tại folder ``pulp-riscv-gnu-toolchain``, thực hiện lệnh sau:

.. code-block:: bash

    ./configure --prefix=<INSTALL_DIR> --with-arch=rv32imc --with-cmodel=medlow --enable-multilib
    make

Tại ``<INSTALL_DIR>`` thì viết cái địa chỉ phía trên của mình.

.. warning::
    Nếu ``<INSTALL_DIR>`` nằm trong root ``/`` như ``/opt/riscv`` thì phải sử dụng lệnh ``sudo make``.

Cài đặt Pulp SDK
================

Prerequisites
_____________

.. code-block:: bash

    sudo apt-get install -y build-essential git libftdi-dev libftdi1 doxygen python3-pip libsdl2-dev curl cmake libusb-1.0-0-dev scons gtkwave libsndfile1-dev rsync autoconf automake texinfo libtool pkg-config libsdl2-ttf-dev


.. code-block:: bash

    pip install argcomplete pyelftools prettytable


Getting the sources
___________________

.. code-block:: bash

    git clone https://github.com/pulp-platform/pulp-sdk.git
    cd pulp-sdk
    git submodule update --init --recursive

Cài đặt pulp-open.sh
____________________

Chúng ta sửa file ``pulp-sdk/configs/pulp-open.sh``, ở dưới dòng 32 thêm:


.. code-block:: bash

    export PULP_RISCV_GCC_TOOLCHAIN=<INSTALL_DIR> 

Trong đó, ``<INSTALL_DIR>`` ta thêm vào địa chỉ install đã sử dụng khi cài đặt
``pulp-riscv-gnu-toolchain``.

Build gvsoc
___________

Trong folder ``pulp-sdk``, ta nhập lệnh sau:

.. code-block:: bash

    source configs/pulp-open.sh
    make build

.. warning::
    Trong thư mục ``pulp-sdk`` trước khi build hay làm bất cứ thứ gì đều băt
    buộc phải chạy lệnh ``source configs/pulp-open.sh``.

Build ``tests/hello``
=====================

Trong folder ``pulp-sdk/tests/hello`` nhập lệnh sau:


.. code-block:: bash

    make clean all run


.. warning::
    Không nên thêm ``VERBOSE=1`` vào lệnh trên vì sẽ bị lỗi.


Show real code 
==============

.. code-block:: bash

    make dis

Open gtkwave
============

Build và tạo file VCD 

.. code-block:: bash

    make clean all run runner_args="--vcd"

Chạy file vcd

.. code-block:: bash

    gtkwave /home/pham_tuan/pulp-sdk/tests/perf/double_buffering/BUILD/PULP/GCC_RISCV/all.vcd /home/pham_tuan/pulp-sdk/tests/perf/double_buffering/view.gtkw


.. note::
    Thay thế "pham_tuan" thành tên user của bạn thì sẽ thành công

