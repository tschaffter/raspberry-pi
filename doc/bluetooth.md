# Bluetooth

## Hardware

| HCI Device Name | Bluetooth | HCI* |
| --- | ----------- | --- |
| Built-in Bluetooth of the Raspberry Pi 4 | 5.0 | `hci0` |
| ASUS USB-BT400 USB Adapter | 4.0 | `hci1` |

\* These HCI identifiers may be different on your system.

## Nomenclature

- Host Controller Interface (HCI)

## Add user to the group bluetooth

Add the user to the group `bluetooth` to authorize him to communicate with
Bluetooth devices. The permissions given to this group are defined in
`/etc/dbus-1/system.d/bluetooth.conf`.

    sudo usermod --append --groups bluetooth $(whoami)

## Check that devices are not blocked

Check that Bluetooth devices are not blocked (should say "no").

    $ rfkill
    ID TYPE      DEVICE      SOFT      HARD
    0 wlan      phy0     blocked unblocked
    1 bluetooth hci0   unblocked unblocked
    3 bluetooth hci1   unblocked unblocked

If it is blocked, unblock it by ID or TYPE using the commands below.

    sudo rfkill unblock <ID>
    sudo rfkill unblock <TYPE>

To soft block a device, that is, blocked by software

    sudo rfkill block <ID>

## Install bluetoothctl

`bluetoothctl` is the cli tool provided by [BlueZ] to control Bluetooth devices
on Linux. Contrary to what the name's structure might lead you to expect,
`bluetoothctl` is not part of systemd.

Install `bluetoothctl`

    sudo apt-get install pi-bluetooth bluez

Enable the bluetooth service to start at boot and check its status

    sudo systemctl enable bluetooth
    sudo systemctl status bluetooth

> Note: "ctl" in `bluetoothctl`, `systemctl` and `journalctl` stands for
"control".

## What to do first when Bluetooth experiences issues

Try one of the following approaches when Bluetooth does not behave as expected.

Restart the systemd service `bluetooth`

    sudo systemctl restart bluetooth

Reset the Bluetooth receiver `hciX`

    sudo hciconfig hci0 reset

Remove device when using `bluetoothctl` before adding it again

    bluetoothctl
        remove XX:XX:XX:XX:XX:XX

## Configure devices with bluetoothctl

Start the interactive program `bluetoothctl` as a non-root user.

    bluetoothctl

If the command hangs on first use, make sure that the user is a member of the
group `bluetooth`.

    groups

List of useful commands, typically types in this order to connect the first
device.

| Bluetoothctl Option | Description |
| --- | ----------- |
| `help` | Show list of options |
| `agent on` | Enable agent that manages the Bluetooth 'pairing code' |
| `scan <on/off>` | Scan for devices |
| `list` | List Bluetooth controllers (`hciX`) |
| `devices` | Show list of devices that are available, connected or not |
| `pair <MAC>` | Pair with device |
| `connect <MAC>` | Connect device |
| `trust <MAC>` | Trust device so it can automatically connect |
| `remove <MAC>` | Remove device |

## Configure devices with hciconfig and hcitool

[BlueZ] also provides the following commands that can be used as an alternative
to the interactive program `bluetoothcl`.

- `hciconfig`: configure Bluetooth devices
- `hcitool`: configure Bluetooth connections

List Bluetooth receivers (`hciX`)

    hciconfig -a

The above command can be used to get the following information:

- Bus: UART, USB
- MAC address of the HCI device
- Whether the HCI device is up or down
- RX and TX stats
- Bluetooth version

Same as above but returns only the MAC addresses of the devices

    hcitool dev

Turn down/up the HCI device and get its status

    hciconfig hciX up
    hciconfig hciX down
    hciconfig hciX

Display active connections with remote devices

    hcitool con

Get information from remote device

    hcitool info XX:XX:XX:XX:XX:XX

## Dump HCI data

Install `hcidump`

    sudo install -y bluez-hcidump

Dump packages received by `hciX` to stdout (sudo required)

    sudo hcidump -i hci0 --hex

Prefix each lines with the timestamp. See [strftime.org] for a list of the time
formats supported.

    sudo apt install moreutils
    sudo hcidump -i hci0 --hex | ts '%s'

## Discover, read, and write characteristics with gatttool

We can discover, read, and write characteristics with gatttool. GATT stands for
Generic Attribute and defines a data structure for organizing characteristics
and attributes. Launch gatttool in interactive mode.

## Update Bluetooth and Logitech Unifying firmwares

[fwupd] is an open-source cli and daemon for managing the installation of
firmware updates on Linux-based systems. Here we use it to update the firmware
of Logitech Unifying adapters and Bluetooth USB dongles.

The Linux Vendor Firmware Service (LVFS) provides resources and support for
helping vendors package their firmware updates to support the use of this
framework, and serves as an online repository for obtaining these updates.

Install fwupd

    sudo apt install -y --no-install-recommends fwupd

Show client and daemon versions

    $ fwupdmgr --version
    client version: 1.2.13
    daemon version: 1.2.13

Get all devices that support firmware updates

    fwupdmgr get-devices

Refresh metadata from remote server

    fwupdmgr refresh

Gets the list of updates for connected hardware

    fwupdmgr get-updates

Updates all firmware to latest versions available

    sudo fwupdmgr update

> Note: How to update a single device without physically disconnecting the
devices that should not be updated?

Erase all firmware update history to prevent. This can be a solution to make
**fwupd** stop asking to upload usage metrics to its remote servers.

    fwupdmgr clear-history

Check the status of the daemon

    sudo systemctl status fwupd

> Note: Running a command `fwupdmgr [OPTIONS]` starts the daemon if it is
stopped.


<!-- Definitions -->

[BlueZ]: http://www.bluez.org/
[fwupd]: https://github.com/fwupd/fwupd
[strftime.org]: https://strftime.org