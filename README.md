# Replacement Firmware for the Aeris Aair 3-in-1 Pro Air Purifier <!-- omit in toc -->

- [Disclaimer](#disclaimer)
- [Software Needed:](#software-needed)
- [Teardown/Accessing Board](#teardownaccessing-board)
- [Flashing Replacement Software](#flashing-replacement-software)
  - [Initial Connection, Driver Install](#initial-connection-driver-install)
  - [Taking a Backup](#taking-a-backup)
  - [Display Startup](#display-startup)
- [Setup Process](#setup-process)
  - [Stage One (WiFi Network Connecting)](#stage-one-wifi-network-connecting)
  - [Stage Two (MQTT and Display Customization)](#stage-two-mqtt-and-display-customization)
  - [Home Assistant](#home-assistant)
    - [Defining Sensors and Fan](#defining-sensors-and-fan)
    - [Automation YAML](#automation-yaml)
- [Restoring Original Firmware](#restoring-original-firmware)

This guide will walk you through flashing a new firmware onto the Aeris Aair 3-in-1 Pro Air Purifier, which will enable local control of the device over MQTT, which can then be added and controlled via Home Assistant. It also allows some small customization of the appearance of the top display.

This is [myurasov's personal fork](https://github.com/myurasov/aeris-aair-home-assistant) of [mjaymeyer's firmware](https://github.com/mjaymeyer/aeris-aair-home-assistant). It adds PM1.0 reporting, corrects particulate mass decoding, gives each purifier a unique MQTT client ID, and adds a display-startup workaround. See the [changelog](CHANGELOG.md) for the full differences from upstream and validation status. **Build from this fork's source to use these changes**; the upstream release binaries linked below do not include them.

<table>
  <tr>
    <td valign="middle">
      <img src="./media/animation.gif" alt="Animation" height="250"/>
    </td>
    <td valign="middle">
      <img src="./media/31.png" alt="Device" width="300"/>
    </td>
  </tr>
</table>

<br>
<br>

## Disclaimer

The steps below are not complicated to do but make sure you read through everything before committing. In terms of feature loss, there is no built in logic to adjust fan speed based on air quality. Since the device can be controlled over MQTT now that logic can be done via Home Assistant. I have included an example automation you can add to Home Assistant to do just this.

<br>
<br>

## Software Needed:

The guide below and the software linked to here are for flashing from Windows. If using Mac or Linux then after getting access to the board navigate down to Mac / Linux section.

1.  [Zadig](<https://zadig.akeo.ie/>)

2.  [dfu-util v0.9](<http://dfu-util.sourceforge.net/releases/dfu-util-0.9-win64.zip>)

3.  [Particle CLI](<https://binaries.particle.io/cli/installer/windows/ParticleCLISetup.exe>)

4.  [Free Particle CLI Account](<https://login.particle.io/signup>)

5.  [Particle Bootloader update](<https://github.com/particle-iot/device-os/releases/download/v3.3.1/photon-bootloader%403.3.1%2Blto.bin>) — the v3 firmware needs Device OS **3.3.1 or newer** (system module version 3302; 3.3.0 is 3301 and is not enough).

6.  [Upstream Replacement Software Binary](https://github.com/mjaymeyer/aeris-aair-home-assistant/releases) (upstream build; for this fork's changes, [build from source](#building-this-fork-from-source))

<br>
<br>

## Teardown/Accessing Board

1.  Unplug unit, remove top cover and flip over

<img src="./media/image1.jpeg" alt="" width="600"/>

3.  Remove the eight short screws, carefully lift out the inner-part of
    the case bottom to top to not damage the wires upper-left

<img src="./media/image2.jpeg" alt="" width="600"/>

3.  Remove the four silver screws, again carefully lift the outer
    bracket bottom to top to not damage the wires

<img src="./media/image3.jpeg" alt="" width="600"/>

4.  Remove the 12 slightly longer screws. Carefully pull the plastic
    cover bottom to top to reveal the main board.

<img src="./media/image4.jpeg" alt="" width="600"/>

5.  Carefully pull off the wire connector to separate the main board
    from the body entirely. No need to unscrew the board from the white
    plastic backing.

<img src="./media/image5.jpeg" alt="" width="600"/>

6.  For the flashing process we will use the two buttons 'RESET' and
    'SETUP' that should be pretty easy to work with.

<img src="./media/image6.jpeg" alt="" width="600"/>

## Flashing Replacement Software

The below shows the steps to flash from a Windows machine. If you're
using Mac or Linux it shouldn't be too hard to obtain the relevant
versions and follow along.

### Initial Connection, Driver Install

1.  Extract dfu-util to a folder. I've used C:\\DFU (no subfolders, you
    should be able to execute C:\\DFU\\dfu-util.exe)

2.  Attach the micro-usb cable to the unit, two LEDs should come on

<img src="./media/image7.jpeg" alt="" width="600"/>

3.  Put the device in DFU mode by pressing both the RESET
    and SETUP buttons. Immediately let go of RESET but continue holding
    SETUP for 3 seconds. LED59 (to the right of the
    setup button) should start flashing yellow, you are in DFU mode and
    you can let go of the SETUP button.

4.  Launch Zadig

<img src="./media/image8.png" alt="" width="400"/>

5.  Go to Options and check 'List All Devices'

<img src="./media/image9.png" alt="" width="400"/>

6.  Select Photon DFU Mode in the dropdown (if you do not see it try
    unplugging the micro-usb cable, waiting a few, then plugging it back
    in and putting the device back in DFU mode again).

7.  Note the USB ID listed (2B04:D006). We will reference this in later
    steps so take note if yours is different and adjust accordingly.

<img src="./media/image10.png" alt="" width="200"/>

8.  Select libusbK (v3.1.0.0) on the right using the down arrows (note
    that I already installed the driver on my device so it matches the
    driver listed on the left, yours likely will not). Click 'Replace
    Driver' and wait for it to install. Once installed you can close
    Zadig.

<img src="./media/image11.png" alt="" width="400"/>

<img src="./media/image12.png" alt="" width="400"/>

### Taking a Backup

First we'll make a backup of the original software.

1.  Open an admin-level command prompt and navigate to the folder you
    extracted dfu-utils to.

<img src="./media/image13.png" alt="" width="600"/>

2.  Execute the following command to backup the firmware, adjusting the
    device ID if necessary.

```dfu-util.exe -d 2b04:d006 -a 0 -s 0x08020000:0xE0000 -U original_firmware.bin```
>
<img src="./media/image14.png" alt="" width="600"/>

3.  This should export original_firmware.bin (despite it saying Upload)
    to the same directory you put dfu-util in. Backup the .bin to a safe
    place.

### Updating the Bootloader and Base Particle System

You will likely need to update the bootloader of the device to flash the
new software to it. It's possible there are some units out there with a
later bootloader but given the software support of Aeris, I doubt it. I
don't recall ever receiving a firmware update on my device from the
manufacturer.

#### Updating Bootloader

1.  Download the bootloader if you have not already, rename it
    to bootloader.bin to make it easier. I placed it in the same DFU
    folder to keep things simple.

2.  Install Particle CLI if you have not already. Test that you can
    execute it from command prompt (you may have to open a new Window if
    you just installed it)

```particle version```
>
<img src="./media/image15.png" alt="" width="600"/>

3.  Run the below command to flash the bootloader

```particle flash --local C:\DFU\bootloader.bin```
>
<img src="./media/image16.png" alt="" width="600"/>

<img src="./media/image17.png" alt="" width="600"/>

#### Updating the Particle software

1.  To get the latest system version from particle we need to login.
    Enter the below command followed by your email and password you
    setup when you created an account on particle.io

```particle login```

<img src="./media/image18.png" alt="" width="600"/>

2.  To update the bootloader and base system simply enter the below.

```particle update```

3.  Check the result with `particle serial inspect` (device in normal mode, not DFU). The v3 firmware depends on **system part 2 version 3302 or newer (Device OS 3.3.1+)**. My unit (2020 production) shipped with Device OS 0.5.0 (version 21); later production runs may be newer. Device OS 3.3.0 reports 3301, which is *not* enough — the app will not start (status LED blinks blue, no display, no `Aeris-xxxx` network).

<img src="./media/image19.png" alt="" width="600"/>

<img src="./media/image20a.png" alt="" width="600"/>

NOTE: Your device may seem to freeze (like below) when it gets to "Switching device to normal mode", if it says it fails after this do not panic, it is likely fine. You likely don't need to update again but are welcome to put your device back in DFU mode and try it again if you'd like, though I don't believe anything else needs updated. My device would seemingly fail after the switch back to normal mode but the software was definitely good to go regardless.

<img src="./media/image21a.png" alt="" width="600"/>

### Mac / Linux: offline flashing (no Particle account)

Tested on macOS with dfu-util 0.11 and Particle CLI 3.50 (`npm install particle-cli`). No Zadig or driver step is needed.
Do **not** just run `particle flash --local aerisFirmware.bin` on a stock unit: on an early unit still running Device OS 0.5.0
(mine is from 2020) it writes only the application and leaves the system firmware untouched, so the unit ends up with a blue blinking LED, a dark display and
no `Aeris-xxxx` network. Write the Device OS system parts explicitly instead.

1.  Download from the [Device OS 3.3.1 release](https://github.com/particle-iot/device-os/releases/tag/v3.3.1):
    `photon-system-part1@3.3.1.bin`, `photon-system-part2@3.3.1.bin`, `photon-bootloader@3.3.1+lto.bin`.

2.  Put the unit in DFU mode (blinking yellow) and take the backup as described above:

```dfu-util -d 2b04:d006 -a 0 -s 0x08020000:0xE0000 -U original_firmware.bin```

3.  Still in DFU mode, write the two system parts and then the application (`:leave` reboots the unit):

```
dfu-util -d 2b04:d006 -a 0 -s 0x08020000 -D photon-system-part1@3.3.1.bin
dfu-util -d 2b04:d006 -a 0 -s 0x08060000 -D photon-system-part2@3.3.1.bin
dfu-util -d 2b04:d006 -a 0 -s 0x080A0000:leave -D aerisFirmware.bin
```

4.  Once the unit is back in normal mode (`particle usb list` shows it as `Photon`), update the bootloader:

```particle flash --local photon-bootloader@3.3.1+lto.bin```

5.  Verify: `particle serial inspect` should show bootloader 1100, system parts 3302 / 3302 and user part 6.
    The display then shows the setup screen described below.

## Flashing the compiled binary

1.  Use the `aerisFirmware.bin` you [built from this fork](#building-this-fork-from-source),
    or download an upstream release to use mjaymeyer's build. Place it in the same folder as dfu-util. Put your
    device back into DFU mode if it is not.

2.  In CMD make sure you are back in the same folder where you have
    dfu-util.

<img src="./media/image22.png" alt="" width="600"/>

3.  Simply run the below command to flash the new software, it should
    install very quickly.

```dfu-util.exe -d 2b04:d006 -a 0 -s 0x080A0000:leave -D aerisFirmware.bin```
>
<img src="./media/image23.png" alt="" width="600"/>

4.  Assuming it flashed if you look on the front panel you should see
    the following setup screen

<img src="./media/image24.jpeg" alt="" width="600"/>

At this point you can unplug the USB and put back together your air
purifier, though you can run through the below first if you'd like.

### Display Startup

This fork addresses a blank or white panel on cold startup by holding the backlight off for a 500 ms power-settle delay, initializing the display, sending `ILI9341_SWRESET` at 10 MHz, waiting 150 ms, then running the full display initialization again. The display has no wired reset pin (`TFT_RST = -1`), and the pinned `Adafruit_ILI9341` 1.0.3 library does not issue a software reset in `begin()`.

The workaround has been compiled and flashed, and the tester reported a working display after retesting. **Controlled cold-start coverage remains limited**; prolonged power-off testing has not been recorded. See also the [upstream blank-display report](https://github.com/mjaymeyer/aeris-aair-home-assistant/issues/2).

## Setup Process

It is highly recommended to set the device up from a laptop as it is
much easier than fighting with a phone.

### Stage One (WiFi Network Connecting)

1.  An unencrypted WiFi network named Aeris-xxxx should be broadcasting.
    Connect to it and in a web browser navigate to <http://192.168.0.1>

2.  Enter your WiFi network credentials and click CONNECT, the device
    should restart.

<img src="./media/image25.png" alt="" width="600"/>

3.  Assuming you entered valid credentials you should see the unit
    display OFF and the buttons should function.
<img src="./media/image26.jpeg" alt="" width="800"/>

### Stage Two (MQTT and Display Customization)

1.  In a web browser now navigate to the true IP of the air purifier on
    your network (such as http://192.168.0.32). If it doesn't load confirm that you are hitting the right device and that you are not using https://, you should get the below page.

<img src="./media/image27.png" alt="" width="600"/>

2.  The display settings are already configured though you can of course
    adjust them to achieve a different look. In the top MQTT section
    enter your MQTT broker IP (e.g., 192.168.0.100), the port if
    different, and your username and password. You can change the
    broadcast MQTT topic prefix if you like (such as if you have
    multiple Aeris Purifiers and you need them unique).

3.  Hit save and the unit will reboot once more and you should see
    any display changes if you made them.

4.  Note that you can enter your purifier back into DFU Mode over this interface. This way you can flash a new firmware to it without having to unscrew the top to get to the board again.

Each purifier uses an MQTT client ID derived from its Particle device ID (`Aeris-<device-id>`),
so multiple units can share a broker. Configure a separate topic prefix for each unit as well.

### Home Assistant

PM1.0, PM2.5 and PM10 are 16-bit big-endian mass concentrations at zero-based offsets 4, 6 and 8 (bytes 4–9) of the sensor's 32-byte `32 3D` frame. Earlier builds mistook particle-count fields at offsets 10/12/14 for mass concentrations, so readings may decrease after upgrading; review existing thresholds and historical comparisons. PM1.0 is published at `<prefix>sensor/pm1` with the same 10-sample smoothing and five-second reporting interval as PM2.5 and PM10.

The mapping was checked against 40 valid raw frames and independently corroborated by the [Neutree Yishan-format decoder](https://github.com/Neutree/STM32f103DriverLib/blob/28dfb70b6b0aa8e0de940ae8aa417a3989dbe3a5/lib/offchip/yishan_PM2_5/yishan_PM2_5.cpp) and [Makeblock's PM sensor decoder](https://github.com/Makeblock-official/Makeblock-Libraries/blob/e76c787dbf6bbff4d322d2a7a835ea85bcece4da/src/MePm25Sensor.cpp). **The exact sensor model fitted by Aeris is still unconfirmed**; a matching protocol does not establish the model.

#### Defining Sensors and Fan

1.  In your configuration.yaml add the following (adjust the prefix if
    you changed that in Stage Two) and reboot your Home Assistant.

```
mqtt:
  sensor:
    - name: "Aeris PM1.0"
      state_topic: "aeris/purifier/sensor/pm1"
      unit_of_measurement: "µg/m³"
      device_class: pm1
      state_class: measurement
    - name: "Aeris PM2.5"
      state_topic: "aeris/purifier/sensor/pm25"
      unit_of_measurement: "µg/m³"
      device_class: pm25
      state_class: measurement
    - name: "Aeris PM10"
      state_topic: "aeris/purifier/sensor/pm10"
      unit_of_measurement: "µg/m³"
      device_class: pm10
      state_class: measurement
  fan:
    - name: "Aeris Air Purifier"
      state_topic: "aeris/purifier/fan/state"
      command_topic: "aeris/purifier/fan/set"
      percentage_state_topic: "aeris/purifier/fan/state"
      percentage_command_topic: "aeris/purifier/fan/set"
      speed_range_min: 1
      speed_range_max: 255
      payload_on: "30"
      payload_off: "0"
      optimistic: false
      state_value_template: >
        {% if value == '0' %}
          0
        {% else %}
          30
        {% endif %}
```

#### Automation YAML

This automation provides smart mode while the fan is on: a 10% floor through 40 µg/m³ PM2.5, then PM2.5 × 0.25% up to 100%, rounded to the firmware's 5% speed steps. A one-percentage-point tolerance accommodates MQTT's reported-speed rounding and avoids repeated commands.

It reads current states on PM updates, fan off-to-on, smart-mode off-to-on, Home Assistant startup, and once per minute. Enabling smart mode or turning the fan on therefore applies the curve even when PM is unchanged; unknown or unavailable PM readings are skipped.

Use the automation editor's YAML view. The entity IDs below match the sensor and fan names above; after saving, confirm the automation is `automation.air_purifier_smart_control` and adjust the references if Home Assistant assigned different IDs.

```yaml
alias: Air Purifier Smart Control
description: PM2.5-based control while the fan is on; manual OFF sticks.
triggers:
  - trigger: state
    entity_id: sensor.aeris_pm2_5
  - trigger: state
    entity_id: fan.aeris_air_purifier
    from: "off"
    to: "on"
  - trigger: state
    entity_id: automation.air_purifier_smart_control
    from: "off"
    to: "on"
  - trigger: homeassistant
    event: start
  - trigger: time_pattern
    minutes: "/1"
variables:
  target_speed: >-
    {% set pm = states('sensor.aeris_pm2_5') | float(0) %}
    {% set raw = 10 if pm <= 40 else ([pm * 0.25, 100] | min) | int %}
    {{ 5 * ((raw + 2) // 5) }}
conditions:
  - condition: state
    entity_id: automation.air_purifier_smart_control
    state: "on"
  - condition: state
    entity_id: fan.aeris_air_purifier
    state: "on"
  - condition: template
    value_template: "{{ is_number(states('sensor.aeris_pm2_5')) }}"
  - condition: template
    value_template: "{{ ((state_attr('fan.aeris_air_purifier', 'percentage') | float(0)) - (target_speed | float)) | abs > 1 }}"
actions:
  - action: fan.set_percentage
    target:
      entity_id: fan.aeris_air_purifier
    data:
      percentage: "{{ target_speed | int }}"
mode: restart
```

Smart mode is the enabled state of `automation.air_purifier_smart_control`; add that entity to your dashboard as its toggle. **Manual OFF stays off:** the automation only adjusts an already-running fan. **Disabling smart mode leaves the current speed unchanged:** manual speed control then persists through PM updates and minute ticks. These are automation behaviors, not firmware power-loss state persistence.

## Restoring Original Firmware

Thanks to @martinszelcel for noting how they were able to accomplish this!

If you would like to restore the original firmware that we backed up earlier you can run the below:

```dfu-util -d 2b04:d006 -a 0 -s 0x08020000:leave -D original_firmware.bin```


## Building This Fork From Source

Build from this fork to get the changes listed in [CHANGELOG.md](CHANGELOG.md); the upstream binary download is a different build. The Particle CLI command below uses the cloud compiler and requires a free Particle account (`particle login`). The resulting firmware operates locally without a Particle cloud connection. This part of the guide assumes you
have at least run through the driver install, backup, updating
bootloader, and updating the particle software sections.

If you already have this fork checked out, run step 6 from its `Source/` directory. Otherwise, prepare the same layout as follows.

1.  In the root of your hard drive create a folder 'aeris_project'

2.  Download this fork's [Source/project.properties](Source/project.properties) to the root as `project.properties`

3.  Create a sub-folder 'src'

4.  Download this fork's [Source/src/purifier.ino](Source/src/purifier.ino) to `src/purifier.ino`

5.  In command prompt, navigate to the aeris_project folder

<img src="./media/image28.png" alt="" width="600"/>

6.  Run the below command to compile the .ino code into a binary

```particle compile photon . --target 3.3.1 --saveTo aerisFirmware.bin```

<img src="./media/image29.png" alt="" width="600"/>

<img src="./media/image30.png" alt="" width="600"/>

7.  Flash the firmware to your device with:

    ```dfu-util.exe -d 2b04:d006 -a 0 -s 0x080A0000:leave -D aerisFirmware.bin```

The inherited settings signature uses the compile timestamp: the first boot of a newly compiled binary resets settings and enters setup mode. Plan to re-enter Wi-Fi, MQTT, topic-prefix and display settings after upgrading. Each purifier still needs its own topic prefix even though its MQTT client ID is now unique.
