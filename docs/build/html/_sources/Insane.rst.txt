.. -*- coding: utf-8 -*-

=================
Installing Insane 
=================

You'll need conda and pip

Make a new conda environment 
============================

make a new conda environment

.. code-block:: bash

    conda create --name insane python=2.7
    conda activate insane
  


Building  
========

add numpy and pip 

.. code-block:: bash

    pip install numpy
    pip install insane

Then make sure that the path it spits out is in your path.
Often this is ~/.local/bin/insane

add the following line to your bashrc

.. code-block:: bash

   PATH=$PATH:~/.local/bin/


