.. -*- coding: utf-8 -*-

==================
Installing FATSLiM
==================

You'll need a conda and pip

Make a new conda environment 
============================

make a new conda environment::

    conda create -name fatslim python=3.5
  


Building  
========

add numpy and the right cython 

.. code-block:: bash

    pip install cython==0.16
    conda install numpy
    pip install fatslim  

