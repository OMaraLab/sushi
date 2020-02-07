.. -*- coding: utf-8 -*-

======================
Making MemSurfer Work
======================

You'll need a GitHub account and sudo

Environment Prereqs 
===================

get swig cython MPFR GMP and CGAL from the repos::

    sudo apt install swig 
    conda install cython 
    sudo apt install libgmp-dev libmpfr-dev libcgal-dev

clone the repo off the memsurfer github ::

    git clone --recursive git@github.com:LLNL/MemSurfer.git
    MEM_HOME=`pwd`/MemSurfer
    

Building 
========
run their install script following their instructions

.. code-block:: bash

    cd MemSurfer 
    export CC=`which gcc`
    export CXX=`which g++`
    chmod +x install_deps.sh
    ./install_deps.sh

everything should work now 

.. code-block:: bash

   cd $MEM_HOME
   CC=`which gcc` CXX=`which g++` LDCXXSHARED="`which g++` -bundle -undefined dynamic_lookup" 
   python setup.py install
