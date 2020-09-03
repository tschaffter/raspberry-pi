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

## Change the hostname

Edit the file `/etc/hostname`. The new hostname will be effective next time the
system starts.

## Install essentials

    sudo apt install -y \
        apt-transport-https \
        ca-certificates \
        && sudo update-ca-certificates

    sudo apt install -y \
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

    sudo apt install -y ufw
    sudo ufw allow ssh
    sudo ufw enable
    sudo ufw status

## Build the kernel with SELinux support

### Clone the source code

First install Git and the build dependencies.

    sudo apt install -y git bc bison flex libssl-dev make libncurses-dev

Next get the source code. This will take some time.

    git clone --depth=1 https://github.com/raspberrypi/linux

### Generate the default configuration

From the folder `linux`, create the Raspberry Pi 4 default build configuration.

    cd linux
    make bcm2711_defconfig

> Note: Navigate to the section [Kernel building] of the Raspberry Pi website
to ensure that `bcm2711_defconfig` is the latest configuration to use for the
Raspberry Pi 4.

### Customize the configuration

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
    [*] Socket and Networking Security Hooks
```

Now it is possible to select the SELinux option:

```
Security options
    [*] NSA SELinux Support
```

There are also a number of individual SELinux options that you might wish to
enable. Please see the help for the individual different items for more
descriptions on what they do in.

```
Security options
    [*] NSA SELinux Support
    [*]   NSA SELinux boot parameter
    [ ]   NSA SELinux runtime disable
    [*]   NSA SELinux Development Support
    [*]   NSA SELinux AVC Statistics
    (1)   NSA SELinux checkreqprot default value
```

Source: [Linuxtopia: Security - Chapter 9.  Kernel Configuration Recipes][linuxtopia_selinux]

### Building

This command builds the kernel and generates *.deb packages that are convenient
to install. This operation should takes about one hour on a Raspberry Pi 4. We
start the building process from a tmux session so that it does not get interrupted
if the SSH session closes due to inactivity.

    tmux new -s plop
    make deb-pkg -j$(($(nproc)+1))

- Press `Ctrl+B` followed by `D` to detach from a session
- Run `tmux a -t <session-name>` to attach to a session
- Press `Ctrl+C` followed by `Ctrl+D` to close a session

Once the kernel has been successfully completed, step back from the current
directory to find the .deb packages along other build artifacts.

## Install the new kernel

Install the new kernel.

    sudo dpkg -i linux-*.deb

The new kernel image is present in the `/boot` and starts with `vmlinuz-`.
Update the command below accordingly.

    sudo sh -c "echo 'kernel=vmlinuz-5.4.61-20200902-hardened+' >> /boot/config.txt"

Reboot, then check that the new kernel is correctly installed by printing its name.

    uname -a

The the products of the kernel build, including the .deb packages, can be deleted
after having successfully installed the kernel.

## Update the kernel

Go to the `linux` folder that includes the source code of the kernel and update it.

    cd linux
    git pull

Repeat the operations introduced in the above sections to build and install the
kernel. The configuration of the kernel can be skipped, unless a significant
amount of time has elapsed since the last time you have configured it. If it is
the case, then it is recommended to thoroughly clean the folder and reconfigure
again the kernel build.

    make mrproper
    make menuconfig

## Enable SELinux

The kernel installed above include SELinux support. We now need to install the
SELinux software and enable it.

    sudo apt install -y selinux-basics selinux-policy-default auditd
    sudo sh -c "sed -i '$ s/$/ selinux=1 security=selinux/' /boot/cmdline.txt"
    sudo touch /.autorelabel
    sudo reboot

The reboot will take more time than usual as the system needs to update the
label of all the files. Once connected back to the Raspberry Pi, check that
SELinux is in permissive mode with

    sestatus

In permissive mode, SELinux does not block process that violates its rules;
instead violations are simply logged and its up to the user to check the log
regularly.

Set the mode to `enforcing` in `/etc/selinux/config` if you want SELinux to block
processes that are not authorized by one of the policies installed. Restart the
system and check the new mode is correctly set using `sestatus`.

<!-- Definitions -->

[Raspberry Pi OS Lite (32-bit)]: https://www.raspberrypi.org/documentation/installation/installing-images/README.md
[linuxtopia_selinux]: https://www.linuxtopia.org/online_books/linux_kernel/kernel_configuration/ch09s06.html
[Kernel building]: https://www.raspberrypi.org/documentation/linux/kernel/building.md
