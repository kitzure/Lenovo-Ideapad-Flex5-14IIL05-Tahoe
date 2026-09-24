# Configuration inventory

Generated automatically from `EFI/OC/config.plist` at build time.

## Boot arguments

```
-igfxdbeo igfxfw=2 -vi2c-force-polling watchdog=0 -igfxblr -noDC9 -revsbvmm alcid=11 -amfipassbeta brcmfx-aspm=0 -btlfxboardid -bwfxbeta brcmfx-country=#a
```

## ACPI

- `SSDT-AWAC.aml`
- `SSDT-PTSWAKTTS-iGPU.aml`
- `SSDT-I2C-XPD0.aml`
- `SSDT-I2C-XPL1.aml`
- `SSDT-GPRW.aml`
- `SSDT-ALS0.aml`
- `SSDT-I2C-TPAD.aml`
- `SSDT-EC-USBX-LAPTOP.aml`
- `SSDT-PLUG.aml`
- `SSDT-PNLF.aml`
- `SSDT-RHUB.aml`
- `SSDT-XOSI.aml`

## UEFI drivers

- `OpenRuntime.efi`
- `OpenCanopy.efi`
- `AudioDxe.efi` (optional)
- `ResetNvramEntry.efi`
- `HfsPlus.efi`

## Tools

- `OpenShell.efi`
- `ResetSystem.efi`
- `BootKicker.efi`
- `ChipTune.efi`
- `CleanNvram.efi`
- `ControlMsrE2.efi`
- `CsrUtil.efi`
- `GopStop.efi`
- `KeyTester.efi`
- `MmapDump.efi`
- `OpenControl.efi`
- `RtcRw.efi`
- `TpmInfo.efi`

## Device properties

### `PciRoot(0x0)/Pci(0x1F,0x3)`

- `layout-id` = `0b 00 00 00`

### `PciRoot(0x0)/Pci(0x1b,0x0)`

- `layout-id` = `0b 00 00 00`

### `PciRoot(0x0)/Pci(0x2,0x0)`

- `AAPL,ig-platform-id` = `01 00 5c 8a`
- `agdpmod` = `01 00 00 00`
- `device-id` = `5c 8a 00 00`
- `enable-backlight-registers-fix` = `01 00 00 00`
- `enable-cdclk-frequency-fix` = `01 00 00 00`
- `enable-dbuf-early-optimizer` = `01 00 00 00`
- `enable-dvmt-calc-fix` = `01 00 00 00`
- `framebuffer-fbmem` = `00 00 90 00`
- `framebuffer-patch-enable` = `01 00 00 00`
- `framebuffer-stolenmem` = `00 00 30 01`
- `framebuffer-unifiedmem` = `00 00 00 c0`

## Kernel quirks

- `AppleCpuPmCfgLock` = `False`
- `AppleXcpmCfgLock` = `True`
- `AppleXcpmExtraMsrs` = `False`
- `AppleXcpmForceBoost` = `False`
- `CustomPciSerialDevice` = `False`
- `CustomSMBIOSGuid` = `False`
- `DisableIoMapper` = `True`
- `DisableIoMapperMapping` = `False`
- `DisableLinkeditJettison` = `True`
- `DisableRtcChecksum` = `False`
- `ExtendBTFeatureFlags` = `False`
- `ExternalDiskIcons` = `False`
- `ForceAquantiaEthernet` = `False`
- `ForceSecureBootScheme` = `False`
- `IncreasePciBarSize` = `False`
- `LapicKernelPanic` = `False`
- `LegacyCommpage` = `False`
- `PanicNoKextDump` = `True`
- `PowerTimeoutKernelPanic` = `True`
- `ProvideCurrentCpuInfo` = `False`
- `SetApfsTrimTimeout` = `-1`
- `ThirdPartyDrives` = `False`
- `XhciPortLimit` = `False`

## Booter quirks

- `AllowRelocationBlock` = `False`
- `AvoidRuntimeDefrag` = `True`
- `ClearTaskSwitchBit` = `False`
- `DevirtualiseMmio` = `True`
- `DisableSingleUser` = `False`
- `DisableVariableWrite` = `False`
- `DiscardHibernateMap` = `False`
- `EnableSafeModeSlide` = `True`
- `EnableWriteUnprotector` = `True`
- `FixupAppleEfiImages` = `False`
- `ForceBooterSignature` = `False`
- `ForceExitBootServices` = `False`
- `ProtectMemoryRegions` = `False`
- `ProtectSecureBoot` = `False`
- `ProtectUefiServices` = `True`
- `ProvideCustomSlide` = `True`
- `ProvideMaxSlide` = `0`
- `RebuildAppleMemoryMap` = `True`
- `ResizeAppleGpuBars` = `-1`
- `SetupVirtualMap` = `True`
- `SignalAppleOS` = `False`
- `SyncRuntimePermissions` = `True`
