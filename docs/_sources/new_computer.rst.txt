.. -*- coding: utf-8 -*-
.. _new-computer-label:

========================
Installing Ubuntu 19.04+
========================

Obtain a USB drive with Ubuntu
===============================

You need to somehow create a bootable USB stick. This is easiest in Ubuntu and there are many tutorials already, so I won’t clutter it up.

Install Ubuntu and drivers
==========================

Stick the USB in and boot up the computer. You need to change the boot order to boot off the USB stick first. This differs across operating systems; on our old Ubuntu 16.04s, we hit F10 to bring up Boot Options.

You can choose a normal installation or a minimal one. Out of convenience, we go with the normal installation; the only weirdly irritating thing it installs is Amazon. This shouldn’t take long – ours was under 10 min.

The computer will reboot after this.

.. note::

    If your screen flickers and dies, this is due to the graphics drivers not working properly. Go into Advanced Options and boot into Recovery mode.

**Do not reboot before updating drivers and setting up SSH.**

Updates
=======

Most of these should already be installed; check anyway.

.. code-block:: console

    sudo apt update
    sudo ubuntu-drivers autoinstall
    sudo apt install build-essential
    sudo apt install linux-headers-$(uname -r)
    sudo apt install vim git net-tools openssh-server checkinstall

.. important::

    This will install NVIDIA drivers. Check that it is working with ``nvidia-smi``. If you see a table of performance numbers, all is well. If a message comes up that ``nvidia-smi`` cannot communicate with your card, you likely have Safe Boot on. Safe Boot does not allow third-party drivers; NVIDIA is a third party. Disable Safe Boot by typing:

    .. code-block:: console

        sudo mok-utils --disable-validation

    It will ask you for a password. When you reboot, a blue screen will show up. This is the MOK management screen. Press any key to perform MOK management and select :menuselection:`Change Secure Boot state`. It will ask you password questions.

    Alternatively, you can sign modules yourself.


Reconfiguring grub and the display server
=========================================

Edit ``/etc/gdm3/custom.conf``. Uncomment:

.. code-block:: console

    # WaylandEnable=false
    
This forces the computer to use the Xorg display server.

Then edit ``/etc/default/grub``. Replace:

.. code-block:: console

    GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"

with:

.. code-block:: console

    GRUB_CMDLINE_LINUX_DEFAULT="quiet splash nomodeset"

This will disable video drivers until the kernel has started. Update with:

.. code-block:: console

    sudo update-grub

Configure SSH
=============

You should have installed ``openssh-server`` above. Enable the service and start it.

.. code-block:: console

    sudo systemctl enable ssh
    sudo systemctl start ssh

Test your access by ssh-ing in. To set up a firewall, first check that the line below is in ``/etc/default/ufw``.

.. code-block:: console

    IPV6=yes

--------
SSH keys
--------

Set up SSH keys to avoid typing your password all the time.

.. code-block:: console

    ssh-keygen -t rsa -b 4096

Hit enter when it asks you to. Don't create a passphrase, that really defeats the point.

To copy your details to other places, type:

.. code-block:: console

    ssh-copy-id username@destination

You can set up your ``.ssh/config`` file to avoid specifying usernames, IP addresses for hosts, etc. Have a look on the shared drive in ``SHARED/ssh_config`` for details, or just copy that file to ``~/.ssh/config``.

Store
=====

On the computers with SSDs and HDDs, we call our HDDs ``/store``. For now, we have been making a directory called ``/store`` on the computers without SSDs to maintain consistency. It's important that all users can read, write, and execute files in ``/store``.

On a new computer, your HDD will likely not be mounted at ``/store`` automatically. Search (by hitting the windows button) for :menuselection:`Disks`. On the 2.0TB hard disk entry, click the gears button to bring up a menu of actions. Choose :menuselection:`Edit Mount Options...`, turn off :menuselection:`User Session Defaults`, and change the :menuselection:`Mount Point` to ``/store``.

Generally in ``/store`` there is an ``opt/`` directory, where programs are installed, and a ``src/`` directory, where the source code is. However, due to natural human disorganisation, it is likely to be missing one folder or another, or to have ``packages/`` and ``modules/`` directories (see below), or for programs to just be installed willy-nilly. Nonetheless for the sake of the future, it's best to choose one scheme and stick to it (I recommend the module system).

Checkinstall
============

We go now into installing things. Here is where I recommend ``checkinstall``. This replaces ``make install`` in building packages. It builds a portable ``.deb`` package that can be installed on other computers with ``dpkg -i my_package.deb``, and it can be easily removed with ``dpkg -r my_package``. This solves problems with trying to remove packages when you can't remember where they have installed files, e.g. when ``make uninstall`` is not a possibility.

You should have installed ``checkinstall`` earlier, but if not:

