# sushi
Computer set-ups, shell environments, installation notes, etc.

## Setting up a new computer

### Essentials

```console
lily@gavle$ sudo apt-get update
lily@gavle$ sudo apt-get install build-essential git vim gcc-multilib g++-multilib checkinstall sublime openssh-server tmux
```

**Usually a good idea**:
```console
lily@gavle$ ubuntu-drivers autoinstall
```

### NVIDIA driver

Black screen? Update your NVIDIA driver and blacklist the Nouveau one.

1. Install the latest NVIDIA driver (418 here, but check for the latest.)

```console
lily@gavle$ sudo apt-get install nvidia-driver-418
```

2. If something like `/etc/modprobe.d/nvidia-installer-disable-nouveau.conf` now exists, skip this step. Else, write this to file `/etc/modprobe.d/blacklist-nouveau.conf`:

```console
blacklist nouveau
options nouveau modeset=0
```

3. Update initramfs and reboot.

```console
lily@gavle$ sudo update-initramfs -u
lily@gavle$ sudo reboot
```

### Environment modules
1. Install dependencies. Here th elatest tcl-dev is 8.6, but check.
```console
lily@gavle$ sudo apt-get install tcl tcl8.6-dev
```

2. Download and unzip the latest [Environment Modules](http://modules.sourceforge.net/). Here it's 4.2.3. Navigate to the directory.
```console
lily@gavle$ cd modules-4.2.3/
```

3. Make folders to store the modulefiles and packages
```console
lily@gavle$ sudo mkdir /modules /packages
```

4. Set up the build, make, and checkinstall it.
```console
lily@gavle$ ./configure --with-module-path=/modules/
lily@gavle$ make
lily@gavle$ sudo checkinstall
```
5. If `/etc/profile.d/modules.sh` and `/etc/profile.d/modules.csh` already exist, skip this step. Otherwise, copy or symlink them.
```console
lily@gavle$ sudo ln -s /usr/local/Modules/init/profile.sh /etc/profile.d/modules.sh
lily@gavle$ sudo ln -s /usr/local/Modules/init/profile.csh /etc/profile.d/modules.csh
```
Now, when you install applications, try to install them into /packages. Modulefiles are included in the repo for your convenience.

### Anaconda and Python
Download and install Anaconda **before** you do anything with Python. This will likely install Visual Studio Code as well, which is super helpful. Open a folder in VS Code with

```console
lily@gavle$ code .
```

#### Environments
Create a new environment with 
```console
lily@gavle$ conda create --name myenv [python=3.6] [scipy=0.15.0]
```
Use the options in brackets if you need a specific version of Python and/or specific packages. You can also activate it and install directly into it.

**To do**
I'll add an environment.yml file for my computational chemistry environment here soon.

```console
lily@gavle$ conda activate myenv
```

If you have a conda environment activated, running `pip install` will install that package into your current environment. This is very useful! Isolate your virtual environments and never let the twain meet. I also don't modify the base environment that is installed by default.

#### Jupyter
Often you want to be developing in a specific environment in a Jupyter notebook. Unfortunately, kernels are not automatically created with your environment. Create a kernel by installing `ipykernel` and running the command below:

```console
lily@gavle$ conda activate myenv
lily@gavle$ conda install ipykernel
lily@gavle$ python -m ipykernel install --user --name myenv --display-name "Python (myenv)"
```
If you have the omara bash aliases (please download from shared drive), this is also available as the command
```console
lily@gavle$ jupyterkernel myenv
```
