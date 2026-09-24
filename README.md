# Lenovo IdeaPad Flex 5 14IIL05 — OpenCore EFI for macOS Tahoe 26.x

OpenCore 1.0.7 EFI that boots macOS Tahoe on a **Lenovo IdeaPad Flex 5 14IIL05** (Intel Ice
Lake), using the **OCLP-CustoMac** fork for the required root patches (Modern Wi-Fi and
Modern Audio). Everything except HDMI output and the headphone jack is working; see the
status table below for the honest list.

---

## Tested hardware

| Component | Model | Notes |
| :--- | :--- | :--- |
| CPU | Intel Core i5-1035G1 | Ice Lake, 4C/8T, 15 W |
| iGPU | Intel UHD Graphics (G1), 32 EU | real device id `0x8A56`, spoofed to `0x8A5C` |
| RAM | 8 GB DDR4-3200 | soldered, single channel, not upgradeable |
| Display | 14" FHD 1920x1080 touch | |
| Audio | Realtek ALC257 | AppleALC layout 11 |
| Wi-Fi / BT | Dell DW1820A (BCM94350ZAE) | M.2 2230; see "known issues" |
| Keyboard / touchpad | PS/2 keyboard, I2C touchpad | VoodooPS2 + VoodooI2C |
| Storage | NVMe | |

---

## Status

| Feature | State |
| :--- | :--- |
| Boot (OpenCore, OpenCanopy GUI, no verbose) | working |
| Graphics acceleration | working — Metal 3, 3072 MB dynamic VRAM |
| Backlight control / brightness keys | working (`-igfxblr` + BrightnessKeys) |
| Internal audio (speakers) | working after the Modern Audio root patch |
| Wi-Fi 5 GHz | working — 80 MHz, 802.11ac, 867 Mbps observed |
| Bluetooth | works, but a BLE mouse disconnects under continuous use |
| Sleep / wake | works; Bluetooth may need a nudge after wake |
| Touchpad | working |
| SD card reader | not tested |
| **HDMI output** | **not working — Ice Lake LSPCON, no known fix** |
| **Headphone jack** | **not detected (see known issues)** |

---

## Requirements

1. **macOS Tahoe 26.x** (this EFI was validated on 26.x).
2. **OCLP-CustoMac** — the fork, not upstream OCLP. Upstream OCLP will refuse the root
   patches this machine needs on Tahoe. Apply both **Modern Wi-Fi** and **Modern Audio**
   root patches after installation.
   Modern Audio requires a Kernel Debug Kit, which requires working networking, so get
   Wi-Fi up first and then tick Modern Audio.
3. Your own SMBIOS values (section above).

## BIOS settings

* Secure Boot — disabled
* Intel SGX — disabled
* SATA mode — AHCI
* Graphics mode — UMA only (only exposed on some SKUs)
* VT-d — disabled, or keep enabled and leave `DisableIoMapper` as shipped here

## Installation outline

1. Create a Tahoe installer USB, replace its `EFI` folder with this one.
2. Generate your own SMBIOS values and put them into `EFI/OC/config.plist` **before** you
   install — changing them later means re-authenticating everything.
3. Boot the installer and install macOS.
4. Boot the installed system, then run OCLP-CustoMac and apply the **Modern Wi-Fi** and
   **Modern Audio** root patches, then reboot.
5. Sign in / set up iMessage only after the root patches are applied and Wi-Fi is working.

---

## Notable configuration decisions

### Boot arguments

| Argument | Owner | Purpose |
| :--- | :--- | :--- |
| `-igfxdbeo` | WhateverGreen | Ice Lake: avoids the 10–15 s garbled/black screen after boot |
| `igfxfw=2` | WhateverGreen | force-load the iGPU firmware (Ice Lake stability) |
| `-igfxblr` | WhateverGreen | backlight register fix — macOS brightness is otherwise very dim |
| `-noDC9` | (Ice Lake display power state) | avoids black screen + panic after wake on Ice Lake |
| `-vi2c-force-polling` | VoodooI2C | force polling instead of GPIO interrupts for I2C input |
| `watchdog=0` | kernel | disable the watchdog timer |
| `-revsbvmm` | RestrictEvents | force VMM SB model so OTA updates are accepted |
| `alcid=11` | AppleALC | audio layout 11 |
| `-amfipassbeta` | AMFIPass | AMFI exemption without fully disabling SIP |
| `brcmfx-aspm=0` | AirportBrcmFixup | disable ASPM on the Broadcom card |
| `brcmfx-country=#a` | AirportBrcmFixup | country code `#a` — unlocks all 5 GHz channels |
| `-btlfxboardid` | BlueToolFixup | board-id spoof needed for Bluetooth firmware upload |
| `-bwfxbeta` | BlueWakeFixup | Bluetooth wake handling |

