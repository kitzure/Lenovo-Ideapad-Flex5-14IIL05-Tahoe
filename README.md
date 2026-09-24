# Hackintosh OpenCore EFI — Lenovo IdeaPad Flex 5 14IIL05 (macOS Tahoe 26.x)

OpenCore EFI and notes for running macOS **Tahoe** on the **Lenovo IdeaPad Flex 5 14IIL05**
(Intel Ice Lake). The setup depends on the **OCLP-CustoMac** fork for the root patches that
Tahoe needs — upstream OCLP will not apply them.

Credit for the starting point belongs to the four public projects for this laptop family listed
at the bottom of this file; this repository is the same machine taken to Tahoe, with the
Broadcom Wi-Fi and Bluetooth chain, the Tahoe USB map and the boot cosmetics redone.

---

## Specifications

| Component | Model | Notes |
| :--- | :--- | :--- |
| CPU | Intel Core i5-1035G1 | Ice Lake, 4 cores / 8 threads, 15 W |
| iGPU | Intel UHD Graphics (G1), 32 EU | real device id `0x8A56`, injected as `0x8A5C` |
| RAM | 8 GB DDR4-3200 | soldered to the board — not upgradeable |
| Display | 14" FHD 1920x1080, 10-point touch | |
| Audio | Realtek ALC257 | AppleALC, layout 11 |
| Wi-Fi | Dell DW1820A (BCM94350ZAE) | M.2 2230, replaced the stock card |
| Bluetooth | BCM2045A0 (on the DW1820A) | same card, internal USB device |
| Keyboard | PS/2 | VoodooPS2Controller |
| Trackpad | HID I2C | VoodooI2C + `SSDT-I2C-TPAD.aml` |
| Storage | NVMe | NVMeFix |

---

## Working status

