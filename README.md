# Lenovo IdeaPad Flex 5 14IIL05 with macOS Tahoe (26.6.2)
![ai gen banner lol](/docs/info.png)
Before using the prebuilt EFI, make sure you know what you're doing. <br>
For instructions on how to install it, visit [here](https://dortania.github.io/OpenCore-Install-Guide/)

---

## Specifications

| Component | Model | Notes |
| :--- | :--- | :--- |
| CPU | Intel Core i5-1035G1 | |
| GPU | Intel UHD Graphics (G1)|  |
| RAM | 8 GB DDR4-3200 | soldered to the board — not upgradeable |
| Storage | ADATA SX8200PNP 512GB | NOT STOCK |
| Display | 14" FHD 1920x1080, 10-point touch | |
| Audio | Realtek ALC257 | |
| Wi-Fi | Dell DW1820A (BCM94350ZAE) | replaced the stock card, intel method should be [here](https://www.reddit.com/r/hackintosh/comments/1ntlejq/finally_another_success_story_with_tahoe_macos_260/) |
| Bluetooth | BCM2045A0 (on the DW1820A) | same card, internal USB device |
| Keyboard | PS/2 | VoodooPS2Controller |
| Trackpad | HID I2C | VoodooI2C + `SSDT-I2C-TPAD.aml` |


---

## Working status
Almost 100%, but still need to fix.
| Component | State | Notes |
| :--- | :---: | :--- |
| Built-in display | ✅ | Thanks to `WhateverGreen.kext` and the Ice Lake DeviceProperties. Accelerated — Metal 3, 3072 MB dynamic VRAM. |
| Audio | ⚠️ | Thanks to `AppleALC.kext`. Requires the **Modern Audio** root patch. Still, microphone need to find it yourself if possible |
| USB ports | ✅ | Port map with the four Tahoe-era keys per port; internal ports declared `255`. |
| Battery / charging | ✅ | `VirtualSMC.kext` + `SMCBatteryManager.kext`, plus `ECEnabler.kext`. |
| Sleep / wake | ✅ | Works; Bluetooth can need a nudge after wake (`sudo killall -9 bluetoothd`). |
| Trackpad | ✅ | Thanks to `SSDT-I2C-TPAD.aml` by [@Micael106](https://github.com/Micael106), with VoodooI2C in polling mode. |
| Touchscreen | ✅ | `VoodooI2C.kext` with `VoodooI2CHID.kext`, driven in polling mode by -vi2c-force-polling. |
| Keyboard | ✅ | Thanks to `VoodooPS2Controller.kext`. |
| Webcam | ✅ | |
| Wi-Fi/ Bluetooth | ✅ | `IOSkywalkFamily.kext` + `IO80211FamilyLegacy.kext` + `AirportBrcmFixup.kext`+ `AMFIPass.kext` for the Wi-Fi, and `BrcmPatchRAM3.kext` + `BrcmFirmwareData.kext` + `BlueToolFixup.kext` + `BlueWakeFixup.kext` for Bluetooth. Needs the **Modern Wi-Fi** root patch. Check Known Issues. |
| AirDrop / Continuity | ✅ | Comes with the restored Apple wireless stack. |
| HDMI output | ❌ | Ice Lake has no working external display path on macOS — no fix exists. |
| SD card reader | ❓ | Not tested. |

---

## Requirements

1. **macOS Tahoe 26.x.**
2. **[OCLP-CustoMac](https://github.com/kgp-macPro/OCLP-CustoMac)** — the fork. Apply both the
   **Modern Wi-Fi** and **Modern Audio** root patches after installing.
   Modern Audio needs a Kernel Debug Kit, which needs working networking — so get Wi-Fi (Or get an iPhone for network) up
   first, then tick Modern Audio and press *Start Root Patching*.
---

## Known issues

* **Wi-Fi/ Bluetooth** — sometimes it disconnects for a while, still useable since DW1820A kinda suck tbh, planning to switch to another one
* **HDMI** — no external display output on Ice Lake under macOS. Unfixed since 2021 in the Ice
  Lake hackintosh trackers. A USB-C DisplayLink adapter is the practical workaround.
* **Performance** — 8 GB of soldered RAM on a 15 W quad-core is the floor for Tahoe. Expect
  lag when multitasking. `Reduce transparency` and `Reduce motion` in Accessibility → Display
  are the free wins. Not thermal: `pmset -g therm` reports `CPU_Speed_Limit = 100`.

---

## Credits
* **[BestRazer](https://github.com/BestRazer/Lenovo-IdeaPad-Flex5-14IIL05-Hackintosh)** — IdeaPad Flex 5 14IIL05 OpenCore EFI; the working Ice Lake iGPU DeviceProperties and the component/kext breakdown this README is modelled on.
* **[RobyRew](https://github.com/RobyRew/Lenovo-Ideapad-Flex-5-14IIL05_Hackintosh_OpenCore)** — IdeaPad Flex 5 14IIL05 hackintosh guide; the audio layout, installation guide and the GenSMBIOS instructions.
* **[Micael106](https://github.com/Micael106/Lenovo-Ideapad-Flex-5-14IIL05)** — the trackpad fix, `SSDT-I2C-TPAD.aml`, which is the file used here.
* **[chundk](https://github.com/chundk/Hackintosh-4-Lenovo-Ideapad-Flex5-15IIL05-Guide-OpenCore-EFI)** — the Flex 5 15IIL05 guide and EFI for the same platform, source of the BIOS settings and kext-by-kext notes.
* **[OpenCore](https://github.com/acidanthera/OpenCorePkg)** and the Acidanthera kexts — Lilu, WhateverGreen, VirtualSMC, AppleALC, AirportBrcmFixup, BrcmPatchRAM, BlueToolFixup, RestrictEvents, NVMeFix, CpuTscSync, BrightnessKeys, VoodooPS2.
* **[OCLP-CustoMac](https://github.com/kgp-macPro/OCLP-CustoMac)** — the patcher fork that makes Modern Wi-Fi and Modern Audio work on Tahoe.
* **[VoodooI2C](https://github.com/VoodooI2C/VoodooI2C)** and **[USBToolBox](https://github.com/USBToolBox/kext)**.
* **[Mirone](https://github.com/Mirone)** — `BlueWakeFixup.kext`, the post-wake Bluetooth fix, and Wi-Fi Patcher Pro.
