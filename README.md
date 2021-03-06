openSUSE nvidia-prime like package
==================================

Assumptions
-----------

* You are running openSUSE Tumbleweed
* You don't have bumblebee installed
* You installed nvidia drivers using http://opensuse-community.org/nvidia.ymp

Installation/usage
------------------

1. Run "prime-select nvidia" log out and login again, hopefully you are
   using nvidia GPU. To switch back to intel GPU run "prime-select intel" (modesetting driver) or 
   "prime-select intel2" (Intel Open Source driver, requires xf86-video-intel package).
   Remember to run as root.
2. To check which GPU you're currently using run "prime-select get-current".
3. In intel configurations, powering off the NVIDIA card with bbswitch to save power and decrease temperature is supported but requires additional manual setup. Refer to instructions below.

Contact
-------

* Bo Simonsen <bo@geekworld.dk>
* Michal Srb <msrb@suse.com>

Related projects
----------------

* SUSEPrimeQT <https://github.com/simopil/SUSEPrimeQt/> Provides a simple GUI for SUSEPrime

NVIDIA power off support
-------------------------

Powering off the NVIDIA card when not in use is very efficient for significantly decreasing power consumption (thus increase battery life) and temperature. However, this is complicated by the fact that the card can be powered off
only when the NVIDIA kernel modules are not loaded.

### Install bbswitch

bbswitch is the kernel module that makes it possible to power off the NVIDIA card entirely.
Install it with:

```
zypper in bbswitch
```

### Blacklist the nvidia modules so it can be loaded only when necessary

The NVIDIA openSUSE package adds the NVIDIA driver modules to the kernel initrd image. This will make the system always load them on boot. This is problematic for disabling the NVIDIA card with bbswitch as it can only turn off the card when the modules are not loaded. Instead of unloading the modules before making use of bbswitch, the reverse is way easier: have the NVIDIA modules always unloaded and load them only when needed.
To prevent the modules from being automatically loaded on boot, we need to blacklist them in initrd.
This is easily done with:

```
cp /etc/prime/09-nvidia-blacklist.conf /etc/modprobe.d
dracut -f
```

This will also blacklist the `nouveau` module which can really get in the way with Optimus and causing black screens.

### Install the systemd services for doing switch and set correct card during boot

```
cp /etc/prime/prime-select.service           /usr/lib/systemd/system
systemctl enable prime-select
```

If nvidia is set, it will load the NVIDIA modules before starting the Graphical Target.
Moreover, if an intel config is set but the Intel card was disabled in BIOS (leaving only the dGPU), this service will automatically switch to the nvidia config.


## FAQ

### How do I select a driver ?

prime-select `<driver>`

Where `<driver>` is one of:

- `intel`: use the `modesetting` driver
- `intel2`: use the `intel` driver (xf86-video-intel)
- `nvidia`: use the NVIDIA binary driver


### How do I check the current driver configured and the power state of the NVIDIA card ?

```
/usr/sbin/prime-select get-current
Driver configured: intel
[bbswitch] NVIDIA card is OFF
```

To get more details on the Xorg driver, install package `inxi` if necessary and use `inxi -G`:

```
inxi -G
Graphics:  Device-1: Intel UHD Graphics 630 driver: i915 v: kernel 
           Device-2: NVIDIA GP107GLM [Quadro P600 Mobile] driver: N/A 
           Display: x11 server: X.Org 1.20.4 driver: intel resolution: 3840x2160~60Hz 
           OpenGL: renderer: Mesa DRI Intel UHD Graphics 630 (Coffeelake 3x8 GT2) v: 4.5 Mesa 18.3.4
```
