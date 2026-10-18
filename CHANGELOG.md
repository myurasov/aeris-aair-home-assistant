# Changelog <!-- omit in toc -->

Versions follow [semantic versioning](https://semver.org), continuing the upstream releases: release v4 is
4.0.0 and release v5 is 5.0.0. Builds of unreleased changes report the next version with `-dev`.

- [Unreleased (6.0.0)](#unreleased-600)
  - [Added](#added)
  - [Fixed](#fixed)
  - [Upgrade Notes](#upgrade-notes)
  - [Validation](#validation)
- [5.0.0 - 2026-10-16](#500---2026-10-16)
- [4.0.0 - 2026-10-15](#400---2026-10-15)

## Unreleased (6.0.0)

Builds report `6.0.0-dev`.

### Added

- **Fan modes:** A switches between manual, auto-low and auto-high; an auto mode shows under the % sign. The
  auto modes set the speed on the unit from PM2.5 (auto-low 20 % up to 5 µg/m³, then +1.6 % per µg/m³ to
  100 % at 55; auto-high 1.25 times that), so no Home Assistant automation is needed. `fan/mode/set` and the
  retained `fan/mode/state` (also as Home Assistant fan presets); any manual speed change returns to manual.
- **Power button:** a short press switches the screen on or off, holding it for a second switches the unit
  on or off; `fan/power/set` (`ON` / `OFF`) does the same and keeps the mode. Off also darkens the ring and
  button lights, whatever their follow setting; their settings return when the unit turns on.
- **Version:** the firmware reports its version on the web page, retained on `version`, and on the screen
  while A is held for two seconds.
- **Fan speed step:** 1 % by default, 1-25 % in the web page's Fan section or over `fan/step/set`
  (retained `fan/step/state`); saved on the unit. The + and - buttons repeat while held.
- **Fan resume:** the unit saves its fan speed (or off) a few seconds after a change and resumes it after a
  reboot or power cut. The power button resumes the last speed exactly (it used to raise anything below 10 %).
- **- to off:** holding - stops at 20 %; a separate press of - then stops the fan but keeps the screen on
  (showing OFF).
- **Web page:** a REBOOT button restarts the unit with all settings kept.

### Fixed

- **Fan speeds:** 5 % left the fan standing still, and the steps were very uneven (5 -> 10 % went from off
  to about a fifth of full speed, 50 -> 100 % added only a quarter more). Speeds now follow a measured speed
  curve, and the percentage is the estimated share of maximum airflow: 20 % is the slowest steady speed,
  100 % full speed, and lower non-zero settings run at 20 %. A start from standstill gets half a second of
  extra drive.
- **Buttons:** the panel's button lines are active-low, but they were read as active-high: every action ran
  on release instead of on press, and at startup each button counted as pressed once, which switched the fan
  off after every boot. They are now read as active-low with pull-ups.
- **Display on:** switching the display on showed the screen being wiped from top to bottom; it is now
  redrawn before the backlight comes on.
- **Web page reset:** FACTORY RESET erased the Wi-Fi and MQTT settings on one click; it and ENTER DFU MODE now
  ask for confirmation.
- **Button lights:** full brightness lit only each button's own LED line. The top of the brightness range now
  also lights the line shared by all four buttons, about 1.5 times as bright at full; it adds light only while
  all four are lit. The dimmest level is unchanged.

### Upgrade Notes

- Fan percentages now mean estimated airflow (the old 10 % runs like the new 20 %, the old 50 % like about
  77 %, the old 75 % like about 90 %); review automations that set fixed percentages. `fan/set` and
  `fan/state` keep their 0-255 payload; the README's Home Assistant fan example shows the unit's exact
  percentage in 1 % steps.
- A no longer switches the screen (it changes the mode); a short press of power does, and switching the
  unit off needs power held for a second. The README's Home Assistant fan example uses `fan/power/set` for
  on / off and offers the auto modes as presets; its smart-control automation is replaced by the auto modes.
- New MQTT topics: `fan/power/set`, `fan/mode/set` and `fan/mode/state`, `fan/step/set` and
  `fan/step/state`, `version`. Existing topics keep their payloads.
- The fan settings (step, mode and the state to resume) live at EEPROM address 1088, apart from the other
  settings. The first boot after updating starts in manual at 25 %.

### Validation

- Buttons: measured on a unit (all four lines rest high and go low while pressed; the panel reports one key
  at a time). Every button action, the modes, unit off and resuming across reboots checked on the unit, over
  MQTT and in Home Assistant.
- Fan speed: measured with a microphone (fan-tone pitch and loudness) across every PWM value: start, stop and
  the speed curve; the new steps checked for even airflow and starts from standstill.
- Web page: REBOOT, sent the way a browser sends it, restarted the unit with every setting kept.

## 5.0.0 - 2026-10-16

### Fixed

- **PM readings:** PM2.5 and PM10 come from the sensor's mass fields (µg/m³). Before, they came from its
  particle-count channels, which read many times higher and jumped with airflow.

### Upgrade Notes

- PM2.5 and PM10 readings drop to real µg/m³ values; review automation thresholds tuned to the old numbers.

## 4.0.0 - 2026-10-15

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

### Upgrade Notes

- Existing MQTT topics, payloads and web settings are unchanged.
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
