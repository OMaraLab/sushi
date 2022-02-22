.. -*- coding: utf-8 -*-

===============
Archiving Files 
===============

Its very important to save your raw unprocessed trajectories for reproducibility
and legal reasons. The primary way of doing this is by archiving data on a long term storage drive.

Our long term data policy is still under construction, however it is generally a good idea to follow the rule of 3

* 3 copies of your data
* 2 different locations
* 1 copy offsite 

Most long term storage drives require data to be tarballed.

MDSS
====

A great offsite long term storage option is the MDSS tape drive at NCI.
You can use the netcp command to move stuff to MDSS easily:

.. code-block:: bash

    netcp -C -l other=mdss,mem=160GB,walltime=10:00:00 -z -t tarball.tar target destination

where destination is a path on MDSS.


This handy bash macro accomplishes this easily, you can add it to yout bashrc.

.. code-block:: bash
    
    mdssput () {
        netcp -C -l other=mdss,mem=160GB,walltime=10:00:00 -z -t ${1}.tar $1 $2
    }

It is **VERY VERY** important that you check that the job exits without errors.
Check the .e and .o files whcih should be in the directory you launched the job from.

You can also manually transfer your tarballed files directly to mdss.

.. code-block:: bash

    mdss put tarball.tar destination


Verifying archive integrity
===========================

Everytime you transfer data remotely, there is a small but non-negligible chance of data loss.
To minimise this use rsync rather than scp as it has error checking.

Regardless, it is important to verify the intergity of your archive for long term storage.

I reccomend several checks to verify that your archive is working as intended.

The first one is to compute the md5sum of your tarball before and after it has been transferred.

.. code-block:: bash

    md5sum tarball.tar 

If the two do not match, you have lost some data somewhere.


You should also untar your newly transferred tarball on /scratch to verify that everything is working as intended.
You should also download a trajectory or three and see that they load ok in vmd or with `gmx check`.



