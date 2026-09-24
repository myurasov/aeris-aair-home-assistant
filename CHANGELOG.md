# Changelog <!-- omit in toc -->

- [Upstream Reference](#upstream-reference)
- [Added](#added)
- [Fixed](#fixed)
- [Upgrade Notes](#upgrade-notes)
- [Validation](#validation)

## Upstream Reference

Current changes in [myurasov's personal fork](https://github.com/myurasov/aeris-aair-home-assistant) relative to [mjaymeyer/aeris-aair-home-assistant at `3420b51`](https://github.com/mjaymeyer/aeris-aair-home-assistant/tree/3420b51), including the initial PM1.0 addition in `83c3432` and subsequent fixes.

## Added

- **PM1.0:** MQTT topic `<prefix>sensor/pm1`, 10-sample smoothing, five-second reporting, and a Home Assistant sensor YAML example.
- **Fork guidance:** source-build instructions pinned to Device OS 3.3.1 and sensor-protocol research links in the [README](README.md).

## Fixed

- **PM decoding:** mass fields use zero-based offsets 4/6/8 in 32-byte `32 3D` frames. Upstream PM2.5/PM10 used particle-count offsets 12/14; the initial fork PM1 implementation used offset 10.
- **MQTT identity:** `"Aeris-" + System.deviceID()` replaces the shared `Aeris` client ID, preventing collisions between units on one broker.
- **Display startup workaround:** 500 ms power settling, software reset at 10 MHz, 150 ms wait, then full reinitialization. With `TFT_RST = -1`, the pinned `Adafruit_ILI9341` 1.0.3 library's `begin()` performs no reset. Controlled cold-start coverage remains limited.
- **Smart-control example:** current-state checks on PM updates, fan-on, mode-enable, Home Assistant start and minute ticks. The speed curve uses 5% steps and a one-percentage-point tolerance. Manual OFF sticks; disabling smart mode preserves manual speed.

## Upgrade Notes

- Build this fork's source; upstream release binaries do not contain these changes.
- Newly compiled builds reset settings on first boot. Repeat setup and use a distinct topic prefix per purifier.
- Corrected PM readings may be lower. Review air-quality thresholds and comparisons with historical readings.

## Validation

- Sensor mapping checked against 40 valid raw frames and corroborated by independent Yishan-format decoders linked in the README; the exact Aeris sensor model remains unconfirmed.
- MQTT control and live Home Assistant integration verified: PM1 readings, fan, display switch, smart mode and dashboard. Smart-mode tests covered enabling with unchanged PM, fan-on, persistent manual OFF, and disabled mode preserving manual speed across a minute tick.
- Display workaround compiled and flashed; the tester reported that the display worked after retesting. **Prolonged power-off and repeated controlled cold-start testing have not been recorded.**
