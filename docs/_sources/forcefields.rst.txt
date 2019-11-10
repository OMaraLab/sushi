.. -*- coding: utf-8 -*-
.. include:: <isonum.txt>
.. include:: <isotech.txt>

===================
Lab 5: Force fields
===================

Aim
====

The aim of this lab is to:

#. Develop a force field for a molecule of water
#. Look at dihedrals and assign atom types for alanine.

First set up your computer environment by following the instructions in the Setup section. Then work through your Mathematica notebook, which you can download from Wattle. The information on this page is a supplement to the notebook. 

.. important:: 

   **Submission:**
   To submit this lab, please upload both your Mathematica notebook and a PDF including the questions in the Questions section. 

Setup and scripts
=================

First download the Lab 5 folder on Wattle. 

Then open Terminal. You can find this by opening Spotlight (command + space) and typing "terminal".

Type the command below to go to the folder you have just downloaded.

.. code-block:: bash

   cd ~/Downloads/Lab\ 5*

Then type the command below to install the Python library you need and the custom scripts. You may have to wait a while.

.. code-block:: bash

   bash setup.sh

When it is finished, close that Terminal window and open a new one. You will need a terminal to use the custom scripts. To use them, just type :code:`my_script_name my_file`. For example:

.. code-block:: bash

   out_to_xyz my_file.out

VMD
---

Use VMD the same way as the scripts. The command is :code:`vmd`.

.. code-block:: bash

   vmd my_file.xyz



Functional forms of classical force field terms
================================================


Taylor series
-------------

We can use a Taylor series to approximate a function :math:`f(x)` at point :math:`a`.

.. math::
   f(x) = f(a) + \frac{f'(a)}{1!}(x-a) + \frac{f''(a)}{2!}(x-a)^2 + \frac{f'''(a)}{3!}(x-a)^3 + ...

We can approximate the energy function :math:`E_{bond}` for stretching a bond between two atoms as a Taylor series around an equilibrium bond length :math:`r_{eq}`. We can substitute this equilibrium bond length :math:`r_{eq}` for :math:`a` above, and the current bond length :math:`r` for :math:`x` to give:

.. math::
   E_{bond}(r) = E(r_{eq}) + \frac{dE}{dR}(r_{eq}) + \frac{d^2E}{2}(r-r_{eq})^2 + \frac{d^3E}{6}(r-r_{eq})^3 + ...

:math:`E(r_{eq})` is the energy associated with the equilibrium bond length. This is normally set to 0, as it is just the zero point of the energy scale. 

As the Taylor expansion is around the equilibrium value, the gradient of the energy is 0. Therefore, :math:`\frac{dE}{dR}(r_{eq})` is set to zero.

In molecular dynamics, a Taylor series expansion for bonds, angles, and improper angles normally truncates it at the second order. Without :math:`E(r_{eq})` and :math:`\frac{dE}{dR}(r_{eq})`, this gives:

.. math::
   E_{bond}(r) = \frac{d^2E}{2}(r-r_{eq})^2

Force fields usually represent the :math:`d^2E` term as force constant :math:`k_{bond}`, resulting in the quadratic equation:

.. math::
   E_{bond}(r) = \frac{1}{2}k_{bond}(r-r_{eq})^2

This is the same functional form as the potential energy stored in a spring, which why molecular dynamics is often described as systems of balls on springs.

Morse potential
---------------

The quadratic function above is the simplest possible, and is sometimes not enough to reproduce experimental observations. Another functional form that can be used to aproximate the energy function for stretching a bond is the Morse potential.

.. math::
   E_{bond} (r) &= D(1 - e^{-a (r-r_{eq})})^2

   a &= \sqrt{\frac{k}{2D}}

Here, :math:`D` is the dissociation energy and :math:`a` is related to the force constant. The Morse potential reproduces bond behaviour quite accurately; however, it is computationally expensive to calculate. At ambient temperatures, e.g. 300 K, bond geometry does not vary much from its equilibrium value and a Taylor expression is sufficient.

IQMol instructions
====================

You may find it easier to download and edit files instead of constructing them yourselves.

Potential energy scans
----------------------

In this lab, we will use IQMol to submit the job. However, you will have create the file yourself, as IQMol has not actually implemented a way to create PES jobs via the Job Setup.

A normal potential energy scan optimises the geometry of the molecule for each point in the surface. This can take quite a while. Instead, we will perform a "frozen scan", which just computes the energy for each point.

Z-matrices
~~~~~~~~~~

Normally, your input coordinates are in a Cartesian format, with x, y, and z positions.

.. code-block::

   $molecule
   0 1
   O   - 0.0147820    0.0293097   - 0.0232867
   H    0.1429191   - 0.8319170   - 0.4820959
   H    0.0916809   - 0.4117318    0.8516712
   $end



