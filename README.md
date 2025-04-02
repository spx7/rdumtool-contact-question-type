# rdumtool - RDTech UM24C/UM25C/UM34C Bluetooth interface tool

*This script is currently in an early stage and could change significantly.*

The RDTech (RuiDeng) [UM24C](https://www.aliexpress.com/item/RD-UM24-UM24C-for-APP-USB-2-0-LCD-Display-Voltmeter-ammeter-battery-charge-voltage-current/32845522857.html), [UM25C](https://www.aliexpress.com/store/product/RD-UM25-UM25C-for-APP-USB-2-0-Type-C-LCD-Voltmeter-ammeter-voltage-current-meter/923042_32855845265.html) and [UM34C](https://www.aliexpress.com/store/product/RD-UM34-UM34C-for-APP-USB-3-0-Type-C-DC-Voltmeter-ammeter-voltage-current-meter/923042_32880908871.html) are low-cost USB pass-through power measurement devices, and support a decent number of collection features, as well as full control via Bluetooth.  (The non-C versions of these devices support the same features as the C versions, but without Bluetooth control.)  This program implements all exposed commands and data collection available by the device's Bluetooth interface.

## Compatibility

 * UM24C support is complete and tested.
 * UM25C and UM34C support should be complete, but has not been tested.  Hardware should arrive by March 2019.
 * Tested under Python 3.6, but should work with 3.4 or later.
 * Linux: Tested fine with both PyBluez (direct) and pyserial (e.g. /dev/rfcomm0 via ```rfcomm bind```).
 * Windows: Tested fine with pyserial (e.g. COM4 as set up automatically by Windows).  Author could not get PyBluez compiled/installed.
 * MacOS: When using pyserial (e.g. /dev/cu.UM24C-Port as set up automatically by MacOS), writes to the device would succeed (e.g. 0xf2 to rotate the screen), but reads from the device never arrive.  Author could not get PyBluez compiled/installed.

## Setup

rdumtool requires Python 3, and [PyBluez](https://pypi.org/project/PyBluez/) and/or [pyserial](https://pypi.org/project/pyserial/) modules.  Installation varies by operating system, but on Debian/Ubuntu, these are available via the python3-bluez and python3-serial packages.
Also install python3-setuptools (at least at Kubuntu noble 24.4)

To install rdumtool:

```
$ sudo python3 setup.py install
```

rdumtool may also be run directly from its source directory without installation.

## Bluetooth setup

Varies by operating system.  If the pairing procedure asks for a PIN, enter "1234" verbatim.

For command-line installation on Linux:

```
$ bluetoothctl
Agent registered
[bluetooth]# scan on
Discovery started
[NEW] Device 00:90:72:56:98:D7 UM24C
[CHG] Device 00:90:72:56:98:D7 RSSI: -60
[bluetooth]# pair 00:90:72:56:98:D7
Attempting to pair with 00:90:72:56:98:D7
[CHG] Device 00:90:72:56:98:D7 Connected: yes
Request PIN code
[UM241m[agent] Enter PIN code: 1234
[CHG] Device 00:90:72:56:98:D7 UUIDs: 00001101-0000-1000-8000-00805f9b34fb
[CHG] Device 00:90:72:56:98:D7 ServicesResolved: yes
[CHG] Device 00:90:72:56:98:D7 Paired: yes
Pairing successful
[bluetooth]# trust 00:90:72:56:98:D7
[CHG] Device 00:90:72:56:98:D7 Trusted: yes
Changing 00:90:72:56:98:D7 trust succeeded
[bluetooth]# exit
Agent unregistered
```

Device MAC address will vary.  Again, the PIN for the device is "1234".

If you then want to use rdumtool via pyserial, bind it via rfcomm:

```
$ sudo rfcomm bind 0 00:90:72:56:98:D7
```

## Usage

To get device information via PyBluez:

```
$ rdumtool --device-type=UM24C --bluetooth-device=00:90:72:56:98:D7
```

Or via pyserial:

```
$ rdumtool --device-type=UM24C --serial-device=/dev/rfcomm0
```

Without any additional arguments, rdumtool will display all information available from the device.  There are various additional flags to set parameters in the device such as threshold logging; see ```rdumtool --help``` for details.

## Example

```
$ rdumtool --device-type=UM24C --bluetooth-device=00:90:72:56:98:D7 --rotate-screen --set-record-threshold=0.20

$ rdumtool --device-type=UM24C --bluetooth-device=00:90:72:56:98:D7
rdumtool 1.0
Copyright (C) 2019 Ryan Finnie

USB:  5.08V,  0.166A,  0.843W,   30.6Ω
Data:  2.99V(+),  0.00V(-), charging mode: unknown (normal)
Recording (on) :    0.009Ah,    0.046Wh,  197 sec at >= 0.20A
Data groups:
     0:    0.015Ah,    0.077Wh       5:    0.000Ah,    0.000Wh
     1:    0.000Ah,    0.000Wh       6:    0.000Ah,    0.000Wh
    *2:    0.040Ah,    0.204Wh       7:    0.000Ah,    0.000Wh
     3:    0.000Ah,    0.000Wh       8:    0.000Ah,    0.000Wh
     4:    0.000Ah,    0.000Wh       9:    0.000Ah,    0.000Wh
Temp:  25C ( 77F)
Screen: 1/6, brightness: 5/5, timeout: 2 min
```

## Miscellaneous

Thanks to [the sigrok wiki](https://sigrok.org/wiki/RDTech_UM24C) for a reverse engineering of the UM24C's communication protocol, which was referenced to produce this tool.

Thanks to RDTech for their power management devices.  The author currently has a UM24C, and a DPS3003 DC-DC power supply, both of which are excellent quality and performance for their price.  Even more thanks would be in order if they released proper protocol specifications and design schematics, hint hint.
