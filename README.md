# sushi
Computer set-ups, shell environments, installation notes, etc.

## Setting up a new computer

### Essentials

```console
lily@gavle$ sudo apt-get update
lily@gavle$ sudo apt-get install build-essential git vim gcc-multilib g++-multilib checkinstall sublime openssh-server
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