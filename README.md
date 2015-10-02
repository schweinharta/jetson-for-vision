# How to: Jetson TK1

## Prepping your host machine

In order to get a Jetson TK1 running, you need a host machine running linux, ideally Ubuntu. Essentially all the support documents are written assuming that this is the case. Obviously, if you already have a computer running Ubuntu, you can skip this. 

If you don't alredy have such a computer, you're still in luck. You can get linux running on Windows and Mac computers for free using [Ubuntu](http://www.ubuntu.com/download/desktop) and [VirtualBox](https://www.virtualbox.org/wiki/Downloads).

The following instructions apply to your **host machine** either running OS X or Windows. Download the latest Ubuntu ISO from the link above as well as the latest VirtualBox. Use VirtualBox to install a new linux virtual machine. Select something like 50GB for the hard disk size and select the option that dynamically allocates memory to this hard disk (so that the virtual hard drive doesn't actually take up 50GB on your real, non-virtual hard drive until you actually put that much stuff in it).

Next, install VirtualBox's ["guest additions"](http://virtualboxes.org/doc/installing-guest-additions-on-ubuntu/).

```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install build-essential module-assistant
sudo m-a prepare
```

At the top of your VirtualBox window, click on `Devices` and then `Insert Guest Additions CD image...` and then

```
sudo sh /media/cdrom/VBoxLinuxAdditions.run
```


## Flashing a new/broken Jetson TK1

These steps basically come from [this forum post](https://devtalk.nvidia.com/default/topic/823132/embedded-systems/-customkernel-the-grinch-21-3-4-for-jetson-tk1-developed/) from the person who maintains the Grinch build for Jetson.

First step is to install "Linux for Tegra". Using your **host machine** do the following to prepare the L4T files

```
wget http://developer.download.nvidia.com/embedded/L4T/r21_Release_v3.0/Tegra124_Linux_R21.3.0_armhf.tbz2
wget http://developer.download.nvidia.com/embedded/L4T/r21_Release_v3.0/Tegra_Linux_Sample-Root-Filesystem_R21.3.0_armhf.tbz2
tar -xvf Tegra124_Linux_R21.3.0_armhf.tbz2
cd Linux_for_Tegra/rootfs
sudo tar xpf ../../Tegra_Linux_Sample-Root-Filesystem_R21.3.0_armhf.tbz2
cd ..
sudo ./apply_binaries.sh
```

Next, activate the force recovery mode on the Jetson TK1. Hold down the `FORCE RECOVERY` button. While that is held down, press and release the `RESET` button. After about 2 seconds, release the `FORCE RECOVERY` button.

Then, connect your Jetson to your host machine via the usb micro cable. If you are using Ubuntu through VirtualBox, be sure to connect your physical device's USB to your virtual machine. Select the USB symbol in the bottom right corner of VirtualBox and ensure that the NVIDIA line is checked.

Then flash the Jetson,

```
sudo ./flash.sh -S 14580MiB jetson-tk1 mmcblk0p1
```

This will take about 30 minutes. When this is finished, press the reset button again and ensure that the Jetson boots into Ubuntu successfully.

Now, from your Jetson, we will install the Grinch Kernel. My preferred method is to use the [jetsonhacks](http://jetsonhacks.com/) script.

First, install git onto your Jetson

```
sudo apt-get update
sudo apt-get install git
```

Next, clone the script and then run it

```
git clone https://github.com/jetsonhacks/installGrinch.git
cd installGrinch
./installGrinch.sh
```

Now, your Jetson should be running smoothley with many of the most common drivers pre-installed.

## Important

Accoriding to [this](http://elinux.org/Jetson_TK1#An_important_step_before_connecting_the_Jetson_to_Internet), executing the following is a big deal to ensure that your NVidia drivers are not overwritten:

```
sudo apt-mark hold xserver-xorg-core
```

## Install CUDA

I followed [these isntructions](http://elinux.org/Jetson/Installing_CUDA) to install CUDA.

## SSH

(how to connect to a single board computer that is connected to the same local area network)

Find the board computer on the network. To do this, search the network map with

```
nmap -sP 10.0.1.*
``` 

(you may have to install nmap; also, you may need to replace `10.0.1.*` with the prefix for your local network). 

Next list all the local devices with 

```arp -a```

Look for the board computer using the MAC address (Jetsons begin with `0:4:4b`). Make note of the ip address. SSH into it using 

```ssh ubuntu@10.0.1.X``` 

where X is the last bit of the IP address you found in the last step. The password is also `ubuntu` (these are the defaults and probably should be changed before the boards are actually used).

### Transfering a single file
(copied from [here](http://unix.stackexchange.com/questions/106480/how-to-copy-files-from-one-machine-to-another-using-ssh))


To copy a file from your computer to the Jetson while logged into your computer (make sure that ubuntu has write priveleges for /path/to/destination):

```
scp /path/to/file ubuntu@10.0.1.X:/path/to/destination
```

### Exiting

When you're done with ssh, you can leave simply with

```
exit
```

## Install things

First first thing is to follow the "Important" heading above.

First thing is to add repositories and update all of your packages:
```
sudo apt-add-repository universe
sudo apt-add-repository multiverse
sudo apt-get update
```

Ubuntu/Debian use Aptitude to manage packages. So things are installed wiht, e.g.,  `apt-get instal emacs` (replace `emacs` with whatever you want to install). You probably will have to appends `sudo ` to the beginning to use super-user priveleges. You may find it easier to use `sudo -i` but some people think this is a bad habit. 

If you want to search aptitude for a package use `apt-cache search X` to search for X.
