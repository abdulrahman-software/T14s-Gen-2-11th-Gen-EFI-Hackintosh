# Lenovo ThinkPad T14s Gen 2 OpenCore EFI

Hardware-specific OpenCore EFI for the Lenovo ThinkPad T14s Gen 2. Big Sur is the tested macOS version.

## Working on Big Sur

The following have been tested and are working:

- Internal keyboard
- Trackpad
- TrackPoint
- Wi-Fi through AirportItlwm
- Brightness slider
- Bluetooth

## Known issues

- Sleep and wake do not work reliably.
- The App Store has glitches.

## Other macOS versions

Sequoia and Tahoe have not been tested with this EFI. Testing and improvements for both versions are especially welcome.

## Important: PS/2 mouse kext

Do not disable `VoodooPS2Mouse.kext`. On this laptop, the PS/2 TrackPoint driver is required for the internal keyboard to work. Disabling the mouse kext also disables the keyboard.

## Graphics credit

Intel graphics support uses [WhateverXe](https://github.com/b00t0x/WhateverXe), a Lilu plugin that adapts Apple graphics drivers for supported Intel Xe-LP GPUs. Credit and thanks to the WhateverXe contributors for their work.

## Contributions

Pull requests with fixes and improvements are welcome, especially for sleep/wake, App Store glitches, and Sequoia or Tahoe support. Please include the macOS version and the changes and tests you made.

## Before use

This public copy contains sanitized SMBIOS placeholders. Generate your own values and replace `PlatformInfo -> Generic -> MLB`, `ROM`, `SystemSerialNumber`, and `SystemUUID` in `OC/config.plist` before booting. Never publish or reuse another person's SMBIOS values.
