# Hackintosh Note

This is a sidenote about hackintosh mainly for Acer E5-576G laptop using OpenCore, but some of them also works on E5-574G or similar model (same series, same bios manufacturer and somewhat fragile hinges)

## Hardware specs (E5-576G / E5-574G)

### Things that works

- CPU - i5-8250U / i5-6200U
- iGPU - UHD 620 / HD 520  
  HDMI output might not work well, requires some tweaks with WhateverGreen
- Audio - ALC662  
  Headphone jack is a little buggy
  If you notice high CPU usage on single thread by kernel_task after sleep, **DO NOT PLUG ANYTHING INTO IT**
- Keyboard  
  Most of Fn key works (Except Wi-Fi On/Off, Screen mirroring and Stop button because macOS doesn't have corresponding function)  
  Needs SSDT patch for brightness control
- Trackpad - Elan 501  
  Interrupt mode w/ VoodooI2C didn't work well on E5-574G, might use VoodooPS2 + VoodooInput or applesmarttouchpad instead
- Ethernet - RTL8111 or something like that
- SATA port / DVD drive  
  DVD drive need AppleAHCIPort patch to work properly  
  (E5-574G) If sata device doesn't show up during installation, try SATA-100-series-unsupported.kext
- NVMe SSD (E5-576G only)  
  PCIe 3.0 x2 is fast enough for normal use, but be care of SSD temperature

### Things that doesn't work

- dGPU - MX150 (GT 1030 mobile version?) / 940M  
  macOS doesn't support NVIDIA Optimus(TM), and have no nvidia driver for newer OS anyway
- Card Reader - PCIe one / USB one  
  For card reader that `connected to PCIe`, you can try [Sinetek's RTSX driver](https://www.insanelymac.com/forum/topic/321080-sineteks-driver-for-realtek-rtsx-sdhc-card-readers/)
- Wi-Fi card - Intel Wireless-AC 3168 / Qualcomm Atheros QCNFA435  
  Bluetooth part of card might work somehow, but don't quote me on that  
  BCM94360CS2 + adapter will fit (needs some modification on backplate for E5-576G)

## Installation

### References

- Read [Dortania's OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide/) first, most of thing you need to know how to boot is in there
- Read [OpenCore](https://github.com/acidanthera/OpenCorePkg) document, don't forget to check after every update

### Troubleshooting

#### Stuck at the begining of the boot

1. EC / USBX device not exist  
  macOS needs EC and USBX to make USB work properly
    - Check if fake EC device exists in your custom SSDT / SSDT-EC-USBX
    - Disable or delete it if `rename EC0 to EC` exists in ACPI/Patch

2. Typo in config.plist  
  If you edit config file manually, make sure every entry is correct, especially SSDT and kext name, or OpenCore will hang up and notice you something wrong during boot time
    - Use [ProperTree](https://github.com/corpnewt/ProperTree), press Cmd/Ctrl + Shift + R and select your OC directory

#### Restart / kernel panic after boot
1. DVMT pre-allocated memory size is not equal to or greater then 64 MB
  macOS requires these memory to startup, but default is set to 32 MB in BIOS and couldn't be changed (unless you modded your BIOS)
    - Use `framebuffer-stolenmem` patch with WhateverGreen
    - Try [RU](http://ruexe.blogspot.com/) or mod your bios

2. [CFG lock is not disabled](https://dortania.github.io/OpenCore-Post-Install/misc/msr-lock.html#turning-off-cfg-lock-manually)  
  macOS will try to write some register (MSR 0xE2) in your CPU, but normally it will be locked by manufacturer (unless, again, you modded your BIOS)
    - Apply `AppleCpuPmCfgLock` and `AppleXcpmCfgLock` in config.plist
    - Try [RU](http://ruexe.blogspot.com/) or mod your bios

#### Black screen after boot

1. iGPU works but your screen brightness is at lowest level  
  Everything looks right: verbose text is printing and white progress bar and apple logo shows up, but suddenly everything is gone, not even a backlight, you -- just cranked the brighness down too much ;)
    - Press Fn + Right Arrow to turn the brightness up
    - Press Pause/Break also works somehow
    - Reset NVRam
    - Disable SSDT-PNLF and let your screen always at max brightness (why though)
  > If iGPU (and maybe dGPU) did not recognized correctly by macOS, the screen would still showed up but with some artifacts and glitchs

#### ACPI patch didn't work

1. You disabled the patch  
  The stupid thing we all did before
    - Enable it

2. ACPI patch *should* not work, because it's in your custom SSDT  
  ACPI patch only apply to DSDT/SSDT that read from your BIOS but not your custom one, on the other hand, you could replace a method/device by this mechanism
    - Rewrite your SSDT
  > WhateverGreen also do similar thing by its own, however, is after every SSDT injected and do the patch thing in other place, so it also affect custom one

#### DVD drive shows up in system profiler but doesn't work

1. I don't realy know why but it just works
    - Apply [AppleAHCIPort](https://www.insanelymac.com/forum/files/file/815-appleahciportkext/) patch by vit9696 from comment section (needs some edit to match OpenCore's format)
  

### BIOS Modding

The reason I talked about bios modding but not mentioned GRUB/EFI Shell is that doesn't work on my systems (E5-574G at least) and didn't know that tools like [RU](http://ruexe.blogspot.com/) exists, definitely not because I'm aggresive and want everything unlocked permanently OωO

> Before you do anything below, just give [RU](http://ruexe.blogspot.com/) a try, it doesn't require any special hardware and has *relatively* less risk to brick your laptop  
> [Here](https://www.reddit.com/r/hackintosh/comments/hz2rtm/cfg_lockunlocking_alternative_method/) has a great tutorial by u/Wouter_001 about how to disable CFG lock and *theoretically* it could also change DVMT pre-allocated memory size, enable GPIO and more!

**To be continued**