.. -*- coding: utf-8 -*-
.. _remote-work-label:

================
Working remotely
================

This document is to help you use your remote (i.e. Research School of Chemistry) workstation from your local laptop. It's targeted at people with Mac/Unix-based devices because I have no idea how to use Windows. If you know how to use Windows, please let me know so this can be updated.

**Iodine** is our local GPU cluster. We use it as a public SSH gateway to our local RSC network. You must already have an account set up on Iodine to use it (contact RSC IT, Lily, or Hugo for this). 



Connecting via SSH
==================

Secure Shell (SSH) is a protocol for connecting between a client and a server. In this case your remote workstation is running a SSH server and your laptop, the client, will be connecting to it -- but through the intermediary server, Iodine.


---------------
Setting up keys
---------------

Your remote and local computers should already have SSH set up. You can probably skip a lot of this for the remote workstations and Iodine, as you've probably already set up your keys etc. when we set up your computer. However, check that your local machine is also set up.

If not, you will need someone with admin permissions on Ubuntu:

.. code-block:: bash

    sudo apt install openssh-server
    sudo systemctl enable ssh
    sudo systemctl start ssh


SSH is pre-installed on Macs. You may need to turn it on:

.. code-block:: bash

    sudo systemsetup -setremotelogin on


Then, you need set up SSH keys on both your local laptop and Iodine, as we will be connecting through Iodine.

.. code-block:: bash

    ssh-keygen -t rsa -b 4096

Copy your Iodine key to your RSC computer while logged into Iodine:

.. code-block:: bash

    ssh-copy-id username@destination

And the same from your laptop to Iodine.

---------------------------
Setting up your config file
---------------------------

We want to make connecting to your RSC computer as painless as possible, so let's set up our SSH config file. On both Mac and Ubuntu this will be located at ``~/.ssh/config``. You may have to create it if it does not exist.

Add the following entry for Iodine::

    Host iodine
        Hostname iodine.anu.edu.au
        User my_iodine_username

And another for your computer. Mine is called gavle. Your hostname will be your IP address, which you can get with ``curl ifconfig.me``. ::

    Host gavle
        Hostname xxx.xxx.xx.xxx
        ProxyCommand ssh my_iodine_username@iodine nc %h %p
        User lily
        ForwardAgent yes
        #ForwardX11 yes

That last option controls whether you want to forward displays, e.g. VMD. You can turn it on by default by uncommenting that line, or you can manually connect with ``ssh -X``. The ProxyCommand pipes your connection through Iodine to your computer.

Now copy your key to your computer, directly from your laptop.

.. code-block:: bash

    ssh-copy-id lily@gavle


You should now be able to SSH directly into your computer without manually logging into Iodine.

Environments for remote work
============================

------------------
Terminal emulators
------------------

From unhappy experience, it is *extremely easy* to get confused about where you are when you have multiple Terminal windows open, on different remote computers. For this reason you are **strongly** encouraged to use a terminal emulator with automatic profile switching.  I configure my terminal emulators to change themes when I am on Iodine, gavle, Gadi, and the shared drive.

On Mac I use iTerm2_ and on Ubuntu I use Tilix_ . On both you will need to download scripts for shell integration (`for iterm2 <https://www.iterm2.com/documentation-shell-integration.html>`_ , `for Tilix <https://github.com/gnunn1/tilix/wiki/Automatic-(Triggered)-Profile-Switching>`_ )

.. _iTerm2: https://www.iterm2.com/
.. _Tilix: https://gnunn1.github.io/tilix-web/

----------------------------
Continuous terminal sessions
----------------------------

Working remotely is all well and good but having your process die every time your internet connection drops makes it unfeasible to do anything that will take a significant amount of time. 

tmux_ is a terminal multiplexer, allowing you to switch between different sessions within a terminal window, keep processes running in the background, and reattach to sessions to check on them. It's very powerful and has `lots of advanced features <https://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/>`_ but I really only use it for running processes that I want to take a while and come back to. A session is like a different terminal window -- you can switch conda environments, etc.

It's installed on most of the RSC computers, but if not, you will need admin permissions:

.. code-block:: bash

    sudo apt install tmux


To start a new session::

    tmux new -s session-name

You don't *have* to name it, you can just type ``tmux new``, but this typically means you then reattach to all of them trying to work out which session had the thing you were working on.

To detach (while in the session) ::

    tmux detach

To re-attach::

    tmux attach -t session-name

(``tmux attach`` re-attaches to your last open session).

To see which sessions you have running::

    tmux ls

To kill a session::

    tmux kill-session -t session-name


I use it for stuff that takes forever (e.g. ``gmx cluster``) or I want to go on indefinitely (e.g. starting a Jupyter server). Note that you may have to background your process to be able to detach properly (e.g. ``jupyter notebook &``). 


.. _tmux: https://github.com/tmux/tmux/wiki


Remote analysis
===============

Ok, now we've set it up so we can run command-line stuff reasonably comfortably. What about all the other stuff, like looking at trajectories through VMD, or plotting, or making more than one-line changes to files? My personal laptop can just about handle opening VS Code and Firefox before it starts melting down. It absolutely cannot store the TB of data that molecular dynamics tends to build up to.

-----------------
Visual forwarding
-----------------

