# Hackintosh OpenCore on MSI MPG Z490 Gaming Plus

I've successfully installed macOS Catalina and updated to the latest 10.15.7 (19H1519) on my i7-10700K (Comet Lake) running on a MSI MPG Z490 GAMING PLUS (no Wifi) motherboard. I plan to update to Big Sur and Monterey, but *starting* with Catalina tends to be much easier and more stable while working out hardware details like USB mapping.

You can find my `config.plist` in this repository and many notes about current results and my setup process in this document. I carefully followed [Dortania's OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide/), [Getting started with ACPI](https://dortania.github.io/Getting-Started-With-ACPI/), [Multiboot with OpenCore](https://dortania.github.io/OpenCore-Multiboot/), and [OpenCore Post-Install](https://dortania.github.io/OpenCore-Post-Install/), and I also had to look up some details in [OpenCore Reference Manual](https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/Configuration.pdf). If you're going to build a system like this, you should also read these documents carefully before starting and keep them nearby during the setup process.

<img src="https://github.com/infinitewarp/hackintosh-opencore-msi-z490/blob/main/images/screenshot-neofetch.png"/>

## Hardware

- CPU: [Intel Core i7-10700K 3.8 GHz 8-Core Processor](https://pcpartpicker.com/product/yhxbt6/intel-core-i7-10700k-38-ghz-8-core-processor-bx8070110700k) (Comet Lake)
- iGPU: Intel UHD Graphics 630
- dGPU: [EVGA GeForce RTX 3060 Ti 8 GB XC GAMING Video Card](https://pcpartpicker.com/product/ZdpmP6/evga-geforce-rtx-3060-ti-8-gb-xc-gaming-video-card-08g-p5-3663-kr)
    - *Only* used when multibooting other OSes. *macOS completely ignores this GPU.*
- Motherboard: [MSI MPG Z490 GAMING PLUS ATX LGA1200 Motherboard](https://pcpartpicker.com/product/YNTzK8/msi-mpg-z490-gaming-plus-atx-lga1200-motherboard-mpg-z490-gaming-plus)
    - Audio: Realtek ALC1200
    - Ethernet: 2.5Gbit Realtek RTL8125B
    - Rear USB: 1x 3.2 Gen 2 (10 Gbps) Type C, 1x 3.2 Gen 2 (10 Gbps) Type A, 2x 3.2 Gen 1 (5 Gbps) Type A
    - Video: 1x DisplayPort, 1x HDMI
- WiFi & Bluetooth: [fenvi T919 PCI-E BCM94360CD](https://pcpartpicker.com/product/CBPgXL/fenvi-t919-for-macos-pc-wifi-card-continuity-handoff-bcm94360cd-native-airport-wifi-bt-40-1750mbps-5ghz24ghz-3x3-mimo-abgnac-beamforming-wlan-pci-e-card-no-driver-needed-for-macos)
- Memory: [G.Skill Ripjaws V 32 GB (2 x 16 GB) DDR4-3600 CL16 Memory](https://pcpartpicker.com/product/zcH8TW/gskill-ripjaws-v-32-gb-2-x-16-gb-ddr4-3600-memory-f4-3600c16d-32gvkc)
- Storage: [Samsung 980 1 TB M.2-2280 NVME Solid State Drive](https://pcpartpicker.com/product/mKBG3C/samsung-980-1-tb-m2-2280-nvme-solid-state-drive-mz-v8v1t0bam)
- Power supply: [EVGA G5 650 W 80+ Gold Certified Fully Modular ATX Power Supply](https://pcpartpicker.com/product/3gJmP6/evga-g5-650-w-80-gold-certified-fully-modular-atx-power-supply-220-g5-0650-x1)
- Case: [Fractal Design Meshify 2 ATX Mid Tower Case](https://pcpartpicker.com/product/Tjwkcf/fractal-design-meshify-2-atx-mid-tower-case-fd-c-mes2a-05)

## OpenCore required reading

<img src="https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/Logos/OpenCore_with_text_Small.png" width="200" height="48"/>

- [OpenCore 0.7.5 RELEASE](https://github.com/acidanthera/OpenCorePkg/releases/tag/0.7.5)
- [Dortania's OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide/)
- [Getting started with ACPI](https://dortania.github.io/Getting-Started-With-ACPI/)
- [OpenCore Multiboot](https://dortania.github.io/OpenCore-Multiboot/)
- [OpenCore Post-Install](https://dortania.github.io/OpenCore-Post-Install/)
- [OpenCore Reference Manual](https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/Configuration.pdf)

## Verified functionality

This is not an exhaustive list. It mostly covers use cases I personally use or care about.

- :white_check_mark: on-board DisplayPort (iGPU, UHD 630)
    - Note: DisplayPort use was *required* during the initial macOS Catalina install.
- :white_check_mark: 3D and video acceleration
    - Verified Safari, Chess, and Maps. Without acceleration, these would crash immediately upon opening.
- :white_check_mark: Built-in stereo line-out sound
    - Available as output device "Internal Speakers".
    - I normally have wired speakers using the rear stereo line-out. Plugging headphone into my case's *front* line-out is *not autodetected*, but they will be detected and start working after a quick sleep/wake cycle.
- :white_check_mark: Built-in line-in/microphone audio
    - Available as input device "Internal Microphone".
- :white_check_mark: Wi-Fi and Bluetooth
    - Verified WiFI using 802.11ac with WPA2.
    - Verified Bluetooth using AirDrop to and from real nearby Macs.
- :white_check_mark: Ethernet (at least gigabit)
    - Verified by connecting directly to another over ethernet computer and testing with `iperf3`.
    - The computer on the other end only supports gigabit ethernet, and the test sufficiently maxed that out. I cannot confirm if the full 2.5 Gbps speed is available.
        <details>
        <summary>iperf3 server-side output</summary>
        <pre>
        ❯ iperf3 -s
        -----------------------------------------------------------
        Server listening on 5201 (test #1)
        -----------------------------------------------------------
        Accepted connection from 192.168.2.3, port 60656
        [  5] local 192.168.2.1 port 5201 connected to 192.168.2.3 port 60657
        [ ID] Interval           Transfer     Bitrate
        [  5]   0.00-1.00   sec   111 MBytes   928 Mbits/sec
        [  5]   1.00-2.00   sec   112 MBytes   940 Mbits/sec
        [  5]   2.00-3.00   sec   112 MBytes   940 Mbits/sec
        [  5]   3.00-4.00   sec   112 MBytes   940 Mbits/sec
        [  5]   4.00-5.00   sec   112 MBytes   940 Mbits/sec
        [  5]   5.00-6.00   sec   112 MBytes   940 Mbits/sec
        [  5]   6.00-7.00   sec   112 MBytes   940 Mbits/sec
        [  5]   7.00-8.00   sec   112 MBytes   940 Mbits/sec
        [  5]   8.00-9.00   sec   112 MBytes   940 Mbits/sec
        [  5]   9.00-10.00  sec   112 MBytes   940 Mbits/sec
        [  5]  10.00-10.02  sec  1.80 MBytes   942 Mbits/sec
        - - - - - - - - - - - - - - - - - - - - - - - - -
        [ ID] Interval           Transfer     Bitrate
        [  5]   0.00-10.02  sec  1.09 GBytes   939 Mbits/sec                  receiver
        -----------------------------------------------------------
        Server listening on 5201 (test #2)
        -----------------------------------------------------------
        </pre>
        </details>
- :white_check_mark: Sleep and wake from sleep via keyboard/mouse
    - System also wakes up and goes back to sleep periodically for the "Power Nap" setting.
- :white_check_mark: Native NVRAM
    - Verified by clearing with `nvram -c`, setting a value with `nvram`, rebooting, and checking `nvram -p`.
- :white_check_mark: App Store
    - Logged in, searched, downloaded apps, opened downloaded apps from within the App Store app.
- :white_check_mark: iCloud services
    - Messages sending, receiving, and tapback all work normally.
    - Also various services that sync data like Notes, Calendar, etc.
- :white_check_mark: macOS Software Updates
    - Installed all available updates from Apple after the clean install.
    - Automatic rebooting throughout the update install process also works.
- :white_check_mark: Multiboot
    - Verified macOS and Linux on different partitions of the same NVMe and on a separate SATA SSD.
    - Haven't tested with Windows on another partition or drive, but that should work fine too.
- :white_check_mark: Lots of third-party apps working
    - Firefox, mpv, VLC, Adobe Photoshop, Blender, DaVinci Resolve, Handbrake, iTerm2, and many more apps all seem stable.
    - Ditto for numerous command-line programs installed via `brew`.

## Not working, not tested, or unknown

- :grey_question: on-board HDMI
    - I switched to DisplayPort during initial install because the installer image gave only a black screen when using HDMI. I haven't bothered switching back to HDMI after completing the installation.
- :x: Netflix in Safari
    - Native "Fairplay" DRM requires a supported AMD graphics card; I don't have one.
    - Firefox is my primary browser, and it supports Netflix DRM via the proprietary Widevine CMI plugin perfectly fine on the iGPU, as far as I can tell.
- :grey_question: Hibernate
    - Hibernate *probably* works since normal sleep and wake work, but I don't need it for a desktop on a UPS.
- :grey_question: Facetime and Photo Booth
    - Need to get an external video camera to test these, but I suspect they'll work fine.
- :grey_question: Filevault
    - Filevault *probably* works, but I don't have it enabled on this system (yet).
- :grey_question: Apple Music, Apple TV
    - Apple Music *probably* works, but Apple TV *probably doesn't* work since it likely requires hardware Fairplay DRM decoding.
- :grey_question: digital audio out and surround sound
    - Related options appear in System Preferences: Sound: Output, but I don't have the appropriate audio equipment to test.

## Binaries

I have *not* included the various binaries for AMLs, drivers, kexts, etc. in this repo. You should *never* simply clone someone else's EFI repo and use it without verifying all of its parts! The contents of my actual EFI file tree and their sources are:

- EFI
    - BOOT
        - `BOOTx64.efi` from [OpenCore-0.7.5-RELEASE.zip](https://github.com/acidanthera/OpenCorePkg/releases/download/0.7.5/OpenCore-0.7.5-RELEASE.zip)
    - OC
        - ACPI
            - `SSDT-AWAC.aml` from [Getting-Started-With-ACPI](https://github.com/dortania/Getting-Started-With-ACPI/blob/master/extra-files/compiled/SSDT-AWAC.aml)
            - `SSDT-EC-USBX-DESKTOP.aml` from [Getting-Started-With-ACPI](https://github.com/dortania/Getting-Started-With-ACPI/blob/master/extra-files/compiled/SSDT-EC-USBX-DESKTOP.aml)
            - `SSDT-PLUG-DRTNIA.aml` from [Getting-Started-With-ACPI](https://github.com/dortania/Getting-Started-With-ACPI/blob/master/extra-files/compiled/SSDT-PLUG-DRTNIA.aml)
            - `SSDT-RHUB.aml` from [Getting-Started-With-ACPI](https://github.com/dortania/Getting-Started-With-ACPI/blob/master/extra-files/compiled/SSDT-RHUB.aml)
        - Drivers
            - `HfsPlus.efi` from [OcBinaryData](https://github.com/acidanthera/OcBinaryData/blob/master/Drivers/HfsPlus.efi)
            - `OpenLinuxBoot.efi` from [OpenCore-0.7.5-RELEASE.zip](https://github.com/acidanthera/OpenCorePkg/releases/download/0.7.5/OpenCore-0.7.5-RELEASE.zip)
            - `OpenPartitionDxe.efi` from [OpenCore-0.7.5-RELEASE.zip](https://github.com/acidanthera/OpenCorePkg/releases/download/0.7.5/OpenCore-0.7.5-RELEASE.zip)
            - `OpenRuntime.efi` from [OpenCore-0.7.5-RELEASE.zip](https://github.com/acidanthera/OpenCorePkg/releases/download/0.7.5/OpenCore-0.7.5-RELEASE.zip)
            - `OpenUsbKbDxe.efi` from [OpenCore-0.7.5-RELEASE.zip](https://github.com/acidanthera/OpenCorePkg/releases/download/0.7.5/OpenCore-0.7.5-RELEASE.zip)
            - `ext4_x64.efi` from [OcBinaryData](https://github.com/acidanthera/OcBinaryData/blob/master/Drivers/HfsPlus.efi)
        - Kexts
            - `AppleALC.kext` from [AppleALC-1.6.6-RELEASE.zip](https://github.com/acidanthera/AppleALC/releases/download/1.6.6/AppleALC-1.6.6-RELEASE.zip)
            - `CPUFriend.kext` from [CPUFriend-1.2.4-RELEASE.zip](https://github.com/acidanthera/CPUFriend/releases/download/1.2.4/CPUFriend-1.2.4-RELEASE.zip)
            - `CPUFriendDataProvider.kext` generated using [CPUFriendFriend](https://github.com/corpnewt/CPUFriendFriend) (which also needs to download [ResourceConverter.sh](https://raw.githubusercontent.com/acidanthera/CPUFriend/master/Tools/ResourceConverter.sh) and [iasl.zip](https://bitbucket.org/RehabMan/acpica/downloads/iasl.zip))
            - `Lilu.kext` from [Lilu-1.5.7-RELEASE.zip](https://github.com/acidanthera/Lilu/releases/download/1.5.7/Lilu-1.5.7-RELEASE.zip)
            - `LucyRTL8125Ethernet.kext` from [LucyRTL8125Ethernet-V1.1.0.zip](https://github.com/Mieze/LucyRTL8125Ethernet/releases/download/1.1.0/LucyRTL8125Ethernet-V1.1.0.zip)
            - `NVMeFix.kext` from [NVMeFix-1.0.9-RELEASE.zip](https://github.com/acidanthera/NVMeFix/releases/download/1.0.9/NVMeFix-1.0.9-RELEASE.zip)
            - `SMCProcessor.kext` from [VirtualSMC-1.2.7-RELEASE.zip](https://github.com/acidanthera/VirtualSMC/releases/download/1.2.7/VirtualSMC-1.2.7-RELEASE.zip)
            - `SMCSuperIO.kext` from [VirtualSMC-1.2.7-RELEASE.zip](https://github.com/acidanthera/VirtualSMC/releases/download/1.2.7/VirtualSMC-1.2.7-RELEASE.zip)
            - `USBMap.kext` generated using [USBMap](https://github.com/corpnewt/USBMap)
            - `VirtualSMC.kext` from [VirtualSMC-1.2.7-RELEASE.zip](https://github.com/acidanthera/VirtualSMC/releases/download/1.2.7/VirtualSMC-1.2.7-RELEASE.zip)
            - `WhateverGreen.kext` from [WhateverGreen-1.5.5-RELEASE.zip](https://github.com/acidanthera/WhateverGreen/releases/download/1.5.5/WhateverGreen-1.5.5-RELEASE.zip)
        - `OpenCore.efi` from [OpenCore-0.7.5-RELEASE.zip](https://github.com/acidanthera/OpenCorePkg/releases/download/0.7.5/OpenCore-0.7.5-RELEASE.zip)
        - Tools
            - `OpenShell.efi` from [OpenCore-0.7.5-RELEASE.zip](https://github.com/acidanthera/OpenCorePkg/releases/download/0.7.5/OpenCore-0.7.5-RELEASE.zip)
        - `config.plist` customized from [OpenCore-0.7.5-RELEASE.zip](https://github.com/acidanthera/OpenCorePkg/releases/download/0.7.5/OpenCore-0.7.5-RELEASE.zip)

## USB Port Mapping

My custom map removes all ports I couldn't identify, the internal Mystic Light RGB controller, and the USB type C port on the case front.

Here are all the USB ports I could identify in my testing:

| class                | port            | locationID | where     | my notes                                   |
|----------------------|-----------------|------------|-----------|--------------------------------------------|
| `AppleUSB20XHCIPort` | ` 1 (01000000)` | `14100000` | external  | `USB 3.2 Gen 1 5Gbps Type A` (upper, blue) |
| `AppleUSB20XHCIPort` | ` 2 (02000000)` | `14200000` | external  | `USB 3.2 Gen 1 5Gbps Type A` (lower, blue) |
| `AppleUSB20XHCIPort` | ` 3 (03000000)` | `14300000` | external  | `USB 3.2 Gen 2 10Gpbs Type A` (red)        |
| `AppleUSB20XHCIPort` | ` 4 (04000000)` | `14400000` | external  | `USB 3.2 Gen 2 10Gpbs Type C`              |
| `AppleUSB20XHCIPort` | ` 5 (05000000)` | `14500000` | *unknown* | *unknown*                                  |
| `AppleUSB20XHCIPort` | ` 6 (06000000)` | `14600000` | *unknown* | *unknown*                                  |
| `AppleUSB20XHCIPort` | ` 7 (07000000)` | `14700000` | internal  | `JUSB4` (front USB 3.0 Type A, left)       |
| `AppleUSB20XHCIPort` | ` 8 (08000000)` | `14800000` | internal  | `JUSB4` (front USB 3.0 Type A, right)      |
| `AppleUSB20XHCIPort` | ` 9 (09000000)` | `14900000` | external  | `USB 2.0 Type-A` (lower, black)            |
| `AppleUSB20XHCIPort` | `10 (0a000000)` | `14a00000` | external  | `USB 2.0 Type-A` (upper, black)            |
| `AppleUSB20XHCIPort` | `11 (0b000000)` | `14b00000` | internal  | `JUSB2` (Bluetooth PCIe card)              |
| `AppleUSB20XHCIPort` | `12 (0c000000)` | `14c00000` | internal  | `MYSTIC LIGHT` (RGB LED controller)        |
| `AppleUSB20XHCIPort` | `13 (0d000000)` | `14d00000` | internal  | `JUSB5` (front USB 3.1 Type C)             |
| `AppleUSB20XHCIPort` | `14 (0e000000)` | `14e00000` | *unknown* | *unknown*                                  |
| `AppleUSB20XHCIPort` | `15 (0f000000)` | `14f00000` | *unknown* | *unknown*                                  |
| `AppleUSB20XHCIPort` | `16 (10000000)` | `14000000` | *unknown* | *unknown*                                  |
| `AppleUSB30XHCIPort` | `17 (11000000)` | `14100000` | external  | `USB 3.2 Gen 1 5Gbps Type A` (upper, blue) |
| `AppleUSB30XHCIPort` | `18 (12000000)` | `14200000` | external  | `USB 3.2 Gen 1 5Gbps Type A` (lower, blue) |
| `AppleUSB30XHCIPort` | `19 (13000000)` | `14300000` | external  | `USB 3.2 Gen 2 10Gpbs Type C`              |
| `AppleUSB30XHCIPort` | `20 (14000000)` | `14400000` | external  | `USB 3.2 Gen 2 10Gpbs Type A` (red)        |
| `AppleUSB30XHCIPort` | `21 (15000000)` | `14500000` | *unknown* | *unknown*                                  |
| `AppleUSB30XHCIPort` | `22 (16000000)` | `14600000` | *unknown* | *unknown*                                  |
| `AppleUSB30XHCIPort` | `23 (17000000)` | `14700000` | internal  | `JUSB4` (front USB 3.0 Type A, left)       |
| `AppleUSB30XHCIPort` | `24 (18000000)` | `14800000` | internal  | `JUSB4` (front USB 3.0 Type A, right)      |
| `AppleUSB30XHCIPort` | `25 (19000000)` | `14900000` | internal  | `JUSB5` (front USB 3.1 Type C)             |
| `AppleUSB30XHCIPort` | `26 (1a000000)` | `14a00000` | *unknown* | *unknown*                                  |

## Issues encountered along the journey

Setting up OpenCore and getting to a working macOS desktop is not always an easy paint-by-numbers process. I did a fair bit of experimenting and digging through random forum posts and other people's `EFI` folders to figure out how to get my system to a reliable state. [Dortania's OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide/) *and* the related docs are *absolutely critical reading materials*. Be very familiar with the process and technologies involved. Be comfortable working with the command line, restarting many times, checking your work, tweaking configs, rinse, and repeat.

Some specific issues I encountered and resolved include:

- Blank screen when booting the macOS Catalina installer.
    - Resolved by switching from the HDMI port to the DisplayPort one.
- USB ports and devices randomly disconnecting or not working.
    - Resolved by temporarily using `USBInjectAll.kext` with `XhciPortLimit` enabled, following the [USB port-mapping guide](https://dortania.github.io/OpenCore-Post-Install/usb/), building the kexts, and reconfiguring with the specific ports I want to keep.
- macOS installer not recognizing a SSD on an internal SATA connector.
    - Resolved by adding `SATA-unsupported.kext`.
- Garbled output or "no entry"/prohibited symbol during boot.
    - This seemed to happen randomly but moderately frequently during my early setup, and I never quite pinned down the exact cause.
    - Since completing the [USB port-mapping guide](https://dortania.github.io/OpenCore-Post-Install/usb/) process, though, I have not seen this problem again. macOS's USB port detection may just really *really* bad and unpredictable with this motherboard's ACPI/DSDT.
        <details>
        <summary>Example photo of scrambled/garbled video.</summary>
        <img src="https://github.com/infinitewarp/hackintosh-opencore-msi-z490/blob/main/images/boot-garbage.jpg"/>
        </details>
- Broken video acceleration. Many apps like Safari would immediately crash on open.
    - Resolved by setting `device-id=9B3E0000`.
    - Where does this magic value come from? See the [OpenCore Reference Manual](https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/Configuration.pdf) section 6.1. That value (and several related ones) came from `ioreg` output processed by `gfxutil`.
- No audio working on any line-out ports.
    - Resolved by setting `boot-args-alcid=7`. Or maybe `11`. Or maybe `69`. Nice.
    - I stepped through every apparently available ID for my hardware, and these three all seem to function the same way.
    - As mentioned above, this works fine for the rear line-out, but the front line-out is not autodetected when I plug in headphones. Putting the system to sleep and waking it back up triggers whatever hardware refresh is necessary to start using the front line-out again. It's a minor inconvenience, but it's a sufficiently reliable workaround.
    - I tried the `VoohooHDA.kext`, but it wasn't really any better than those options, and Voodoo never seeemed to recognize my front-panel audio ports.
