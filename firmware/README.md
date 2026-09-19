# Firmware Setup

This guide explains how to configure and install the ESPHome firmware for the two sensor styles in this project. The supplied examples are built for an MQTT broker and Node-RED dashboard. They are not turnkey firmware: you must enter your own Wi-Fi, MQTT, and display-location settings before flashing.

## Contents

- [Choose a hardware type](#choose-a-hardware-type)
- [Understand the names](#understand-the-names)
- [Wire the hardware](#wire-the-hardware)
- [Install the required software](#install-the-required-software)
- [Choose a flashing method](#choose-a-flashing-method)
- [Configure private settings](#configure-private-settings)
- [Configure the no-screen sensor](#configure-the-no-screen-sensor-espth)
- [Configure the screen sensor](#configure-the-screen-sensor-espthsc)
- [Closing notes](#closing-notes)
- [Known limitations](#known-limitations)

## Choose a hardware type

There are two distinct devices in this project. Pick the YAML file that matches the device you built.

| Type | YAML file | Purpose |
| --- | --- | --- |
| No-screen sensor | `espth-1.yaml` | A small, low-cost collection sensor. It reads temperature and humidity and publishes them to MQTT for Node-RED or another subscriber. |
| Screen sensor | `espthsc-1.yaml` | A more capable desk/display tool. It has a local indoor sensor, a color screen, weather data, a clock, and optional MQTT support. |

The no-screen build is intended to be placed wherever you need a measurement. The screen build is meant to be useful on its own at a desk, counter, or other visible location.

These examples were developed around the following hardware:

- **No-screen:** Waveshare [ESP32-C6](https://www.waveshare.com/esp32-c6-zero.htm), using the ESP32-C6-DevKitC-1-compatible board definition, plus a **DHT22** temperature/humidity sensor.
- **Screen:** Waveshare [ESP32-S3 1.9inch Display Development Board](https://www.waveshare.com/esp32-s3-lcd-1.9.htm), plus a **BME280** temperature/humidity/pressure sensor.

The boards and sensors can be changed, but doing so requires updating the board definition, pin assignments, and possibly the display or sensor components in the YAML. Project-specific parts lists and enclosure information belong in the [hardware BOMs](../hardware/).

## Understand the names

Device names use a short, repeatable convention:

- `ESP` = ESP32-based device.
- `TH` = temperature and humidity sensor.
- `SC` = screen-equipped version.

Examples:

- `espth-1` means the first ESP32 temperature/humidity collection sensor.
- `espthsc-1` means the first ESP32 temperature/humidity sensor with a screen.

The examples use `espth-1` and `espthsc-1`. Change `device_name` and `friendly_name` in `substitutions:` for additional sensors. Keep every device name unique so that MQTT topics and ESPHome build folders do not collide.

## Wire the hardware

Disconnect USB power before changing any wiring. Double-check labels on your particular sensor breakout before applying power; clones and breakout boards do not always place the pins in the same physical order.

### No-screen sensor: ESP32-C6 and DHT22

The no-screen YAML expects the DHT22 data pin on `GPIO4`.

| DHT22 connection | Connect to ESP32-C6 |
| --- | --- |
| `VCC` or `+` | `3V3` |
| `DATA`, `OUT`, or `S` | `GPIO4` |
| `GND` or `-` | `GND` |

If you are using a bare four-pin DHT22 rather than a breakout module, add the pull-up resistor specified by the DHT22 documentation between `VCC` and `DATA`. Many ready-made DHT22 breakout boards already include it. The remaining bare-sensor pin is not connected.

### Screen sensor: ESP32-S3 display board and BME280

The LCD, backlight, and BOOT button are already built into the Waveshare ESP32-S3 display board. No display wiring is required. Connect only the external BME280 sensor:

| BME280 connection | Connect to ESP32-S3 display board |
| --- | --- |
| `VIN`, `VCC`, or `3V3` | `3V3` |
| `GND` | `GND VSYS` |
| `SDA` | `GPIO47` |
| `SCL` | `GPIO48` |

The supplied YAML expects the BME280 at I2C address `0x77`. Some BME280 breakouts use `0x76`; if the ESPHome I2C scan reports that address instead, change the YAML's `address:` value to match.

Use `GND VSYS` for the BME280 ground connection on this board. It proved more reliable in testing than the other `GND` header position.

The intended desktop orientation places the USB-C connector at the **9 o'clock / left-hand** side. The YAML uses `rotation: 270`. Using another orientation requires revising both rotation and drawing coordinates. Some views still contain portrait-sized coordinates and need correction for this landscape layout; see [Known limitations](#known-limitations).

## Install the required software

### ESPHome

ESPHome reads the YAML configuration, builds the firmware, and installs it on the ESP32. The simplest choice for most people is the **ESPHome Device Builder** desktop application.

1. Open the official [ESPHome installation page](https://esphome.io/install/).
2. Download the Device Builder for your operating system and complete its installation steps.
3. Open Device Builder. It will open a local browser dashboard where you can import and edit these YAML files.

For a first walk-through of the dashboard, see [Getting Started with ESPHome](https://esphome.io/install/getting-started/).

### Docker (optional)

Docker is **not required**. Use it only if you already prefer running ESPHome in a container instead of installing the Device Builder directly.

1. Install [Docker Desktop](https://www.docker.com/products/docker-desktop/).
2. Follow ESPHome's [Docker installation guide](https://esphome.io/install/docker/) to run the Device Builder against the folder that contains your YAML files.

A typical Docker Desktop setup on macOS does not provide direct USB serial passthrough for the first flash. Use the ESPHome Device Builder desktop app or a browser-based serial installer. Once the device is on Wi-Fi and OTA is enabled, future updates can be sent over the network.

### MQTT and Node-RED

The included collection-sensor example expects an MQTT broker and is designed to feed a Node-RED dashboard. Install and configure these separately:

- [Mosquitto MQTT broker](https://mosquitto.org/download/)
- [Node-RED](https://nodered.org/docs/getting-started/)

The firmware does not install or configure either service for you. Record the broker's local address, port, username, and password before continuing.

## Choose a flashing method

Every brand-new ESP32 needs its **first** firmware install over a USB data cable. OTA means *over the air*: it becomes available only after that first install succeeds and the device joins your Wi-Fi network.

### Method A: build and install with ESPHome Device Builder

This is the recommended method.

1. Connect the board to your computer with a USB **data** cable. Many charging-only cables will power the board but cannot flash it.
2. Open ESPHome Device Builder.
3. Add the YAML for your device and its private `secrets.yaml` to the Device Builder configuration directory. If your version offers file import, use it; otherwise copy the files into that directory and refresh the dashboard. Keep both files together so `!secret` resolves.
4. Complete the configuration steps below before clicking **Install**.
5. Click **Install** and choose the USB/serial option for the first flash.
6. Select the device's serial port when prompted. Allow the build to complete and wait for the upload confirmation.
7. After the device joins Wi-Fi, use **Install** again and choose the wireless option for later updates.

ESPHome's official [physical connection guide](https://esphome.io/guides/physical_device_connection/) covers common USB and board-connection problems.

### Method B: build a firmware file and flash it manually

Use this method when you want a reusable `.bin` firmware file or need a separate flashing tool.

1. Import and configure the YAML in ESPHome Device Builder.
2. Open the device menu, choose **Install**, then choose **Manual download**.
3. Save the generated firmware file. Choose the factory/first-install format when ESPHome offers that choice for a new board.
4. Open [ESPHome Web](https://web.esphome.io/) in Google Chrome or Microsoft Edge. These browsers support the USB connection the installer needs.
5. Connect the board with a USB data cable, click **Connect**, and allow the browser permission prompt.
6. Select the serial device that belongs to the board, click **Install**, then select the `.bin` file you downloaded.
7. Wait for the upload to complete, power-cycle the board if prompted, and confirm that it joins Wi-Fi.

Do not commit generated `.bin` files to this repository. The YAML is the source of truth; each user should build firmware for their own board and settings.

## Configure private settings

### 1. Keep credentials out of GitHub

The published starter file is [secrets.example.yaml](secrets.example.yaml). Before building:

1. Copy `secrets.example.yaml` to a private file named `secrets.yaml` in the same `firmware/` folder.
2. Enter your real credentials only in the private `secrets.yaml`.
3. Keep the root `.gitignore`, which already excludes private settings and build outputs, including:

```gitignore
secrets.yaml
*.bin
.esphome/
```

Never publish a Wi-Fi network name, Wi-Fi password, MQTT password, home IP address, or exact latitude/longitude.

### 2. Set Wi-Fi details

In your private `secrets.yaml`, replace the example values with your own network details:

```yaml
wifi_ssid: "YOUR_WIFI_SSID"
wifi_password: "YOUR_WIFI_PASSWORD"
```

ESPHome resolves `!secret wifi_ssid` and `!secret wifi_password` from this file automatically when the YAML and `secrets.yaml` are in the same folder.

### 3. Add MQTT details for the Node-RED path

For a GitHub-safe configuration, keep MQTT details in the same private secrets file:

```yaml
mqtt_broker: "YOUR_MQTT_BROKER_OR_LOCAL_ADDRESS"
mqtt_username: "YOUR_MQTT_USERNAME"
mqtt_password: "YOUR_MQTT_PASSWORD"
```

Both YAMLs already use these `!secret` keys. Both connect to Mosquitto on TCP `1883`. The Node-RED browser port, `1880`, is a different service and must not be entered as the MQTT port. For the temporary anonymous broker test, use empty strings for `mqtt_username` and `mqtt_password`.

### 4. Set fallback and OTA passwords

Replace `fallback_ap_password` with your own 8-63 character password for the setup hotspot. Replace `ota_password` with a private password for wireless firmware updates. The hotspot name is generated from the device name. OTA listens on TCP `3232` by default on these ESP32 devices; it is already enabled for ESPTHSC and optional for ESPTH.

### 5. Set the screen weather location and timezone

For ESPTHSC, change `timezone` in your private secrets file to your IANA timezone. `Etc/UTC` is a neutral example. In `weather_url`, replace `YOUR_LATITUDE` and `YOUR_LONGITUDE` with your weather location. Set the URL's `timezone=` parameter to the same timezone, encoding `/` as `%2F`; the example uses `Etc%2FUTC`. Keep the existing weather fields, units, and five-day forecast parameters because the display parser expects them.

These values are read through `!secret timezone` and `!secret weather_url`, so you do not need to put a location in the public screen YAML. The no-screen device does not use these two keys. Open-Meteo receives the coordinates when the display fetches weather; see its [data attribution and terms](https://open-meteo.com/en/terms).

## Configure the no-screen sensor (`espth`)

Open `espth-1.yaml`. This YAML is for the collection-only sensor: an ESP32-C6 board with a DHT22 connected to `GPIO4`.

1. **Name the device.** Set `device_name` and `friendly_name` under `substitutions:`. The example is `espth-1`.
2. **Configure MQTT.** Set the broker, username, and password. Prefer the `!secret` approach described above. Leave `discovery: false` if you are using the included Node-RED approach.
3. **Confirm Wi-Fi.** The file already uses `!secret wifi_ssid` and `!secret wifi_password`. Make sure your private `secrets.yaml` contains those keys.
4. **Set fallback-hotspot credentials.** Set `fallback_ap_password` in private `secrets.yaml`. The hotspot appears when normal Wi-Fi cannot be joined; `captive_portal:` enables its setup page.
5. **Confirm the hardware settings.** The supplied build expects a DHT22 on `GPIO4` and reports temperature in Fahrenheit every 15 minutes. Change the model, pin, units, or update interval only if your hardware or desired behavior differs.
6. **Enable OTA if you want wireless updates.** The `ota:` section is commented out in this sample. Remove the comment and use the current ESPHome form:

```yaml
ota:
  - platform: esphome
    password: !secret ota_password
```

7. Save the file, validate/build it in ESPHome, and perform the first USB flash.

After connection, the device publishes to `<device_name>/sensor/temperature/state` and `<device_name>/sensor/humidity/state`. Allow up to 15 minutes for the next DHT22 sample; temporarily shorten `update_interval` while diagnosing. Follow the [included Node-RED setup](../nodered/README.md) and select `<device_name>/sensor/+/state` to verify the data path. `npm start` loads the flow automatically.

## Configure the screen sensor (`espthsc`)

Open `espthsc-1.yaml`. This is the display-oriented build for the Waveshare ESP32-S3-LCD-1.9 and BME280 sensor.

### What the display does

The screen advances through seven views every 20 seconds unless blanked or pinned. These are the implemented views, subject to the layout limitations below. A short press wakes the display and advances the stored view index; while the clock is pinned, the clock remains visible until unpinned.

1. **Outside now:** current outside temperature, humidity, wind, and precipitation from Open-Meteo.
2. **Forecast:** today's high, low, precipitation, and short forecast text.
3. **Inside sensor:** local BME280 temperature and humidity.
4. **Inside vs. outside:** a side-by-side comparison and temperature difference.
5. **Five-day outlook:** daily high/low trend and precipitation display.
6. **Clock:** time, date, Wi-Fi state, device IP address, and battery estimate when available.
7. **Moon phase:** current moon phase plus upcoming new/full moon dates.

The board's physical **BOOT** button has three commands:

- **Short press:** wake the display and advance to the next view when the clock is unpinned.
- **Double short press:** pin or unpin the clock view.
- **Press and hold for at least one second:** blank the display. A later short press wakes it again.

### Reliability behavior

- MQTT telemetry can publish available indoor temperature, humidity, pressure, battery percentage, and a Unix timestamp every minute to `<device_name>/telemetry` when its commented configuration and publish action are enabled.
- MQTT is optional to the display itself. If the broker cannot be reached, the display stays running rather than rebooting. The broker must be reachable from the ESP32's Wi-Fi network; a VPN running only on a computer does not provide the ESP32 a route to that broker.
- The clock saves a snapshot, but it is not a battery-backed real-time clock. Do not rely on that saved timestamp after a restart; see the limitations below. Weather is cached in RAM while powered. The display disables ESPHome's Wi-Fi reboot timeout, so it keeps that cache while waiting for a temporary hotspot to return. A manual or power-cycle reboot still clears cached weather.
- The BME280 retries initialization once per minute if its first startup probe fails. Repeated I2C `NACK` errors indicate a sensor power, wiring, address, or seating problem.

### Screen YAML configuration checklist

1. **Name the device.** Set `device_name` and `friendly_name` under `substitutions:`. The example is `espthsc-1`.
2. **Set Wi-Fi, fallback-hotspot, and OTA details.** Enter the private values in `secrets.yaml`; the screen YAML already references them.
3. **Set your timezone.** Change `timezone` in private `secrets.yaml`.
4. **Set the weather location.** Replace the coordinate placeholders in private `weather_url` as described above.
5. **Make both timezones agree.** Keep `timezone` and the weather URL's `timezone=` parameter in sync so forecast hours line up with the clock.
6. **Enable MQTT only if needed.** The sample preserves a commented MQTT block and telemetry action. Uncomment both, then set the three `mqtt_*` values in private `secrets.yaml` to publish the one-minute telemetry feed. The display can operate without MQTT.
7. **Leave sensor/display pins alone unless you changed hardware.** This YAML is pinned for the listed Waveshare board and BME280 wiring. Different boards, displays, or sensors require their own pin/component changes.
8. Save the file, build it, and perform the first USB flash. This YAML already includes the ESPHome OTA component, so later updates can be sent over Wi-Fi after the device is connected.

## Closing notes

- Keep one backup of every working YAML before experimenting with changes.
- Give every device a unique name and MQTT log topic.
- Check the ESPHome logs after flashing. They are the fastest way to find a wrong Wi-Fi password, MQTT address, or disconnected sensor.
- The included firmware is designed around MQTT and Node-RED. ESPHome can also be adapted for Home Assistant's native API, but that alternate setup is not documented or supported by this project.
- Treat the display's weather location, network settings, and credentials as private configuration. Share only the cleaned template YAML files.

For project-wide context, return to the repository [README](../README.md).

## Known Limitations

Both supplied YAMLs compiled successfully with ESPHome 2026.8.2 during the [publication review](../AUDIT.md). Compilation is not a substitute for testing the actual boards.

- Several display views use Y coordinates beyond 170 even though `rotation: 270` creates a 320 x 170 drawing area. Those elements can be clipped. The five-day and moon layouts use landscape coordinates; the other views need a consistent layout pass and physical-board testing.
- The saved-clock fallback persists a `millis()` reference from the previous boot. Since `millis()` resets, its elapsed-time calculation is not reliable across restarts. A battery can help avoid a restart while moving the device, but it does not make this a reliable clock or data recorder.
- The voltage-based battery percentage is a rough estimate, not a measured state of charge. MQTT readings are not queued for later delivery through a network outage.
- Weather currently uses plain HTTP with `verify_ssl: false`; coordinates and weather requests are not encrypted in transit. HTTPS support and certificate validation need testing with the chosen ESPHome framework before changing this behavior.
- Moon phase and dates are approximate. They are not astronomical predictions for precise timing.
- ESPHome 2026.8.2 warns that `neopixelbus` and `st7789v` are deprecated; migrations to `esp32_rmt_led_strip` and `mipi_spi` need testing before a future ESPHome upgrade. It also flags GPIO4 on the C6 and GPIO0 on the S3 as strapping pins. These are the existing wiring/BOOT assignments; check startup behavior on your boards rather than suppressing the warnings.

Firmware and this guide were developed with AI assistance. Original project files use [GPL-3.0-only](../LICENSE); see [NOTICE.md](../NOTICE.md) for attribution and validation limits.