.. code-block::

    sudo apt install checkinstall

Using checkinstall
------------------

As noted, ``sudo checkinstall`` replaces ``sudo make install`` in the installation process. It will bring up a menu of options to create the new package. The 'name' of the package is always the folder that it's building from -- for instance, GROMACS always has a default name of ``build``. Type the number next to the option to change it. 

.. warning::

    Do not name your package a default name that can be found when running ``sudo apt install mypackagename``. Otherwise, it is likely to get replaced by whatever is in the apt repository. e.g. GROMACS 2018.3 should always be called ``gromacs-2018.3``, not ``gromacs``.

Environment modules
===================

On shared computers, and possibly your own, it's a good idea to use Environment Modules. These let you switch between package versions with ``module load`` and ``module unload``. It's also a good way to organise and keep track of where things are installed, as modulefiles contain paths.

------------
Installation
------------

Install dependencies:

.. code-block:: console

    sudo apt install tcl tcl8.6-dev

Make folders to store modulefiles and packages. Typically we put these on ``/store``.

.. code-block:: console

    mkdir /store/modules /store/packages

Download and unzip the latest `Environment modules <http://modules.sourceforge.net/>`. Here, it's 4.2.3. Navigate to the directory. Set up the build, make, and checkinstall it.

.. code-block:: console

    cd modules-4.2.3
    ./configure
    make
    sudo checkinstall

If ``/etc/profile.d/modules.sh`` and ``/etc/profile.d/modules.csh`` already exist, skip this step. Otherwise, copy or symlink them.

.. code-block:: console

    sudo ln -s /usr/local/Modules/init/profile.sh /etc/profile.d/modules.sh
    sudo ln -s /usr/local/Modules/init/profile.csh /etc/profile.d/modules.csh

Note that the default path for modulefiles is ``/usr/local/Modules/modulefiles``. Edit ``/usr/local/Modules/init/modulerc`` and add ``module use /store/modules`` to use the directory you made above.

From now on, install packages into ``/store/packages``. 

------------
Modulefiles
------------

Module files are provided in this repo. Download them and copy the directory structure and files into ``/store/modules``. For example, you should have a ``gromacs`` folder and a ``cuda`` folder in ``/store/modules``. Depending on how you choose to install those packages, you may have to edit the paths of each file.


Anaconda
===================

It is a very good idea to download and install Anaconda before doing anyting else with Python. Conda allows you to separate packages into isolated Python environments, and handles package dependencies for you. Create a new environment with:

.. code-block:: console

    conda create --name myenv [python=3.6] [scipy=0.15.0]

.. _install-gcc-label:

GCC
===
------------
Installation
------------

We often need multiple versions of GCC to compile older packages. Versions 6+ can be installed with ``sudo apt``.

.. code-block:: console

    sudo apt install gcc-6 g++-6 gcc-7 g++-7 gcc-8 g++-8

If you wish to install :ref:`GROMACS 2016.1 <install-gromacs-label>`, you will need :ref:`CUDA 8.0 <install-cuda-label>`. This in turn requires GCC 5.4, which needs to be installed manually.

GCC 5.4 is a bit broken. We have a patched package on our shared drive, complete with built ``.deb``. Note that if you wish to install into ``/store/opt``, you can just run ``dpkg -i  gcc-5.4.0_5.4.0-patched-1_amd64.deb``. Alternatively, copy it and run:

.. code-block:: console

    cd gcc-5.4.0
    ./contrib/download_prerequisites
    cd ..
    mkdir objdir && cd objdir
    $PWD/../gcc-5.4.0/configure --prefix=/store/opt/gcc-5.4.0 --enable-languages=c,c++,fortran --disable-multilib
    make
    sudo checkinstall

--------------------------
Switching between versions
--------------------------

At this point you may wish to set up ``update-alternatives`` for easy switching between compilers. 

Clear any options you may already have for gcc and g++.

.. code-block:: console

    sudo update-alternatives --remove-all gcc
    sudo update-alternatives --remove-all g++

Then configure the program path (second last field, e.g. ``/store/opt/gcc-5.4.0/bin/gcc``) to the first field (e.g. ``/usr/bin/gcc``). The last number indicates priority. A larger number is higher.

.. code-block:: console

    sudo update-alternatives --install /usr/bin/gcc gcc /store/opt/gcc-5.4.0/bin/gcc 10
    sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-6 20
    sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-7 30
    sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-8 40
    sudo update-alternatives --install /usr/bin/g++ g++ /store/opt/gcc-5.4.0/bin/g++ 10
    sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-6 20
    sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-7 30
    sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-8 40

To switch between them, run:

.. code-block::

    sudo update-alternatives --config gcc

.. _install-cuda-label:


CUDA
====

