# OpenCore EFI: ThinkPad T14s Gen 2 (Tiger Lake) on macOS Tahoe

An OpenCore EFI for the Lenovo ThinkPad T14s Gen 2 (Intel Core i5, 11th gen Tiger Lake, Iris Xe) running **macOS Tahoe 26.7.2**, with working graphics acceleration.

> **Unofficial and experimental.** Use it at your own risk. Back up your data first. Do not copy this EFI blindly: it contains no SMBIOS. Generate your own before booting (see [Before you boot](#before-you-boot)).

## Hardware

| Component | Detail |
| :--- | :--- |
| Laptop | Lenovo ThinkPad T14s Gen 2 |
| CPU | Intel Core i5, 11th gen (Tiger Lake) |
| GPU | Intel Iris Xe (`8086:9a49`) |
| Display | 1920x1080 internal |
| Wi-Fi | Intel (works with AirportItlwm) |
| Trackpad | I2C |

## Versions

| Item | Version |
| :--- | :--- |
| macOS | Tahoe 26.7.2 |
| OpenCore | 1.0.8 |
| SMBIOS model | `MacBookPro16,2` |
| Lilu | 1.7.2 |
| WhateverXe | 0.7.0 (patched, see [Graphics](#graphics-acceleration)) |
| AMFIPass | 1.4.1 |

## What works

| Feature | Status |
| :--- | :---: |
| Graphics acceleration (Metal) | ✅ Working |
| Keyboard | ✅ Working |
| Trackpad / mouse | ✅ Working |
| Audio (speakers) | ✅ Working |
| Brightness slider | ✅ Working |
| Battery | ✅ Working |
| Wi-Fi (AirportItlwm) | ✅ Working |
| Bluetooth | ❌ Not working |
| Sleep / wake | ❌ Not working |
| Microphone | ❌ Not working |
| Camera | ❌ Not working |

## Graphics acceleration

Graphics acceleration uses **WhateverXe** together with the **WhateverXe TGL 6-bank patch**.

**The patch is required.** Without it, WhateverXe does not support the 80 EU Tiger Lake i5 (80 EU with 6 L3 banks) on Tahoe or on Sequoia. Stock WhateverXe 0.7.0 rejects that layout, so the accelerator never starts: the display works, but there is no Metal and no blur. The patch relaxes that one check.

**Credit:** the patch was written by [liuzishuo702-sys](https://github.com/liuzishuo702-sys). Thank you for it. It was reported working on Sequoia. In this EFI it also works on Tahoe 26.7.2. Upstream discussion: [b00t0x/WhateverXe#10](https://github.com/b00t0x/WhateverXe/issues/10).

The `WhateverXe.kext` in this EFI is already built with the patch applied (WhateverXe 0.7.0 source). If you want to rebuild it, follow the patch repository's instructions and read its source first.

Symptom to look for if you hit the same problem:

```sh
ioreg -lw0 -d1 -r -n IGPU | grep TGLGT2Qualified
# "TGLGT2Qualified" = No   -> the gate is rejecting your GPU
# "TGLGT2Qualified" = Yes  -> accelerator can start
```

You need two WhateverXe packages installed in macOS: `org.b00t0x.WhateverXe.Common` and `.Tahoe`. They provide the graphics kexts. The EFI only injects the patched `WhateverXe.kext`.

The patched kext is built from the official source plus the one-line patch. Read the patch source before you build it yourself.

## Required downloads and credits

This EFI is only possible because of the projects below. Please support them and read their documentation.

| Project | Used for | Link |
| :--- | :--- | :--- |
| OpenCore | Bootloader | https://github.com/acidanthera/OpenCorePkg |
| Lilu | Kext patching engine | https://github.com/acidanthera/Lilu |
| WhateverXe | Iris Xe graphics acceleration | https://github.com/b00t0x/WhateverXe |
| WhateverXe-Graphics | Graphics packages for macOS | https://github.com/b00t0x/WhateverXe-Graphics |
| WhateverXe TGL 6-bank patch, by **liuzishuo702-sys** | Makes WhateverXe accept 80 EU / 6 L3 bank Tiger Lake parts (see [Graphics](#graphics-acceleration)) | https://github.com/liuzishuo702-sys/wex-tgl-6bank-patch |
| Upstream issue for the patch | Discussion and measurements | https://github.com/b00t0x/WhateverXe/issues/10 |
| itlwm / AirportItlwm (upstream) | Intel Wi-Fi | https://github.com/OpenIntelWireless/itlwm |
| AirportItlwm (included in this EFI), Tahoe build by laobamac | Wi-Fi on Tahoe, built for Tahoe with no root patches needed (v2.4.0 alpha) | https://github.com/laobamac/itlwm |
| SimpleHDA | Restores AppleHDA for audio on Tahoe | https://github.com/laobamac/SimpleHDA |
| AppleALC | Audio codec support | https://github.com/acidanthera/AppleALC |
| VoodooI2C | I2C trackpad | https://github.com/VoodooI2C/VoodooI2C |
| VoodooPS2 | Keyboard | https://github.com/acidanthera/VoodooPS2 |
| BrightnessKeys | Brightness keys | https://github.com/acidanthera/BrightnessKeys |
| ECEnabler | Battery readout | https://github.com/1Revenger1/ECEnabler |
| NVMeFix | NVMe power management | https://github.com/acidanthera/NVMeFix |
| USBToolBox | USB mapping | https://github.com/USBToolBox |

TODO: add links for any other kext you use (for example IntelMausi, TGLDmic, CPUFriend) and verify each URL before publishing.

## Before you boot

This repository leaves out two things on purpose.

### 1. SMBIOS (you must generate your own)

`config.plist` contains placeholders for `SystemSerialNumber`, `MLB`, `SystemUUID` and `ROM`. Generate your own with `macserial` from OpenCore:

```sh
./macserial -m MacBookPro16,2 -n 10
uuidgen
```

Pick a serial that is **not** a real registered Mac. A message that the serial is invalid on Apple's coverage page (`checkcoverage.apple.com`) is the result you want. Never use someone else's serial. Do not publish your own.

## Boot arguments

```
-allow3d -disablegfxfirmware -igfxdvmt -vi2c-force-polling -tgldmic shda-lang=en
```

| Argument | Purpose |
| :--- | :--- |
| `-allow3d` | WhateverXe: allow 3D acceleration |
| `-disablegfxfirmware` | Disable graphics firmware loading |
| `-igfxdvmt` | Work around DVMT pre-allocation limits |
| `-vi2c-force-polling` | Force polling mode for the I2C trackpad |
| `-tgldmic` | Enable the TGLDmic microphone kext (microphone still not working) |
| `shda-lang=en` | SimpleHDA: English audio device names |

## Known issues and notes

- **Sleep / wake does not work.** Avoid sleep. Closing the lid may require a hard restart.
- **Microphone, camera and Bluetooth do not work.**
- **Do not use "Reset NVRAM" from the OpenCore picker.** On this laptop it re-creates a "Windows Boot Manager" entry that can block OpenCore.
- **Keep `VoodooPS2Mouse` enabled.** Disabling it disables the keyboard on this model, even though the trackpad is I2C.
- **Brightness:** the ACPI `SSDT-PNLF` uses the device path `\_SB.PC00.GFX0`, because this laptop's root is `PC00`, not `PCI0`.
- WhateverXe is experimental. The patch has not been validated for long GPU loads, so watch for glitches or hangs.

## Installation outline

1. Make a macOS Tahoe installer or recovery media.
2. Generate your SMBIOS, as described above.
3. Copy `EFI` to the EFI partition of your boot disk.
4. Install macOS, then install the two WhateverXe packages and reboot.
5. Verify graphics:
   ```sh
   system_profiler SPDisplaysDataType | grep -i metal
   ```

## Disclaimer

This is provided as is, with no warranty. It is not affiliated with Apple, Lenovo, Intel or any project listed above. macOS is a trademark of Apple Inc. Check the license of every component before you redistribute it.
