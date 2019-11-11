.. -*- coding: utf-8 -*-
.. _environment-label:

============================
Customising your environment
============================

Customise your environment using your ``.bashrc`` and ``.bash_aliases`` files. I also get most users to source ``/home/shared/.bashrc`` and ``/home/shared/.bash_aliases`` for helpful scripts. These necessitate the use of some other helpful files.

User profile
============

First off is the ``.usr_profile`` file. I use the information in this to set up useful shortcuts for the shared drive, supercomputer, etc. ::

# export SH_DRIVE_FOLDER= # your folder on the shared drive, e.g. LILY
# export RJ_PROJECT= # project folder on Raijin
# export RJ_USER= # username on Raijin
# export RJ_HOME_DIR= # home directory on Raijin

Uncomment lines as they apply.

Shared drive credentials
========================

One of the useful commands in ``.bash_aliases`` is::

    function omara {
        GROUP=`id -gn`
            if [ ! -d "$HOME/mnt/omara/$SH_DRIVE_FOLDER" ]; then
                    echo "Mounting omara shared drive."
                    sudo mount -w -t cifs //ANUFILE01/Omara $HOME/mnt/omara -o rw,credentials=$HOME/.credentials,gid=$GROUP,file_mode=0777,dir_mode=0777,mfsymlinks
            fi
            cd ~/mnt/omara/$SH_DRIVE_FOLDER
    }

This loads the shared drive, if it is not loaded, and goes to your directory. It requires the following::

    sudo apt install cifs-utils

You also need to make a ``mnt`` directory in home. ::

    mkdir -p mnt/omara

However, it requires your (ANU email) login credentials. You can create a ``~/.credentials`` file and change permissions with ``chmod 000 .credentials`` so that other users can't read it. *However*, root will always be able to override this. If this makes you feel uncomfortable, you can use the GUI to mount the drive instead.

.. glossary::

    username
        Your u- number
    
    password
        Your email password

    domain
        UDS

