# Passive RF Drone Detection for the Georgia Tech Police Department
Team members:
- Anya Bhatnagar
- Kenneth Hyman
- Frederic Faulkner
- James Smith
- Trevor Sorrells

## Summary
This aim of this project is to research and evaluate different methods for passively detecting the presence of drones by analyzing drone and controller Radio Frequency communications. As such, documentation will be divided according by methodology as follows:

1. Hardware Setup
  1. Nvidia Jetson Tegra TK1
  2. HackRF One Software Defined Radio
2. Detection Methodologies
  1. HackRF Sweep
  2. GNU Radio
  3. Wi-Fi Detection

# 1. Hardware Setup
## 1.1 Nvidia Jetson Tegra TK1 Hardware Setup
If any of the detection methodologies are to be deployed in the future, they will need to be hosted on a Linux computer. GTRI provided us with several Jetson
### Important Notes: Read Before Proceeding
NEVER run `sudo apt-get dist-upgrade`. This will overwrite important Jetson Tegra TK1 files and render the system inoperable. Follow NVIDIA's directions for upgrading the system.

Try to use tmux or screen when running important commands over ssh, in case the connection is lost.

SSH: By default, you can only connect to the TK1 using ssh password authentication from the local network. In order to connect remotely, you will have to use public key authentication.

### Initial Hardware Setup
*Note:* Make sure to have a USB-RS232 serial adapter available with Null Modem adapter. This provides a more reliable method of communication with the device than SSH / HDMI when something goes awry. **Note that the serial console is unsecured by default.**

Use a program such as *screen* or *minicom* to connect over serial.
```sh
# Example using screen
sudo screen /dev/ttyUSB0 115200
```

### Flashing the Jetson Tegra TK1
#### Important Notes BEFORE Flashing
While you *may* be able to flash using the Jetson Jetpack GUI, we found it more straightforward to reflash from the command line using Ubuntu. If using the offical Jetson Jetpack to flash, it does not work with Ubuntu 18.04+, requires Ubuntu 16.04 (although Nvidia forums have a fix, it appears to only work for newer Jetson models). Unclear if flashing using this method through a VM is possible.
#### Flashing from 64-bit Ubuntu 18.04
If on a 64-bit host, see [this post](https://devtalk.nvidia.com/default/topic/1037298/jetson-tk1/flash-tk1-from-ubuntu-18-04-/) for a fix:
```sh
# Replace this line of Linux_for_Tegra/flash.sh:
mkfs -t $4 ${loop_dev} > /dev/null 2>&1;
# With this line
mkfs -t $4 -O ^metadata_csum,^64bit ${loop_dev} > /dev/null 2>&1;
```

Flashing the TK1 may be necessary initially or if the system becomes corrupted. To do so, [go to 'Tegra K1' Section](https://developer.nvidia.com/embedded/linux-tegra-archive) and select most recent release. As of the writing of this document, R21.7 is the most recent. Directions for reflashing the device can be found in the *Quick Start Guide*, but make sure to observe the notes above.

#### After Flash
Again, make sure to have serial console access for troubleshooting.
```sh
# Change default password for user ubuntu
# Make sure to make note of the new password.
sudo passwd ubuntu
# Enable the Universe repo and update
sudo add-apt-repository universe
sudo apt-get update
sudo apt-get upgrade
# You may want to install some useful programs at this point
sudo apt-get install -y tmux htop screen git bash-completion
```
To change the hostname of the device change 'tegra-ubuntu' to the desired new name in `/etc/hostname` and `/etc/hosts`. (This will not change until reboot, so you can either reboot or run `sudo hostname YOUR-NEW-HOSTNAME-HERE` to avoid rebooting.)

##### SSH Configuration
**Need to create master `/etc/ssh/sshd_config` so each Jetson has same configuration.** #TODO

Copy pubkeys to the Jetson using `ssh-copy-id -i your-key-here jetson-hostname-here` or manually. #TODO

#### Configure Firewall?
*We should probably configure some sort of firewall...ufw?* #TODO

# Build Instructions for HackRF Tools
Follow directions on https://github.com/mossmann/hackrf/tree/master/host
(This doesn't take very long to build. Maybe a few minutes.)

You will also need to make sure that your user (ubuntu) is a member of the *plugdev* group. Simply enter `sudo usermod -a -G plugdev ubuntu`. May require reboot, then type `groups` to verify that ubuntu is a member of *plugdev*.

The TK1 will auto-suspend the USB by default, so we want to disable this and reboot the tegra. Add the following line to `/etc/rc.local` and then reboot ([source](https://elinux.org/Jetson/Performance#Maximizing_CPU_performance)):
```
echo -1 > /sys/module/usbcore/parameters/autosuspend
```

# Installing GNU Radio
~~To install gnuradio, use gnuradios build system called PyBOMBS (link below). The Ubuntu Universe repository version tends to be outdated. Build gnuradio using PyBOMBS from source following [these directions](https://github.com/gnuradio/pybombs/). This takes a significant amount of time. I had trouble using `sudo pip install PyBOMBS`, so instead use `$ [sudo] pip install git+https://github.com/gnuradio/pybombs.git` to install the latest version from git.~~
** Still in progress **
```sh
sudo apt-get install gnuradio
```