### Other choices

* **USB map** — every port carries both Tahoe-era keys (`UsbConnector` *and*
  `usb-port-type`), with the three internal ports declared `255`/`255`. Under Tahoe a port
  missing either key is treated as invalid.
* **Boot cosmetics** — verbose mode removed, picker timeout `1` (note: `0` means *wait
  forever*, not *skip*).
* **Audio** — layout 11 is AppleALC's "Lenovo T480" layout for ALC257. Speakers work.

---

## Known issues and open leads

* **HDMI.** Ice Lake on macOS has no working external display output — documented as
  unfixed since 2021 in Ice Lake hackintosh trackers. A USB-C DisplayLink adapter is the
  practical workaround.
* **Headphone jack.** The jack works under Windows but is never detected by macOS, so it
  is a configuration problem, not hardware. All eight ALC257 layouts shipped by AppleALC
  (11, 18, 86, 96, 97, 99, 100, 101) were tried — none routes the jack. The next step is
  to read the codec's real pin configuration (`cat /proc/asound/card0/codec#0` on any
  Linux live USB) and compare the jack's node against what the layouts expect.
* **Bluetooth.** Firmware upload works (`v26119 c4689`) and the controller gets a valid
  address, but a Logitech MX Anywhere 2S disconnects while being moved. Wi-Fi is stable at
  the same time, which points at the DW1820A's BT link or the antenna path — the fix is a
  supported Broadcom card such as a BCM943602 / BCM94360NG module (M.2 2230).
* **Performance.** 8 GB of soldered RAM on a 15 W quad-core is the floor for Tahoe. Expect
  lag under multitasking; `Reduce transparency` and `Reduce motion` help noticeably.
* **Untested lever:** the Ice Lake community recommends `AAPL,ig-platform-id`
  `0x8A510002` (the MacBookAir9,1 default) over the `0x8A5C0001` used here, for more
  reliable wake from sleep. Not tried in this configuration.

---

## Repository layout

```
EFI/                        the OpenCore EFI (config.plist is sanitised)
  BOOT/BOOTx64.efi
  OC/ACPI/                  SSDTs (AWAC, PLUG, PNLF, EC-USBX, I2C, XOSI, ...)
  OC/Drivers/               OpenRuntime, OpenCanopy, AudioDxe, HfsPlus, ResetNvram
  OC/Kexts/                 all kexts, versions listed in docs/CONFIG-NOTES.md
  OC/Resources/             OpenCanopy theme + boot chime
  OC/Tools/                 OpenShell, CleanNvram, ResetSystem, ...
docs/CONFIG-NOTES.md        generated inventory: boot-args, kexts, quirks, iGPU props
Tools/macserial/            macserial (linux / macOS / Windows) for generating a serial
```

---

## Credits

* [OpenCore](https://github.com/acidanthera/OpenCorePkg) and all Acidanthera kexts — Lilu,
  WhateverGreen, VirtualSMC, AppleALC, AirportBrcmFixup, BrcmPatchRAM, BlueToolFixup,
  RestrictEvents, NVMeFix, CpuTscSync.
* [OCLP-CustoMac](https://github.com/kgp-macPro/OCLP-CustoMac) — the patcher fork that
  makes Modern Wi-Fi and Modern Audio work on Tahoe.
* [VoodooI2C](https://github.com/VoodooI2C/VoodooI2C) and
  [VoodooPS2](https://github.com/acidanthera/VoodooPS2) for input.
* The Ice Lake hackintosh community, whose notes on `-noDC9`, `-igfxdbeo`, `-igfxblr` and
  the HDMI limitation informed this configuration, and the public Flex 5 hackintosh repos
  that provided the starting point.

All kexts and tools remain under their authors' licences.

## Disclaimer

This EFI is published as-is, for the exact hardware listed above. Hackintosh configurations
are device-specific — if your lid, screen, Wi-Fi card or BIOS revision differs, expect to
adjust `config.plist` yourself. Nothing here is endorsed by or affiliated with Apple or
Lenovo.