------
CUDA 8
------

This is necessary for GROMACS 2016.1.

Make sure you have the patched version of ``cuda_8.0.61_375.26_linux.run``. It's too big to put on the repo, but it is on our shared drive in ``SHARED/cuda/``. External users can download the unpatched version and patch it themselves.

Unpack and copy ``InstallUtils.pm`` to your computer. Then install the toolkit.

.. code-block:: console

    /cuda_8.0.61_375.25_linux.run --tar mxvf
    sudo cp InstallUtils.pm /usr/lib/x86_64-linux-gnu/perl-base
    export $PERL5LIB
    sudo sh cuda_8.0.61_375.25_linux.run --toolkit --toolkitpath=/store/packages/cuda/cuda-8.0 --silent

Check that ``/store/packages/cuda/cuda-8.0/bin/nvcc`` exists.

-------
CUDA 10
-------

This shouldn't need the extra copying and exporting, I think.

.. code-block:: console

    sudo sh cuda_10.1.105_418.39_linux.run --toolkit --toolkitpath=/store/packages/cuda/cuda-10.1 --silent

.. _install-gromacs-label:

GROMACS
=======

You need ``cmake`` for this.

.. code-block:: console

    sudo apt install cmake

------
2016.1
------

Make sure you have GCC 5.4 running with ``gcc --version``. If it's another version, change it with ``sudo update-alternatives --config gcc``.

.. code-block:: console

    tar xfz gromacs-2016.1.tar.gz
    cd gromacs-2016.1
    mkdir build && cd build
    cmake .. -DGMX_BUILD_OWN_FFTW=ON -DREGRESSIONTEST_DOWNLOAD=ON \
    -DGMX_GPU=on -DCMAKE_INSTALL_PREFIX=/store/packages/gromacs/gromacs-2016.1 \
    -DCMAKE_CXX_FLAGS=-fPIC -DCUDA_TOOLKIT_ROOT_DIR=/store/packages/cuda/cuda-8.0
    make -j 6
    make check
    sudo checkinstall

------
2018.3
------

I think this works fine with CUDA 10 and GCC 8.3.

.. code-block:: console

    tar xfz gromacs-2018.3.tar.gz
    cd gromacs-2018.3
    mkdir build && cd build
    cmake .. -DGMX_BUILD_OWN_FFTW=ON -DREGRESSIONTEST_DOWNLOAD=ON \
    -DGMX_GPU=on -DCMAKE_INSTALL_PREFIX=/store/packages/gromacs/gromacs-2018.3 \
    -DCMAKE_CXX_FLAGS=-fPIC -DCUDA_TOOLKIT_ROOT_DIR=/store/packages/cuda/cuda-10.1
    make -j 6
    make check
    sudo checkinstall

--------------
Other versions
--------------

Other versions will have largely the same commands. Remember to check which CUDA and GCC versions they should have.


-------------------
g_mydensity (4.5.7)
-------------------

`g_mydensity <http://perso.ibcp.fr/luca.monticelli/tools/index.html>`_ is a great tool with an extremely specific required environment. While it seems to have previously compiled with GROMACS 5.1.4, that may have been a fever dream because it definitely didn't in 2019. The program is tested against GROMACS 4.5.x; we compiled 4.5.7 for this. GROMACS 4.5.7 requires ``fftw-3``.

Start by setting your compiler flags with ``-fPIC``.

.. code-block:: console

    export CXXFLAGS=-fPIC
    export CFLAGS=-fPIC

Install fftw-3
--------------

We downloaded `FFTE 3.3.8 <http://www.fftw.org/download.html>`_. 

.. code-block:: console

    tar xvf fftw-3.3.8.tar
    cd fftw-3.3.8
    ./configure --enable-threads --enable-float
    make -j 12
    sudo checkinstall

If something goes wrong here, enable fPIC, run ``make clean``, and try again.

GROMACS 4.5.7
-------------

.. code-block:: console

    cd gromacs-4.5.7/
    mkdir build && cd build
    cmake .. -DCMAKE_INSTALL_PREFIX=/store/packages/gromacs/gromacs-4.5.7
    make -j 12
    make test
    sudo checkinstall

g_mydensity
-----------

We downloaded ours from `Luca Monticelli's website <http://perso.ibcp.fr/luca.monticelli/tools/index.html>`_. You must source GROMACS 4.5.7 for this to compile. We have chosen to symlink it to ``/usr/local/bin``.

.. code-block::

    module load gromacs/4.5.7
    mv jbarnoud-g_mydensity-4f9a93c151a9 g_mydensity && cd g_mydensity
    make
    sudo chmod a+x g_mydensity
    sudo ln -s /store/opt/g_mydensity/g_mydensity /usr/local/bin/g_mydensity


.. _install-amber-label:

