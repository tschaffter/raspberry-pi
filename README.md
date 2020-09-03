# Raspberry Pi

[![GitHub Stars](https://img.shields.io/github/stars/tschaffter/raspberry-pi.svg?color=94398d&labelColor=555555&logoColor=ffffff&style=for-the-badge&logo=github)](https://github.com/tschaffter/raspberry-pi)
[![GitHub License](https://img.shields.io/github/license/tschaffter/raspberry-pi.svg?color=94398d&labelColor=555555&logoColor=ffffff&style=for-the-badge&logo=github)](https://github.com/tschaffter/raspberry-pi)

Base installation of an hardened Raspberry Pi

## Hardware

- Raspberry Pi 4 Model B 2019 8GB
- SanDisk Extreme 32GB MicroSDHC UHS-3 Card

## Install Raspberry Pi OS

Install [Raspberry Pi OS Lite (32-bit)]. After installing the OS on the SD card,
create an empty file named `ssh` on the boot partition to enable remote
connection to the Pi using SSH.

On Mac OS:

    cd /Volumes/boot && touch ssh

On Windows 10 using Windows Subsystem for Linux (WSL):

    sudo mkdir /mnt/d
    sudo mount -t drvfs D: /mnt/d
    touch /mnt/d/ssh
    sudo umount /mnt/d/

## Boot your new OS

You can now insert the SD card into the Raspberry Pi and power it up.

For the official Raspberry Pi OS, if you need to manually log in, the default
user name is `pi`, with password `raspberry`. After logging to the Pi, change
immediately the default password:

    passwd

## Update the OS

    sudo apt update && sudo apt upgrade -y && sudo apt autoremove -y

## Change your username

It is recommended to create a new user before removing the user `pi` to make the
Pi more secure.

1. Create the new user (replace `abc` by your username).

        sudo -s
        export user=abc
        useradd -m ${user} \
            && usermod -a -G sudo ${user} \
            && echo "${user} ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/${user} \
            && chmod 0440 /etc/sudoers.d/${user} \
            && passwd ${user}

2. Logout and login  using the new user.
3. Delete the default user `pi`.

        sudo deluser -remove-home pi

## Install essentials

    sudo apt-get install -y \
        apt-transport-https \
        ca-certificates \
        && sudo update-ca-certificates

    sudo apt-get install -y \
        bash-completion \
        curl \
        git \
        gnupg2 \
        htop \
        openssh-client \
        sudo \
        tmux \
        vim

## Setup UFW

Install and configure UFW (Uncomplicated Firewall) to protect the Pi against
unauthorized remote connections. Here we only need to open the SSH port (22).

    sudo apt-get install -y ufw
    sudo ufw status
    sudo ufw allow ssh
    sudo ufw enable

## Build the kernel with SELinux support

First install Git and the build dependencies:

    sudo apt install -y git bc bison flex libssl-dev make libncurses-dev

Next get the sources, which will take some time:

    git clone --depth=1 https://github.com/raspberrypi/linux

### Generate default configuration

From the folder `linux`, create the Raspberry Pi 4 default build configuration.

    make bcm2711_defconfig

### Human-friendly customization

Run `make menuconfig` and use the menu to enable the options listed below.

```
General setup
    (-yyyyMMdd-hardened) Enable different security models
```

where `yyyyMMdd` is today's date.

SELinux also requires that audit be enabled in the kernel configuration.

```
General setup
    [*] Auditing support
```

Also, the networking security option must be enabled:

```
Security options
    [*] Enable different security models
    [*]   Socket and Networking Security Hooks
```

Now it is possible to select the SELinux option:

```
Security options
    [*] Enable different security models
    [*] NSA SELinux Support
```

There are also a number of individual SELinux options that you might wish to
enable. Please see the help for the individual different items for more
descriptions on what they do in.

```
Security options
    [*] Enable different security models
    [*] NSA SELinux Support
    [*]   NSA SELinux boot parameter
    [ ]   NSA SELinux runtime disable
    [*]   NSA SELinux Development Support
    [*]   NSA SELinux AVC Statistics
    (1)   NSA SELinux checkreqprot default value
```

Source: [Linuxtopia: Security - Chapter 9.  Kernel Configuration Recipes][linuxtopia_selinux]

### Building

The command below builds the kernel and generates *.deb packages that are
convenient to install.

    make deb-pkg -j$(($(nproc)+1))


## Enable SELinux

    sudo apt install -y selinux-basics selinux-policy-default auditd


Set mode to `enforcing` in `/etc/selinux/config`, then restart the system.

## Install Xbox One Controller

### Install xpadneo driver

The recommended way to instal [xpadneo] is as a kernel module that can be installed,
updated and uninstalled. First, install DKMS (Dynamic Kernel Module Support).

    sudo apt install -y dkms

Then build and install [xpadneo] by following the official instructions.

**Notes**:

> Building [xpadneo] as a kernel modules requires tools available in
`/usr/src/linux-headers-$(uname -r)/scripts`. Cross-compiling the Linux kernel
using `deb-pkg` leads the binaries in this folder to be built for the host and
not for the target architecture. This is a known issue of `deb-pkg`. The binaries
can be recompiled on the Pi using `sudo make scripts` from the scripts folders.
Unfortunately, building these scripts on the Pi for a kernel that has been cross-
compiled may not be easy, and even after successfully building all of them, it
has been observed that [xpadneo] could not be successfully built due to the
following error: `/bin/sh: 1: scripts/mod/modpost: Exec format error`. None of
these issues are met when compiling the Linux kernel on the Pi using `deb-pkg`.

> Another potential issue when install a kernel that has been cross-compiled using
`deb-pkg` with SELinux support is that SELinux header files are not installed to
`/usr/src/linux-headers-$(uname -r)/security/selinux`, leading `sudo make scripts`.

TODO: Add note on disabling "ERTP"

### Connect the controller




sudo service bluetooth stop


<!-- Definitions -->

[Raspberry Pi OS Lite (32-bit)]: https://www.raspberrypi.org/documentation/installation/installing-images/README.md
[xpadneo]: https://github.com/atar-axis/xpadneo
[linuxtopia_selinux]: https://www.linuxtopia.org/online_books/linux_kernel/kernel_configuration/ch09s06.html

