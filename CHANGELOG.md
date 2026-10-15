# Changelog <!-- omit in toc -->

- [Unreleased](#unreleased)
  - [Added](#added)
  - [Fixed](#fixed)
  - [Upgrade Notes](#upgrade-notes)
  - [Validation](#validation)

## Unreleased

### Added

- **USB setup:** [`tools/usb-setup`](tools/usb-setup/README.md) writes the Wi-Fi and MQTT settings over USB,
  skipping the `Aeris-XXXX` hotspot and the settings page. It can also install a newer build without
  resetting the settings.

### Fixed

- **MQTT identity:** each unit connects as `Aeris-<device ID>` instead of the shared `Aeris` client ID, so
  several purifiers can share one broker without disconnecting each other.
- **Ring and button lights:** the 74HC595 shift register behind the ring and button lights (`D5` data, `D6` clock,
  `D7` latch, `A4` boot handshake) is now driven, so the ring switches with the display instead of keeping
  its random power-up state.

### Upgrade Notes

- MQTT topics, payloads and web settings are unchanged.
- A newly compiled build still resets the settings on its first boot, unless it is installed with
  `tools/usb-setup`.

### Validation

- Three purifiers published on one broker at the same time: two with unique client IDs, one still on the
  shared `Aeris` ID.
- USB setup on a purifier with Device OS 3.3.1 (macOS, Node.js): `setup` for a newly compiled build booted it
  straight onto Wi-Fi and MQTT without the hotspot; `upgrade` to another new build kept every setting;
  `check` accepts the release `aerisFirmware.bin`.
- Ring: on a unit whose ring stayed dark, it now lights with the display and goes dark when the display
  is switched off over MQTT.