| Component | State | Notes |
| :--- | :---: | :--- |
| Built-in display | ✅ | Thanks to `WhateverGreen.kext` and the Ice Lake DeviceProperties. Accelerated — Metal 3, 3072 MB dynamic VRAM. |
| Brightness / brightness keys | ✅ | `-igfxblr` (macOS renders these panels dimmer than Windows without it) plus `BrightnessKeys.kext`. |
| Speakers | ✅ | Thanks to `AppleALC.kext`. Requires the **Modern Audio** root patch. |
| Headphone jack | ❌ | Playback is never routed to the jack under macOS — see Known issues. |
| Microphone | ✅ | Internal mic works; the headset mic on the combo jack shares the jack problem. |
| USB ports | ✅ | Port map with the four Tahoe-era keys per port; internal ports declared `255`. |
| Battery / charging | ✅ | `VirtualSMC.kext` + `SMCBatteryManager.kext`, plus `ECEnabler.kext`. |
| Sleep / wake | ✅ | Works; Bluetooth can need a nudge after wake (`sudo killall -9 bluetoothd`). |
| Trackpad | ✅ | Thanks to `SSDT-I2C-TPAD.aml` by [@Micael106](https://github.com/Micael106), with VoodooI2C in polling mode. |
| Keyboard | ✅ | Thanks to `VoodooPS2Controller.kext`. |
| Wi-Fi | ✅ | 5 GHz, 80 MHz, 802.11ac, 867 Mbps observed. Needs the **Modern Wi-Fi** root patch. |
| Bluetooth | ⚠️ | Firmware uploads and the controller gets a valid address, but a BLE mouse disconnects while moving — see Known issues. |
| AirDrop / Continuity | ✅ | Comes with the restored Apple wireless stack. |
| HDMI output | ❌ | Ice Lake has no working external display path on macOS — no fix exists. |
| SD card reader | ❓ | Not tested. |

---

## SMBIOS — generate your own before you use this

`config.plist` ships a **generated placeholder identity**, not a working Mac one:

| Field | Shipped value |
| :--- | :--- |
| `SystemSerialNumber` | `C02FWHYLML7H` |
| `MLB` | `C021251304NP8PGJC` |
| `SystemUUID` | `3B20C1D8-AE0F-46C8-B41A-23D320777CDE` |
| `ROM` | `11:22:33:44:55:66` |

Boot it as-is and macOS runs, but **iMessage/FaceTime will not work**, and everyone who
downloads this repository shares one serial. Generate your own pair with
[GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) or OpenCore's `macserial`
(`macserial -m MacBookPro16,2` gives a serial and its matching board serial), put them into
`PlatformInfo -> Generic` (`SystemSerialNumber` + `MLB`), set `SystemUUID` from `uuidgen`, and
set `ROM` to your own network adapter's MAC address. The same values belong in the
`PlatformInfo -> SMBIOS` block.

---

## Requirements

1. **macOS Tahoe 26.x.**
2. **[OCLP-CustoMac](https://github.com/kgp-macPro/OCLP-CustoMac)** — the fork. Apply both the
   **Modern Wi-Fi** and **Modern Audio** root patches after installing.
   Modern Audio needs a Kernel Debug Kit, which needs working networking — so get Wi-Fi up
   first, then tick Modern Audio and press *Start Root Patching*.

## BIOS settings

* Secure Boot — **disabled**
* Intel SGX — **disabled** (optional)
* SATA mode — **AHCI**
* Graphics mode — **UMA only** (only exposed on some SKUs)
* Intel VT-d — disabled, or enabled with `DisableIoMapper` left as shipped here

## Installation outline

1. Create a Tahoe installer USB and replace its `EFI` folder with this one.
2. **Generate your own SMBIOS first** (section above) — changing it later means re-signing in
   to everything.
3. Install macOS, boot it, then run OCLP-CustoMac and apply the Modern Wi-Fi and Modern Audio
   root patches, and reboot.
4. Only then sign in to iCloud/iMessage.

---

## Boot arguments

| Argument | Owner | Purpose |
| :--- | :--- | :--- |
| `-igfxdbeo` | WhateverGreen | Ice Lake: avoids the garbled/black screen for the first seconds after boot |
| `igfxfw=2` | WhateverGreen | force-load the iGPU firmware |
| `-igfxblr` | WhateverGreen | backlight register fix — otherwise the panel is very dim |
| `-noDC9` | Ice Lake display power state | avoids black screen + panic after wake on Ice Lake |
| `-vi2c-force-polling` | VoodooI2C | use polling instead of GPIO interrupts for I2C input |
| `watchdog=0` | kernel | disable the watchdog timer |
| `-revsbvmm` | RestrictEvents | force the VMM SB model so OTA updates are accepted |
| `alcid=11` | AppleALC | audio layout 11 (the ALC257 layout this machine works with) |
| `-amfipassbeta` | AMFIPass | AMFI exemption without fully disabling SIP |
| `brcmfx-aspm=0` | AirportBrcmFixup | disable ASPM on the Broadcom card |
| `brcmfx-country=#a` | AirportBrcmFixup | country code unlocking the full 5 GHz channel set |
| `-btlfxboardid` | BlueToolFixup | board-id spoof required for the Bluetooth firmware upload |
| `-bwfxbeta` | BlueWakeFixup | Bluetooth post-wake handling |

`docs/CONFIG-NOTES.md` is generated from `config.plist` and lists the exact ACPI files, kexts,
drivers and quirks in this build.

---

## Kext notes

* **Audio** — `AppleALC.kext` with `alcid=11`, injected as a `layout-id` at
  `PciRoot(0x0)/Pci(0x1F,0x3)`. ALC257 also ships layouts 18, 86, 96, 97, 99, 100 and 101; all
  eight were tested on this machine and none of them routes the headphone jack.
* **Wi-Fi / Bluetooth** — `IOSkywalkFamily.kext` + `IO80211FamilyLegacy.kext` +
  `AirportBrcmFixup.kext` + `AMFIPass.kext` for the Wi-Fi side, and `BrcmPatchRAM3.kext` +
  `BrcmFirmwareData.kext` + `BlueToolFixup.kext` + `BlueWakeFixup.kext` for Bluetooth.
  Raise `BrcmPatchRAM3` and `BrcmFirmwareData` **together** or not at all — the newer
  `BrcmPatchRAM3` declares a dependency on a matching `BrcmFirmwareStore`.
* **Trackpad / touchscreen** — `VoodooI2C.kext` with `VoodooI2CHID.kext`, driven in polling
  mode by `-vi2c-force-polling`.
* **USB** — `USBToolBox.kext` + `UTBMap.kext`. Exactly one port map may be active.

---

## Known issues

* **Headphone jack** — works under Windows, never under macOS, so it is a configuration
  problem rather than hardware. All eight ALC257 layouts behave the same, and AppleALC's
  ALC257 layout definitions are byte-identical from 1.7.5 through master, so this is not a
  kext-version issue either. The remaining path is reading the codec's own pin configuration
  (`cat /proc/asound/card0/codec#0` from a Linux live USB) and comparing it with what the
  layouts expect.
* **Bluetooth** — stable enough for keyboards and headsets, but a Logitech BLE mouse drops the
  link while being moved, with Wi-Fi up and stable at the same time. Points at the DW1820A's
  Bluetooth link or its antenna path; the fix is a natively supported Broadcom card (BCM943602
  / BCM94360NG, M.2 2230). Note that the Bluetooth radio shares one of the card's antenna
  chains, so swapping the two antenna leads is worth trying.
* **HDMI** — no external display output on Ice Lake under macOS. Unfixed since 2021 in the Ice
  Lake hackintosh trackers. A USB-C DisplayLink adapter is the practical workaround.
* **Performance** — 8 GB of soldered RAM on a 15 W quad-core is the floor for Tahoe. Expect
  lag when multitasking. `Reduce transparency` and `Reduce motion` in Accessibility → Display
  are the free wins. Not thermal: `pmset -g therm` reports `CPU_Speed_Limit = 100`.

---

## Credits

This EFI builds on four public projects for the IdeaPad Flex 5 family — many thanks to their
authors, whose configurations, fixes and write-ups made this machine work:

* **[BestRazer](https://github.com/BestRazer/Lenovo-IdeaPad-Flex5-14IIL05-Hackintosh)** — IdeaPad Flex 5 14IIL05 OpenCore EFI; the working Ice Lake iGPU DeviceProperties and the component/kext breakdown this README is modelled on.
* **[RobyRew](https://github.com/RobyRew/Lenovo-Ideapad-Flex-5-14IIL05_Hackintosh_OpenCore)** — IdeaPad Flex 5 14IIL05 hackintosh guide; the audio layout, installation guide and the GenSMBIOS instructions.
* **[Micael106](https://github.com/Micael106/Lenovo-Ideapad-Flex-5-14IIL05)** — the trackpad fix, `SSDT-I2C-TPAD.aml`, which is the file used here.
* **[chundk](https://github.com/chundk/Hackintosh-4-Lenovo-Ideapad-Flex5-15IIL05-Guide-OpenCore-EFI)** — the Flex 5 15IIL05 guide and EFI for the same platform, source of the BIOS settings and kext-by-kext notes.

And the upstream projects everything here stands on:

* **[OpenCore](https://github.com/acidanthera/OpenCorePkg)** and the Acidanthera kexts — Lilu, WhateverGreen, VirtualSMC, AppleALC, AirportBrcmFixup, BrcmPatchRAM, BlueToolFixup, RestrictEvents, NVMeFix, CpuTscSync, BrightnessKeys, VoodooPS2.
* **[OCLP-CustoMac](https://github.com/kgp-macPro/OCLP-CustoMac)** — the patcher fork that makes Modern Wi-Fi and Modern Audio work on Tahoe.
* **[VoodooI2C](https://github.com/VoodooI2C/VoodooI2C)** and **[USBToolBox](https://github.com/USBToolBox/kext)**.
* **[Mirone](https://github.com/Mirone)** — `BlueWakeFixup.kext`, the post-wake Bluetooth fix, and Wi-Fi Patcher Pro.

All kexts, patches and guides remain under their authors' licences and terms.

## Disclaimer

Published as-is for the exact hardware listed above. Hackintosh configurations are
device-specific: if your panel, Wi-Fi card or BIOS revision differs, expect to adjust
`config.plist` yourself. Nothing here is endorsed by or affiliated with Apple or Lenovo.
