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
- **Ring and button light control:** the ring and the button lights are two lights with brightness,
  effects (`solid`, `blink`, `breathe`, `pulse`, `candle`; buttons also `chase`, `bounce`, `fill`, `wave`,
  `twinkle`), effect speed and follow-the-display, set over MQTT JSON (`ring/set`, `buttons/set`, retained
  `.../state`), from Home Assistant or the web page, and saved on the unit. Dimming is 128-step PWM at
  156 Hz from a hardware timer.

### Fixed

- **MQTT identity:** each unit connects as `Aeris-<device ID>` instead of the shared `Aeris` client ID, so
  several purifiers can share one broker without disconnecting each other.
- **Ring and button lights:** the 74HC595 shift register behind the ring and button lights (`D5` data, `D6` clock,
  `D7` latch, `A4` boot handshake) is now driven, so the ring switches with the display instead of keeping
  its random power-up state.
- **Web page form submits:** the web server now waits up to a second for the request line. Before, a
  browser request that arrived just after the connection got the plain page back and the form (Save or
  Lights) was silently ignored.
- **PM readings:** PM2.5 and PM10 come from the sensor's mass fields (µg/m³). Before, they came from its
  particle-count channels, which read many times higher and jumped with airflow.
- **Fan speeds:** 5 % left the fan standing still, and the steps were very uneven (5 -> 10 % went from off
  to about a fifth of full speed, 50 -> 100 % added only a quarter more). The 20 steps now follow a measured
  speed curve: 5 % is the slowest steady speed, and each step adds about the same airflow. A start from
  standstill gets half a second of extra drive.

### Upgrade Notes

- MQTT topics, payloads and web settings are unchanged.
- PM2.5 and PM10 readings drop to real µg/m³ values; review automation thresholds tuned to the old numbers.
- The same fan percentage now runs at a different speed (for example 50 % is about 60 % of full speed
  instead of about 77 %); review automations that set fixed percentages. The README's Home Assistant fan
  example now reports the unit's exact percentage in 5 % steps.
- A newly compiled build still resets the settings on its first boot, unless it is installed with
  `tools/usb-setup`.
- Builds need the SparkIntervalTimer 1.3.8 library, declared in `Source/project.properties` (the Particle
  cloud compiler fetches it). The light settings live at EEPROM address 1024, apart from the other settings.

### Validation

- Three purifiers published on one broker at the same time: two with unique client IDs, one still on the
  shared `Aeris` ID.
- USB setup on a purifier with Device OS 3.3.1 (macOS, Node.js): `setup` for a newly compiled build booted it
  straight onto Wi-Fi and MQTT without the hotspot; `upgrade` to another new build kept every setting;
  `check` accepts the release `aerisFirmware.bin`.
- Ring: on a unit whose ring stayed dark, it now lights with the display and goes dark when the display
  is switched off over MQTT.
- Light control on a unit: every effect, speed, the dimmest level, the web form and saving across a
  reboot; Home Assistant lights, speed sliders and switches round-trip; Wi-Fi and MQTT unaffected.
- Fan speed: measured with a microphone (fan-tone pitch and loudness) across every PWM value: start,
  stop and the speed curve; the new steps checked for even airflow and starts from standstill.
