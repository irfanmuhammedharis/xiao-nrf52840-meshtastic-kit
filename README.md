# XIAO nRF52840 + Wio-SX1262 Meshtastic Kit

Working setup for the [Seeed XIAO nRF52840 & Wio-SX1262 Kit for Meshtastic](https://wiki.seeedstudio.com/xiao_nrf52840&_wio_SX1262_kit_for_meshtastic/).

## Firmware

Built from [meshtastic/firmware](https://github.com/meshtastic/firmware) at tag `v2.7.26.54e0d8d`, PlatformIO env `seeed_xiao_nrf52840_kit`:

- `firmware-seeed_xiao_nrf52840_kit-2.7.26.54e0d8d.uf2` — drag-and-drop flashing (UF2 bootloader mode)
- `firmware-seeed_xiao_nrf52840_kit-2.7.26.54e0d8d.zip` — serial DFU package for `adafruit-nrfutil`

The firmware source is not committed here; to rebuild:

```sh
git clone --depth 1 --branch v2.7.26.54e0d8d --recurse-submodules \
    https://github.com/meshtastic/firmware.git meshtastic-firmware
cd meshtastic-firmware
pio run -e seeed_xiao_nrf52840_kit
```

## Flashing

**UF2 (easiest):** double-tap the reset button next to USB-C → a `XIAO-SENSE` drive appears → copy the `.uf2` onto it. On Linux, prefer a sync mount (`mount -o sync`) — async desktop mounts can corrupt the transfer.

**Serial DFU:** put the board in DFU mode with a 1200-baud open/close on its serial port, then:

```sh
adafruit-nrfutil dfu serial -p /dev/ttyACM0 -b 115200 --singlebank \
    -pkg firmware-seeed_xiao_nrf52840_kit-2.7.26.54e0d8d.zip
```

Do **not** use `pio run -t upload` — its automatic 1200-baud touch reboots the device before the DFU tool runs.

## Hard-won notes

- Factory boards ship with UF2 bootloader **0.6.1 (2021), on which firmware UF2s silently fail to apply**. Update the bootloader first by flashing
  [`update-xiao_nrf52840_ble_sense_bootloader-0.11.0_nosd.uf2`](https://github.com/adafruit/Adafruit_nRF52_Bootloader/releases) in UF2 mode.
- USB identities are confusing: the **running Meshtastic app** enumerates as `239a:810b "XIAO-BOOT"` (yes, really), while the **bootloader** is `2886:0045 "XIAO nRF52840 Sense"` (with a `XIAO-SENSE` drive in UF2 mode, serial-only in DFU mode).
- After boot, the node spends ~20 s probing for a GPS the kit doesn't have; the serial API answers only after that, so wait ~40 s before deciding it's dead.
- Stale device prefs from older firmware can leave the serial API and Bluetooth disabled (node looks bricked while running fine). Fix: factory reset / wipe `/prefs`.
- Bluetooth pairing PIN: `123456`. If pairing fails, forget the old device entry in the phone's Bluetooth settings first.

## Status / known issue

Intermittent early-boot hang: occasionally a boot comes up with no serial output, no API, and no BLE advertising (same binary boots fine other times). Workaround: press reset once and wait ~40 s.