A frozen potential energy scan requires the input geometry to be in a Z-matrix format. A Z-matrix is a way to represent a molecule using "internal coordinates", or by defining each atom by its relationship to another atom. Water is written in a Z-matrix format below:

.. code-block::

   $molecule
   0 1
    O
    H  1  r2
    H  1  r3  2  a3

    r2= 0.1234
    r3= 0.1234
    a3= 0.1234
   $end


In this Z-matrix, the oxygen atom is chosen as the fixed point that the other atoms orient around. It is given the number :code:`1`. The line :code:`H  1  r2` represents a H atom, with the distance :code:`r2` from atom 1 (i.e. oxygen). This H atom is given the number "2". 

The third line, :code:`H  1  r3  2  a3`, represents another H atom, with the number #. :code:`r3` is the distance from H to atom 1 (oxygen). :code:`a3` is the angle H-1-2, i.e. H3-O1-H2.

The values of the bond and angles are defined under the Z-matrix specification. Here they are placeholder values, but replace them with your equilibrium bond and angle values. You can copy these coordinates into your QChem Job builder file section for convenience. If you choose to make your own Z-matrix, please make sure the oxygen is first.

Program section
~~~~~~~~~~~~~~~~

.. code-block::

   $rem
      BASIS  =  6-31G*
      FROZEN_SCAN  =  TRUE
      GUI  =  2
      JOB_TYPE  =  PES_Scan
      METHOD  =  MP2
   $end


Choosing a PES Scan job to calculate, and select your Method and Basis sets appropriately. However, you will have to add :code:`FROZEN_SCAN  =  TRUE` in the job section manually. It can appear anywhere in this block, so long as it is in the section that starts with :code:`$rem`.

Scan section
~~~~~~~~~~~~~

.. code-block::

   $scan
   $end


This empty block appears when you choose a PES Scan job. This is where you specify which variables you wish to scan. 

.. code-block::

   $scan
   stre  atom1  atom2  value1 value2 incr
   ...
   bend  atom1  atom2  atom3  value1 value2 incr
   ...
   tors  atom1  atom2  atom3  atom4  value1 value2 incr
   ...
   $end


The available keywords are :code:`stre` for a bond, :code:`bend` for an angle, and :code:`tors` for a dihedral angle. **Only scan across one variable per job, i.e. your input file should only have one line in this section.** Ask a demonstrator if you're uncertain.

Specify the atoms by number. :code:`value1` and :code:`value2` are your starting and ending values, respectively. :code:`incr` stands for "increment", or the step size. For example:

.. code-block::

   $scan
   stre  1  2  0.1 0.9 0.05
   $end


This section would specify that QChem should calculate the energy of the water molecule for 17 structures: when the bond between O1-H2 is 0.1, 0.15, 0.2, 0.15, ..., 0.85, 0.9. 

.. important:: 

   When you define your angle, make sure the central point of the angle is in the middle. e.g. for your H-O-H angle, your :code:`$scan` line should start off with :code:`bend  2  1  3`. 

Here is :download:`an example bond file <files/forcefields/h2o_bond.inp>` and :download:`an example angle file <files/forcefields/h2o_angle.inp>`.

*Ab initio* molecular dynamics
------------------------------

This course largely focuses on classical molecular dynamics, where empirical potentials are used to calculate the potential energy of the system. When a more accurate representation is required, quantum chemistry methods can be used to calculate the energy instead. This is called *ab initio* molecular dynamics (AIMD).

To set up an AIMD job, select :code:`Ab Initio MD` from the **Calculate** dropdown. We will set the :code:`Initial Velocities` to :code:`Thermal`, with a temperature of 300K.

.. important:: 

   Make sure your file includes this line in the :code:`$rem` section: :code:`AIMD_TEMP  =  300`. You may have to add it manually.


The time step is defined in atomic units. We want the simulation to have a time step of 0.5 femtoseconds (1 fs is :math:`10^{-15}` seconds), which is approximately equal to 21 atomic units.

Set the number of steps to 500 and submit the job.

Here is :download:`an example file <files/forcefields/h2o_aimd.inp>`.


Parametrising water
====================

In this part of the lab, you will explore the functional forms of force field terms, and build a force field for a water molecule.

#. Use MP2/6-31G* to optimise the geometry of a water molecule to obtain equilibrium bond and angle values.
#. Scan the potential energy surface of water over a range of bond lengths, and angle values, at MP2/6-31G*.

   * Bond lengths: from 0.5 |angst| below your equilibrium bond value to 1.5 |angst| above your equilibrium bond value, in increments of 0.05 |angst|. Note that you only need to scan one O-H bond.
   * Angle values: from 30 |deg| below your equilibrium angle value, to 30 |deg| above, in increments of 2\ |nbsp|\ |deg|. 
