# Hackintosh on Gigabyte GA-H97-D3H-CF with AMD RX 580 8GB via [OpenCore][1]

![About this mac][100]

### **Hardware configuration**

* Intel Core i5 4590
* Gigabyte GA-H97-D3H-CF
* 4×8GB Kingston DDR3 1600MHz
* XFX Radeon RX 580 GTS XXX Edition 8GB (May work on any other versions)
* Samsung 860 EVO 500GB (macOS)
* [Fenvi FV-HB1200][24] WiFi BT PCIe Card (BCM94360CS2 Based) 

### **Before you start make sure you have**

* Working hardware
* [BIOS][15] version `>= F7`
* Read [OpenCore Desktop Guide][16]
* [OpenCore][1] `= 0.5.9`

## Installation

### BIOS Settings

[ BIOS Features][102] and [Peripherals][103]

* *Save & Exit* → Load Optimized Defaults [**Yes**]
* *BIOS Features* → Fast boot [**Disabled**]
* *BIOS Features* → VT-d [**Enabled**]
* *BIOS Features* → Windows 8 Features [**Windows 8 WHQL**]
* *BIOS Features* → CSM Support [**Disabled**]
* *Peripherals* → Initial Display Output [**IGFX**]
* *Peripherals* → XHCI Mode [**Enabled**]
* *Peripherals* → Intel Processor Graphics Memory Allocation [**64M**]
* *Peripherals* → XHCI Hand-off [**Enabled**]
* *Peripherals* → EHCI Hand-off [**Enabled**]
* *Peripherals* → Super IO Configuration → Serial Port A [**Disabled**]
  * Must be disable it in order that macOS Sleep function will work properly.

### What's behind the scenes

* `CFG-Lock / MSR 0xE2` option is [**UNLOCKED**][104].

## Gathering files

- You must download all not bundled kexts and drivers from repositories by yourself.

### ACPI

You can use `SSDT-EC.aml` and `SSDT-PLUG.aml` files, but it probably better to create your own - [SSDTs: The easy way][23]

### EFI drivers

