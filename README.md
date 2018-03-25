## Note to self
To install the driver for EW-7811UTC on ubuntu server 17.10:
```
apt-get update
apt-get install build-essential git ifupdown wireless-tools wpasupplicant
git clone https://github.com/CristianVladescu/rtl8812au.git
cd rtl8812au
# this will fix driver problems with reconnecting, but does not work after reboot
# so it would have to be 'insmod 8812au.ko' after every reboot
#sed -i 's/CONFIG_POWER_SAVING = y/CONFIG_POWER_SAVING = n/' Makefile
make && make install
echo 'auto enx74da38e666c8
iface enx74da38e666c8 inet dhcp
wpa-essid <essid>
wpa-psk <password>' >> /etc/network/interfaces
reboot
```

On disconnect:
`rmmod 8812au && modprobe 8812au`
or reboot

Debugging commands:
```
dmesg - look for
[  182.422346] usb 1-1: reset high-speed USB device number 2 using ehci-pci
[  189.609639] usbcore: registered new interface driver rtl8812au
[  189.610240] rtl8812au 1-1:1.0 enx74da38e666c8: renamed from wlan0
wpa_cli status
iwconfig
tail -f /var/log/syslog
```

====================================================================

## Realtek 802.11ac (rtl8812au)

This is a fork of the Realtek 802.11ac (rtl8812au) v4.2.2 (7502.20130507)
driver altered to build on Linux kernel version >= 3.10.

### Purpose

My D-Link DWA-171 wireless dual-band USB adapter needs the Realtek 8812au
driver to work under Linux.

The current rtl8812au version (per nov. 20th 2013) doesn't compile on Linux
kernels >= 3.10 due to a change in the proc entry API, specifically the
deprecation of the `create_proc_entry()` and `create_proc_read_entry()`
functions in favor of the new `proc_create()` function.

### Building

The Makefile is preconfigured to handle most x86/PC versions.  If you are compiling for something other than an intel x86 architecture, you need to first select the platform, e.g. for the Raspberry Pi, you need to set the I386 to n and the ARM_RPI to y:
```sh
...
CONFIG_PLATFORM_I386_PC = n
...
CONFIG_PLATFORM_ARM_RPI = y
```

There are many other platforms supported and some other advanced options, e.g. PCI instead of USB, but most won't be needed.

The driver is built by running `make`, and can be tested by loading the
built module using `insmod`:

```sh
$ make
$ sudo insmod 8812au.ko
```

After loading the module, a wireless network interface named __Realtek 802.11n WLAN Adapter__ should be available.

### Installing

Installing the driver is simply a matter of copying the built module
into the correct location and updating module dependencies using `depmod`:

```sh
$ sudo cp 8812au.ko /lib/modules/$(uname -r)/kernel/drivers/net/wireless
$ sudo depmod
```

The driver module should now be loaded automatically.

### DKMS

Automatically rebuilds and installs on kernel updates. DKMS is in official sources of Ubuntu, for installation do:

```sh
$ sudo apt-get install build-essential dkms 
```

The driver source must be copied to /usr/src/8812au-4.2.2

Then add it to DKMS:

```sh
$ sudo dkms add -m 8812au -v 4.2.2
$ sudo dkms build -m 8812au -v 4.2.2
$ sudo dkms install -m 8812au -v 4.2.2
```

Check with:
```sh
$ sudo dkms status
```
Eventually remove from DKMS with:
```sh
$ sudo dkms remove -m 8812au -v 4.2.2 --all
```

### References

- D-Link DWA-171
  - [D-Link page](http://www.dlink.com/no/nb/home-solutions/connect/adapters/dwa-171-wireless-ac-dual-band-usb-adapter)
  - [wikidevi page](http://wikidevi.com/wiki/D-Link_DWA-171_rev_A1)