#. Plot these points to see the shape of the potential energy for these force field terms.
#. Define functions for these common force field functional forms:

   * a Taylor expansion to the second order
   * a Taylor expansion to the third order
   * a Morse potential

#. Use :code:`NonlinearModelFit` to fit your functions to your bond data, and plot each function. Then plot them all together, over your data points. **(3 marks)**
#. Use MP2/6-31G* to simulate water at 300 K with an *Ab initio* molecular dynamics job, for 250 femtoseconds. Initiate with thermal velocities.
#. Use the :code:`out_to_xyz` script to convert the output file to a xyz format. View your trajectory with VMD.
#. Use the :code:`water_geometry` script to extract the bond lengths and angle values for each frame of the trajectory. Plot the values of the bonds and angles.
#. Calculate the mean and standard deviation for the O-H bond length and the H-O-H angle value over the AIMD trajectory.
#. Select the data points of your bond and angle potential energy scans within 3 standard deviations of the mean of the AIMD trajectory.
#. Re-fit the functions from #4 to this subset of data. Compare the standard errors and the p-values with your earlier fits to the complete dataset.
#. Plot the new fitted functions together, over the subset of the data points from the potential energy scan. **(3 marks)**
#. Fit a second order Taylor expansion the subset of your angle PES data.
#. Combine your second-order Taylor expansions for your bonds and angle to create a potential energy function for water. Plot the potential energy surface (z-axis) created by the second order Taylor expansions over the change in O-H bond length (x-axis) and change in H-O-H angle value (y-axis).
#. Calculate the potential energy of each frame of the AIMD trajectory with your function. Compare the shape of the plot to the MP2/6-31G* energy plot.
#. Calculate the average difference between your calculated energy and the MP2 energy. What does this difference represent?
#. Assign atom types, equilibrium bond values, bond force constants, equilibrium angle values, and equilibrium force constants to water. You have now created your own force field for a water molecule. **(3 marks)**

Parametrising alanine
=====================

#. Use :code:`Manipulate` for an interactive plot of the Fourier series function for a dihedral term. What does the :code:`k` parameter correspond to?
#. Use MP2/6-31G* to optimise the geometry of an alanine molecule. Calculate the phase shift of the O=C-CA-N dihedral.
#. Rotate the O=C-CA-N dihedral and calculate a single-point energy in order to determine the force constant of that dihedral term. How much did you rotate it by, and why? 
#. Assign atom types to alanine. Calculate the number of non-bonded equations required for 3 alanine molecules. 
#. Assign united-atom atom types to alanine. Calculate the number of non-bonded equations required for 3 alanine molecules.
#. Calculate the number of non-bonded equations required for 3 coarse-grained alanine molecules.



Questions
=========

This is the total list of questions you should answer, which also includes the questions that were asked within the workbook. Please answer these in a text document and submit it as a PDF. Include your two graphs where you compare the fits of functional forms to the bond data. (#5 and #12) **(6 marks)**

#. Which functional form had the best fit to your overall bond data in the Parametrising Water section? Did this change once you fitted the function to a subset of the data? Which functional form do most force fields use, and why? **(4 marks)**
#. What does the average difference between your calculated energy for water and the MP2 energy for water represent? **(1 mark)**
#. What does the :code:`k` parameter define in a dihedral function? What is the phase shift and force constant of the O=CA-C-N dihedral in an alanine molecule? How did you find the force constant of the O=CA-C-N dihedral? **(4 marks)**
#. How many non-bonded calculations do you have to complete for 3 molecules of alanine, when they are: all-atom, united-atom, or coarse-grained? What are the atom types for the all-atom and united-atom representations? **(7 marks)**
#. There are two approaches to force field parametrisation. Which was followed in this lab? What is the other approach, and what is one reason we did not use that one? **(3 marks)**
#. In this lab you used *ab initio* molecular dynamics (AIMD) simulations to help you choose parameters. Why is it infeasible to use *ab initio* methods for protein simulations? Make reference to the scaling of the computations required. What is one situation when you would need AIMD rather than classical MD, apart from when you need more accurate representations? **(3 marks)**
#. List the 4 bonded and 2 non-bonded interactions that contribute to the potential energy function of a classical molecular dynamics simulation. Give the functional form that is most commonly used for each. **(6 marks)**
#. Explain the concept of additivity as it applies to classical force fields. **(2 marks)**
#. Explain the concept of transferability as it applies to classical force fields. **(2 marks)**

**(Total 38 marks)**