* [HfsPlus.efi][19] - Needed for seeing HFS volumes(ie. macOS Installers and Recovery partitions/images).
* OpenRuntime.efi - Must have to work with native NVRAM
* OpenCanopy.efi - For [OpenCore's GUI][18]

### Kexts

* [VirtualSMC.kext][4] - A advanced replacement of FakeSMC, almost like native mac SMC.
  * SMCProcessor.kext - Used for monitoring CPU temperature.
  * SMCSuperIO.kext - Used for monitoring fan speed.
* [Lilu.kext][3] - Dependency of `VirtualSMC.kext` and `WhateverGreen.kext`
* [WhateverGreen.kext][5] - Need for iGPU support
* [AppleALC.kext][2] - Getting audio to work as easy-peasy `layout-id = 1` defined in `SSDT-EXT.aml`
* [IntelMausi.kext][8] - Intel driver for Ethernet 
* USBH97-D3H-CF.kext - Plist-only kext for USB port mapping

### Resources

- [OcBinaryData][20] - For [Setting up OpenCore's GUI][19]

-----



### Config Property list

Please check `Config Example\config.plist` for post-install config example.

#### Pre-Install

- If you are using dGPU (for example: AMD RX580), under `DeviceProperties` → `PciRoot(0x0)/Pci(0x2,0x0)` :
  - `AAPL,ig-platform-id` = `04001204` [Data]
  - `device-id` = `12040000` [Data]
- If you are using just the iGPU, set `DeviceProperties` → `PciRoot(0x0)/Pci(0x2,0x0)` :
  - `AAPL,ig-platform-id` = `0300220D` [Data]
  - `device-id` = `12040000` [Data]
- Fix DRM for RX580, under `DeviceProperties` add `PciRoot(0x0)/Pci(0x1,0x0)/Pci(0x0,0x0)` dictionary with:
  - `shikigva` = `80` [Number]
- Set AppleALC, under `DeviceProperties` add `PciRoot(0x0)/Pci(0x1B,0x0)` dictionary with:
  - `alc-layout-id` = `01000000` [Data]

![DeviceProperties_rx580][105]

*This is an example for a dGPU(rx580) with DRM fix*

- Populated `PlatformInfo > Generic` section in `config.plist`, can be easily done with `GenSMBIOS` please follow [OpenCore Desktop Guide][7].
- Add the `USBH97-D3H-CF.kext` depends on the model you use `iMac14,1 / iMac14,2 / iMac15,1 ` from `USB Kexts`. (Also add it to your config, you can see an example on `Config Example`)

#### Post-Install

- `Misc -> Boot`
  - Set `PickerMode` as `External` and add files from [Setting up OpenCore's GUI][19]
- `Misc -> Security`
  - Set `ScanPolicy` to `983299` - for more information [Scanpolicy Docs][22]
- `NVRAM -> Add -> 7C436110-AB2A-4BBB-A880-FE41995C9F82 -> boot-args`:
  - Remove `-v` from your config.plist

----

### USB mapping

- First read - [macOS and the 15 Port Limit][21]

Due to these limits disabled interfaces are `HS05, HS06, HS07, HS08 and HS13`. Those are the two usb2 ports near the PS/2 ports & two internal USB 2.0 headers. In addition, interface `HS14` used by `Fenvi FV-HB1200` for bluetooth and configured as internal. I'm using all 15 available ports due the limit but If you want to use other ports use this mapping table and [schema][101] to edit the `Info.plist` file inside the `USBH97-D3H-CF.kext`.

| Name | UsbConnector | port     |
| ---- | ------------ | -------- |
| HS01 | 0            | 01000000 |
| HS02 | 0            | 02000000 |
| HS03 | 0            | 03000000 |
| HS04 | 0            | 04000000 |
| HS05 | 0            | 05000000 |
| HS06 | 0            | 06000000 |
| HS07 | 0            | 07000000 |
| HS08 | 0            | 08000000 |
| HS09 | 0            | 09000000 |
| HS10 | 0            | 0A000000 |
| HS11 | 0            | 0B000000 |
| HS12 | 0            | 0C000000 |
| HS13 | 0            | 0D000000 |
| HS14 | 0            | 0E000000 |
| SS01 | 3            | 10000000 |
| SS02 | 3            | 11000000 |
| SS03 | 3            | 12000000 |
| SS04 | 3            | 13000000 |
| SS05 | 3            | 14000000 |
| SS06 | 3            | 15000000 |

*For Internal USB ports like Bluetooth - use UsbConnector = `255`*

## Issues

1. None :)



Thanks to [Andrii Korzh][25] for his repsotory, knowledge sharing and permission.

---------



## Changelog

###### 01/08/2020 - Init

- Added USB Mapping
- USB Kexts for `iMac14,1 / iMac14,2 / iMac15,1 `
- Config Examples
- GA-H97-D3H-CF bios images
- Using OpenCore's GUI
- Based changed to OpenCore 0.5.9 & macOS Catalina 10.15.5
- Added Changes for Gigabyte GA-H97-D3H-CF
- Initial release (forked from [korzhyk/OpenCore_GA-H97M-D3H][17])

[1]: https://github.com/acidanthera/OpenCorePkg/releases
[2]: https://github.com/acidanthera/AppleALC/releases
[3]: https://github.com/acidanthera/Lilu/releases
[4]: https://github.com/acidanthera/VirtualSMC/releases
[5]: https://github.com/acidanthera/WhateverGreen/releases

[6]: https://lifehacker.com/bypass-a-filevault-password-at-startup-by-rebooting-fro-1686770324
[7]: https://dortania.github.io/OpenCore-Install-Guide/config.plist/haswell.html#platforminfo
[8]: https://github.com/acidanthera/IntelMausi/releases
[9]: https://github.com/acidanthera/AppleSupportPkg
[13]: https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2#Officially_assigned_code_elements
[14]: https://github.com/acidanthera/MacInfoPkg
[15]: https://www.gigabyte.com/Motherboard/GA-H97-D3H-rev-10/support#support-dl-bios
[16]: https://dortania.github.io/OpenCore-Install-Guide/
[17]: https://github.com/korzhyk/OpenCore_GA-H97M-D3H
[18]: https://dortania.github.io/OpenCore-Post-Install/cosmetic/gui.html#setting-up-opencore-s-gui
[19]: https://github.com/acidanthera/OcBinaryData/blob/master/Drivers/HfsPlus.efi
[20]:https://github.com/acidanthera/OcBinaryData
[21]: https://dortania.github.io/OpenCore-Post-Install/usb/#macos-and-the-15-port-limit
[22]: https://dortania.github.io/OpenCore-Post-Install/universal/security.html#scanpolicy
[23]: https://dortania.github.io/Getting-Started-With-ACPI/ssdt-methods/ssdt-easy.html
[24]: https://www.aliexpress.com/item/33034394024.html
[25]: https://github.com/korzhyk
[100]: _static/images/about.png "Abount this mac"
[101]: _static/images/usb_mapping.png "USB Mapping"
[102]: _static/images/bios_features.png "BIOS Features"
[103]: _static/images/bios_peripherals.png "BIOS Peripherals"
[104]: _static/images/cfg_unlock.png "MSR 0xE2 off"
[105]: _static/images/config_device_properties_rx580.png

