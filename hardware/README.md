# Hardware and Enclosures

This folder documents the physical parts and 3D-printed enclosures for the two ESP32 Home Temperature Sensor variants:

- **ESPTH**: a compact, no-screen temperature and humidity sensor intended for data collection.
- **ESPTHSC**: a screen-equipped small temperature and humidity display intended for desktop use.

The firmware setup, pin wiring, and software configuration are documented in the [firmware README](../firmware/README.md).

## Contents

- [ESPTH bill of materials](#espth-bill-of-materials)
- [ESPTHSC bill of materials](#espthsc-bill-of-materials)
- [ESPTHSC display and controls](#espthsc-display-and-controls)
- [Enclosures and assembly](#enclosures-and-assembly)
- [3D-printing guidance](#3d-printing-guidance)
- [Troubleshooting](#troubleshooting)
- [Availability](#availability)

## ESPTH bill of materials

The no-screen ESPTH is the small, collection-only sensor.

| Part | Notes |
| --- | --- |
| [Waveshare ESP32-C6-Zero](https://www.waveshare.com/product/mcu-tools/development-boards/esp32-c6-zero.htm) | The ESP32 controller board used for this build. |
| [Waveshare DHT22 temperature and humidity sensor](https://www.waveshare.com/dht22-temperature-humidity-sensor.htm) | Measures local temperature and humidity. |
| USB-C data/power cable | Use a cable that supports data for the initial firmware flash. A charge-only cable will not work for flashing. |
| Regulated 5 V USB power source, 1 A or greater recommended | This is a practical adapter recommendation for the project, not a published Waveshare minimum current requirement. See the power notes below. |
| [ESPTH STL files](./3D%20Print/espth_files/STL/) | Base, lid, and two holding plates. [Editable STEP files](./3D%20Print/espth_files/STEP/) are also included. |
| Sensor wiring | Three conductors for power, ground, and data; choose connectors or soldered leads to fit the enclosure. |
| M2 x 4 mm and M2 x 6 mm screws | The enclosure is designed around these sizes. This [M2 screw kit](https://www.amazon.com/dp/B0D3X4LJD2) is the set used during development. |

## ESPTHSC bill of materials

The ESPTHSC is the screen-equipped version. It shows local and remote weather information and is intended to live on a desk, shelf, or other visible spot.

| Part | Notes |
| --- | --- |
| [Waveshare ESP32-S3-LCD-1.9](https://www.waveshare.com/esp32-s3-lcd-1.9.htm) | ESP32-S3 board with built-in 1.9-inch display. |
| [Waveshare DHT22 temperature and humidity sensor](https://www.waveshare.com/dht22-temperature-humidity-sensor.htm) | Measures local temperature and humidity. |
| USB-C data/power cable | Use a data-capable cable for the first firmware flash. |
| Regulated 5 V USB power source, 1 A or greater recommended | Practical adapter recommendation; actual current varies with display, Wi-Fi, peripherals, and battery charging. |
| Optional 3.7 V 450 mAh 502535 LiPo battery | The [example battery](https://www.amazon.com/dp/B0GDQMKQ36) is useful as short-term backup power while moving the display between rooms or disconnecting it from a computer. It is not intended to replace normal USB power. **Check the connector before ordering:** Waveshare documents an MX1.25 battery header on this board, while this example battery is listed with a JST-PH 2.0 connector. Use an appropriate adapter or a battery with the correct connector; never force a connector. |
| ESPTHSC enclosure files | Still in development; files will be added under [3D Print](./3D%20Print/) when ready. No screen STL or STEP files are currently included. |
| Sensor wiring | Three conductors for power, ground, and data. The supplied firmware uses `GPIO6` for the DHT22 data line. |
| M2 x 4 mm and M2 x 6 mm screws | This [M2 screw kit](https://www.amazon.com/dp/B0D3X4LJD2) is the set used during development. |

## ESPTHSC display and controls

The ESPTHSC firmware is configured for desktop use with the USB-C connector at the **9 o'clock / left-hand** side. It automatically advances every 20 seconds through this sequence:

1. Indoor temperature and humidity from the DHT22.
2. Indoor versus outside temperature comparison.
3. Current outdoor conditions, including wind speed and compass direction.
4. 12-hour outlook with two-hour samples for temperature, weather, precipitation, wind, and gust alerts.
5. Five-day forecast.
6. Moon phase.
7. Clock, network status, and battery estimate.

The physical **BOOT** button controls the display:

- **One short press:** wakes the display and advances to the next view. If clock-only mode is active, it exits that mode and returns to the indoor view.
- **Two short presses:** toggles clock-only mode. Enabling it opens the clock immediately; disabling it returns to the indoor view.
- **Press and hold for at least one second:** blanks the display. A later short press wakes it.

The short-press action waits 400 ms before it runs so the firmware can reliably distinguish a single press from a double press.

## Raspberry Pi Zero 2W Standalone  (OPTIONAL)

| Part | Notes |
| --- | --- |
| [Raspberry Pi Zero 2 W](https://www.waveshare.com/product/raspberry-pi/boards-kits/raspberry-pi-zero-2-w-cat/raspberry-pi-zero-2-w.htm) | |
| [32 GB SD Card](https://www.amazon.com/SanDisk-Ultra%C2%AE-microSDHC-120MB-Class/dp/B08L5HMJVW/ref=sr_1_4?dib=eyJ2IjoiMSJ9.2sBrO0Sb1oPospxdSFn4yZ1xGvdNdjt_G3KWxn-8V2RBIbe9a04ATK-LaP5H75VdWrNemUAs1TNhE6VQl_eNK7Kc1onwbGV9s_t96SKWX3n4Q45FrnCY3UxH7GYjtBBQ5lL0mb5Eg5wMJse0nz0XbRG4i0s0vtxUrs9nQj1Ft__0g9LRbHWSZzzpCTkPJ4EiG-pYVPhGJwIo3MANt0ndwu6Beq_2Xl7mnWflhPCtKWw.ymg34nY28oz2_uNeSDwVXG54Gv4qfY80SvgIJu35obQ&dib_tag=se&keywords=32gb%2Bmicro%2Bsd%2Bcard&qid=1789494628&sr=8-4&th=1)| |
| Micro-USB Cable |
| Regulated 5 V USB power source, 2.5 A or greater recommended | Practical adapter recommendation; actual current varies with display, Wi-Fi, peripherals, and battery charging. |

### Power and Battery Notes

Waveshare's [C6-Zero FAQ](https://docs.waveshare.com/ESP32-C6-Zero/FAQ) associates USB instability with supply voltage falling below 4.9 V. Its [board description](https://docs.waveshare.com/ESP32-C6-Zero) lists an 800 mA maximum output for the onboard 3.3 V regulator; that is not the board's measured current draw or a USB-adapter requirement. A regulated 5 V adapter rated for 1 A or more is the project's practical starting recommendation for either build. A higher current rating provides capacity; the board draws what it needs.

The [screen-board documentation](https://docs.waveshare.com/ESP32-S3-LCD-1.9) specifies a 3.7 V battery interface with an MX1.25 connector. The linked battery's JST-PH 2.0 plug is not a direct fit. Before use, verify the exact connector, positive/negative polarity, cell charging limits, and available case space. An adapter's physical fit alone does not establish electrical compatibility. Do not compress, puncture, or use a damaged or swollen LiPo cell.

The optional battery is intended to bridge short USB disconnections while moving the display. Continued power can preserve its in-memory state, but uninterrupted operation and runtime have not been verified for the linked pack. It does not preserve Wi-Fi connectivity between networks, guarantee MQTT delivery, or provide historical data storage.

## Enclosures and assembly

The current printable ESPTH files are in [3D Print/espth_files](./3D%20Print/espth_files/). Flash, configure, and test the boards before closing the enclosure. Disconnect USB and any battery before wiring or assembly. [Build photos](./Photos/espth/) show the no-screen enclosure.

- The STL filenames identify their purpose: base, lid, ESP32-C6 holding plate, and DHT22 holding plate.
- The thin holding plates secure the electronics inside the case.
- Use **M2 x 4 mm** screws for the internal holding plates.
- Use **M2 x 6 mm** screws to attach the lid to the base.
- The screw holes are countersunk for the heads in the recommended screw kit.
- Editable STEP versions are included for anyone who wants to adapt the design for different hardware, mounting locations, or preferences.

The ESPTHSC enclosure is currently in development and will be released here once it is ready.

## 3D-printing guidance

These enclosures are not highly structural parts, so use the material and printer settings that work best for you.

- PLA or PETG can be used; PETG is the maintainer's preference. Choose settings appropriate to your printer and filament.
- The maintainer used **20% infill** for these lightly loaded enclosures. Check fit and strength on your printer before final assembly.
- The DHT22 has a power indicator LED. It may shine through translucent or light-colored filament; that is normal.
- For installations with several sensors nearby, add a number to each case and fill it with acrylic paint or nail polish. It makes individual devices much easier to identify later.

Keep the sensor away from direct sunlight, HVAC vents, radiators, computers, and other heat sources. The enclosure protects the components, but it cannot make a poor measuring location accurate.

## Troubleshooting

| Symptom | Things to check |
| --- | --- |
| The board does not power on or repeatedly restarts | Try another USB-C cable and a known-good 5 V, 1 A or greater USB power source. The ESP32-C6-Zero documentation notes that its USB supply should remain at least 4.9 V. |
| The board cannot be flashed | Use a USB data cable, not a charge-only cable. Try a different USB port. For the C6-Zero, hold **BOOT**, press and release **RESET**, then release **BOOT** to enter download mode. |
| No temperature or humidity reading | Recheck the wiring and the pins listed in the [firmware README](../firmware/README.md). Make sure the correct YAML file is flashed for the hardware you built. |
| Measurements seem inaccurate | Give the sensor time to acclimate after power-up or moving it. Relocate it away from sunlight, airflow, warm electronics, or walls that hold heat. |
| The enclosure does not close cleanly | Confirm that the correct holding plate is installed, the components are seated fully, and M2 x 4 mm screws are only used for the internal plates. Do not overtighten plastic parts. |
| Battery will not connect to the ESPTHSC board | Stop and compare the battery plug with the board's MX1.25 header. Do not force a JST-PH 2.0 plug onto a different connector. |

## Availability

Pre-built systems and printed cases are not available for purchase at this time. This is a build-it-yourself project: the files, parts list, and firmware are provided so you can make and modify the system yourself.

This guide was developed with AI assistance. Project enclosure files and photographs are covered by [GPL-3.0-only](../LICENSE); see [NOTICE.md](../NOTICE.md) for attribution and third-party exceptions. The documentation does not claim that the CAD models or photographs were AI-generated.
