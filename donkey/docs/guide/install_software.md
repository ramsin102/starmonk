# Install Software

This guide will help you to setup the software to run Donkey on your Raspberry Pi, as well as the host PC operating system of your choice.

* Setup [RaspberryPi](#get-the-raspberry-pi-working)
* Setup [Linux Host PC](#install-donkeycar-on-linux)
* Setup [Windows Host PC](#install-donkeycar-on-windows)
* Setup [Mac Host PC](#install-donkeycar-on-mac)

----
### Get the Raspberry Pi working.

You need to flash a micro SD image with an operating system.

1. Download [Raspian Lite](https://downloads.raspberrypi.org/raspbian_lite_latest) (300MB). 
2. Follow OS specific guides [here](https://www.raspberrypi.org/documentation/installation/installing-images/).
3. Leave micro SD card in your machine and edit/create some files as below:

### Setup the Pi's WiFi for first boot

We can create a special file which will be used to login to wifi on first boot. More reading [here](https://raspberrypi.stackexchange.com/questions/10251/prepare-sd-card-for-wifi-on-headless-pi), but we will walk you through it. 

On Windows, with your memory card image burned and memory disc still inserted, you should see two drives, which are actually two partitions on the mem disc. One is labeled __boot__. On Mac and Linux, you should also have access to the __boot__ partition of the mem disc. This is formated with the common FAT type and is where we will edit some files to help it find and log-on to your wifi on it's first boot.

* Start a text editor: `gedit` on Linux. Notepad on Windows. TextEdit on a Mac.
* Paste and edit this contents to match your wifi:
```
country=US
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
    ssid="<your network name>"
    psk="<your password>"
}

```

Replace `<your network name>` with the ID of your network. Leave the quotes. I've seen problems when the network name contained an apostrophe, like "Joe's iPhone".
Replace `<your password>` with your password, leaving it surrounded by quotes. 
If it bothers you to leave your password unencrypted, you may change the [contents later](https://unix.stackexchange.com/questions/278946/hiding-passwords-in-wpa-supplicant-conf-with-wpa-eap-and-mschap-v2) once you've gotten the pi to boot and log-in.

* Save this file to the root of __boot__ partition with the filename `wpa_supplicant.conf`. On first boot, this file will be moved to `/etc/wpa_supplicant/wpa_supplicant.conf` where it may be edited later. If you are using Notepad on Windows, make sure it doesn't have a .txt at the end.

##### Setup Pi's Hostname
We can also setup the hostname so that your Pi easier to find once on the network. If yours is the only Pi on the network, then you can find it with 

```
ping raspberrypi.local
```

once it's booted. If there are many other Pi's on the network, then this will have problems. If you are on a Linux machine, or are able to edit the UUID partition, then you can edit the `/etc/hostname` and `/etc/hosts` files now to make finding your pi on the network easier after boot. Edit those to replace `raspberrypi` with a name of your choosing. Use all lower case, no special characters, no hyphens, yes underscores `_`. 

```
sudo vi /media/userID/UUID/etc/hostname
sudo vi /media/userID/UUID/etc/hosts
```

##### Enable SSH on boot

Put a file named __ssh__ in the root of your __boot__ partition.


Now you're SD card is ready. Eject it from your computer, put it in the Pi 
and plug in the Pi.


### Connecting to the Pi

If you followed the above instructions to add wifi access you're Pi should
now be connected to your wifi network. Now you need to find it's IP address
so you can connect to it via SSH. 

The easiest way (on Ubuntu) is to use the `findcar` donkey command. You can try `ping raspberrypi.local`. If you've modified the hostname, then you should try: `ping <your hostname>.local`. This will fail on a windows machine. Windows users will need the full IP address (unless using cygwin). 

If you are having troubles locating your Pi on the network, you will want to plug in an HDMI monitor and USB keyboard into the Pi. Boot it. Login with:

* Username: __pi__
* Password: __raspberry__
 
Then try the command:

```
ifconfig wlan0
```

If this has a valid IPv4 address, 4 groups of numbers separated by dots, then you can try that with your SSH command. If you don't see anything like that, then your wifi config might have a mistake. You can try to fix with

```
sudo nano /etc/wpa_supplicant/wpa_supplicant.conf
```

If you don't have a HDMI monitor and keyboard, you can plug-in the Pi with a CAT5 cable to a router with DHCP. If that router is on the same network as your PC, you can try:

```
ping raspberrypi.local
```

Hopefully, one of those methods worked and you are now ready to SSH into your Pi. On Mac and Linux, you can open Terminal. On Windows you can install [Putty](http://www.putty.org/) or [one of the alternatives](https://www.htpcbeginner.com/best-ssh-clients-windows-putty-alternatives/2/).

If you have a command prompt, you can try:

```
ssh pi@raspberrypi.local
```

or

```
ssh pi@<your pi ip address>
```

or via Putty.

* Username: __pi__
* Password: __raspberry__
* Hostname:`<your pi IP address>`

### Update and upgrade

```bash
sudo apt-get update
sudo apt-get upgrade
```

### Raspi-config

```bash
sudo raspi-config
```

* enable I2c
* enable camera
* exapand filesystem
* change hostname
* change default password for pi

> Note: reboot after changing these settings

### Install dependencies

```bash
sudo apt-get update
sudo apt-get install build-essential python3 python3-dev python3-virtualenv python3-numpy python3-picamera python3-pandas python3-rpi.gpio i2c-tools avahi-utils joystick libopenjp2-7-dev libtiff5-dev gfortran libatlas-base-dev libopenblas-dev libhdf5-serial-dev git
```

### Install optional OpenCV dependencies
```bash
sudo apt-get install libilmbase-dev libopenexr-dev libgstreamer1.0-dev libjasper-dev libwebp-dev libatlas-base-dev libavcodec-dev libavformat-dev libswscale-dev libqtgui4 libqt4-test
```

### Setup virtual env

```bash
python3 -m virtualenv -p python3 env --system-site-packages
echo "source env/bin/activate" >> ~/.bashrc
source ~/.bashrc
```

### Install Tensorflow

You can check [this](https://github.com/lhelontra/tensorflow-on-arm/releases/) page to find the one you like. Or install this one:
```bash
wget https://github.com/lhelontra/tensorflow-on-arm/releases/download/v1.10.0/tensorflow-1.10.0-cp35-none-linux_armv7l.whl
pip install tensorflow-1.10.0-cp35-none-linux_armv7l.whl
```

### Install Donkeycar Python code and install

This will pull the latest from Tawn's fork:

```bash
git clone https://github.com/tawnkramer/donkey
pip install -e donkey[pi]
```

### Joystick setup

If you plan to use a joystick, jump over to [here](https://github.com/tawnkramer/donkey/blob/master/docs/parts/controllers.md#physical-joystick-controller).


### Create your car application.

```
donkey createcar --path ~/d2
```

See also [more information.](https://github.com/tawnkramer/donkey/blob/master/docs/utility/donkey/#create-car)

### Calibrate and then Get Driving!

You need to [calibrate](https://github.com/tawnkramer/donkey/blob/master/docs/guide/calibrate.md) your pwm outputs to steering and throttle.

And then [get driving!](https://github.com/tawnkramer/donkey/blob/master/docs/guide/get_driving.md)


----
## Now let's setup things on your PC. Install varies depending on platform.

* Setup [Linux Host PC](#install-donkeycar-on-linux)
* Setup [Windows Host PC](#install-donkeycar-on-windows)
* Setup [Mac Host PC](#install-donkeycar-on-mac)

----
## Install donkeycar on Linux

* Install dependencies, setup virtualenv
```bash
sudo apt-get install virtualenv build-essential python3-dev gfortran libhdf5-dev libatlas-base-dev
virtualenv env -p python3
source env/bin/activate
pip install keras==2.2.2
```

* Install tensorflow. If you have a gpu with nvidia cuda drivers:
```
pip install tensorflow-gpu==1.10.0
```
otherwise:
```
pip install tensorflow==1.10.0
```


* Install donkeycar
```bash
git clone https://github.com/tawnkramer/donkey
pip install -e donkey[pc]
```

----

## Install donkeycar on Windows

* Install [miniconda Python 3.6 64 bit](https://conda.io/miniconda.html). Be sure to check the box to allow it to modify your system path variable to add conda.

* Install [git 64 bit](https://git-scm.com/download/win)

* From the start menu start the Anaconda Prompt.

* Change to a dir you would like to use as the head of your projects.

```
mkdir projects
cd projects
```

* Get the latest donkey from Github.

```
git clone https://github.com/tawnkramer/donkey
cd donkey
```

* Create the Python Anaconda environment

```
conda env create -f envs\windows.yml
activate donkey
```

* Install donkey source and create your local working dir:

```
pip install -e .[pc]
donkey createcar --path ~/d2
```

* Install tensorflow. If you have a gpu with nvidia cuda drivers:
```
pip install tensorflow-gpu==1.10.0
```
otherwise:
```
pip install tensorflow==1.10.0
```

> Note: After closing the Anaconda Prompt, when you open it again, you will need to 
> type ```activate donkey``` to re-enable the mappings to donkey specific 
> Python libraries

----

## Install donkeycar on Mac

* Install [miniconda Python 3.6 64 bit](https://conda.io/miniconda.html)

* Install [git 64 bit](https://www.atlassian.com/git/tutorials/install-git)

* Start Terminal

* Change to a dir you would like to use as the head of your projects.

```
mkdir projects
cd projects
```

* Get the latest donkey from Github.

```
git clone https://github.com/tawnkramer/donkey
cd donkey
```

* Create the Python anaconda environment

```
conda env create -f envs/mac.yml
source activate donkey
```

* Install Tensorflow

```
pip install tensorflow
```


* Install donkey source and create your local working dir:

```
pip install -e .[pc]
donkey createcar --path ~/d2
```

> Note: After closing the Terminal, when you open it again, you will need to 
> type ```source activate donkey``` to re-enable the mappings to donkey specific 
> Python libraries

-------

### Install another fork of donkeycar

Occasionally you may want to run with changes from a separate fork of donkey. You may uninstall one and install another. That's fastest, but leaves you with only the forked version installed:

``` 
pip uninstall donkeycar
git clone --depth=1 https://github.com/<username>/donkey donkey_<username>
cd donkey_<username>
pip install -e .<[pi] or [pc]>
```

To get back to the stock donkey install:

```
pip uninstall donkeycar
git clone --depth=1 https://github.com/wroscoe/donkey donkey
cd donkey
pip install -e .
```