If you're SSH-ing with X11 forwarding (``ssh -X``, you'll be able to pipe stuff like VMD and xmgrace to your laptop. This requires a decent internet connection to not be miserable, but the option is there. Every RSC computer should be set up to allow X11 forwarding, but just in case, edit ``/etc/ssh/sshd_config`` (requires admin permissions) and set/uncomment::

    X11Forwarding yes

The first time you SSH in you may get a message like::

    /usr/bin/xauth:  file /home/lily/.Xauthority does not exist

Try typing ``xauth``, exiting, and seeing if it works now. 


------------
Text editing
------------

**vim**

Editing files with ``vim`` is a lot more pleasant if you configure it a bit. For example, my ``~/.vimrc`` is currently

.. code-block:: vim

    syntax on

    set softtabstop=4               " virtual tab stop
    set tabstop=8                   " display files with tabs...
    set shiftwidth=4                " spaces per indent
    set smartindent                 " smart indent
    set autoindent                  " same indent as line above
    set expandtab                   " tab to space
    set preserveindent              " preserve indents


    set colorcolumn=80              " line at column 80 (pep8)
    set mouse=a                     " use mouse
    set number                      " line numbers
    set wrap                        " wrap long lines

Comment the ``set mouse=a`` if you don't like using your mouse to move the cursor. It means that if you highlight, you will enter visual mode and copy is replaced by ``y`` (yank) and paste is ``p`` (paste). Personally, I just use ``less`` if I don't want mouse mode to activate. 

**Visual Studio**

The alternative is using a non-commandline text editor. I use Visual Studio with the `Remote SSH <https://code.visualstudio.com/docs/remote/ssh>`_ extension. I've really liked it so far -- if your internet connection drops, it will ask you to reload the window, but it won't lose unsaved work unless you close the VS Code instance on your local laptop. 

-----------------
Jupyter notebooks
-----------------

Jupyter notebooks run on a server, which means that you can access notebook servers from your laptop. First, set up your Jupyter configuration and your password on your RSC computer. Your configuration file should be at ``~/.jupyter/jupyter_notebook_config.py``. If it does not exist, make a new one::

    jupyter notebook --generate-config

And if you have not already, set up a password::

    jupyter notebook password

You can re-run that command to reset a lost password or to change it.

Then start a server in a ``tmux`` session to ensure that it persists beyond closing the terminal window or drops in internet::

    tmux new -s jupyter
    jupyter notebook --port=8889

Now set-up an SSH tunnel to that port. To connect the 8889 port on your laptop to the 8889 port on your RSC computer, type::

    ssh -N -f -L 127.0.0.1:8889:127.0.0.1:8889 lily@gavle

Now if you open a web browser and go to ``localhost:8889/``, it will ask you for a password. Type in the Jupyter password you have set, and *you're in*. Any notebook that you create and edit is actually being created and edited on your remote computer. (Note: if you have JupyterLab installed, that's another alternative to remote text editing to ``vim`` and VS Code described above).

If your internet drops out, you might need to re-establish the tunnel. I have a command in my ``~/.bash_aliases`` to make this easier:

.. code-block:: bash

    function tunnel {
        port=$1
        ssh -N -f -L 127.0.0.1:${port}:127.0.0.1:${port} lily@gavle
    }

which I call by typing ``tunnel 8889``. Occasionally I also want to free up that port, so this is also in my ``~/.bash_aliases``:

.. code-block:: bash

    function killport {
        port=$1
        lsof -ti:${port} | xargs kill -9
    }

and is called by typing ``killport 8889``. This kills *all* processes associated with that port, not just tunnels. Bash aliases are your friend in remote work.


Data and backups
================

Working remotely is great when your remote computer is accessible. Our experience has been that it is unwise to rely blindly on university internet or server infrastructure. 

Follow the 3-2-1 rule for backups:

* **three** copies of your data
* **two** of the copies on different media
* **one** of them should be off-site.

**Use the shared drive to store your data**. The shared server is in a different location and is maintained by central IT. If you have not already, install the ANU VPN so you can access the shared drive off-campus. However, don't work off the shared drive (e.g. running analysis on it, reading/writing too many files, etc) because it's really slow. Instead:

**Copy any data you're currently working on to your RSC computer**. Don't save it in your home directory (``~/Documents``), because our computers don't come with that much space. We have an integrated hard drive with 1-2 TB storage, usually mounted at ``/store``. Make a folder for yourself and work off that. However, we will lose access to both the shared drive and our RSC computers if the network goes down or there's a power outage. So, 

**Copy your data to a portable hard drive and keep that with you.** Once you're into the analysis stage for that project, your data won't change much so you won't have to update this regularly.

---------------
Version control
---------------

The best way to back up your data analysis is version control with ``git``. If you're a student, you can get a Pro GitHub account (and other stuff) for free with the `Github Student Developer Pack`_ . (Once you're on GitHub, join the `OMaraLab`_ organisation!) With a Pro account, you can have private repositories. I use these to store my analysis scripts and notebooks (not images because GitHub has a file size limit). 

Alternatively, you can use OneDrive / Dropbox / other cloud storage services. All ANU staff and students have an Office365 subscription. On Ubuntu you can download `this free OneDrive client <https://github.com/skilion/onedrive>`_ that works quite well for me. 




.. _`Github Student Developer Pack`: https://education.github.com/pack
.. _`OMaraLab`: https://github.com/OMaraLab