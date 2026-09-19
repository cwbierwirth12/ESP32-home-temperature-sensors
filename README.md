# ESP32-Home-Temperature-Sensors

The purpose of this project is to deploy small ESP32 temperature and humidity sensors throughout your home and monitor them over your local network. This project uses ESPHome firmware, a Mosquitto MQTT broker, and a Node-RED diagnostic dashboard. ESPTH is the collection-only device utilizing an ESP32-C6 board; ESPTHSC adds a desktop display and online weather on an ESP32-S3 board. Home Assistant is an optional adaptation to be able to see your sensors, with no setup instructions included here.

This project is not turn-key. Users will need to configure the sensors for their home networks and make any adjustments necessary. Additionally the chips, boards, sensors all come bare bones and need to be assembled. 3D STL files are available for download to 3D print enclosures. At this time I do not know of any commercially available enclosures.

## Quickstart

1. Build the physical sensor (screen or no screen)
2. Copy `firmware/secrets.example.yaml` to private `firmware/secrets.yaml` and configure your network and location.
3. Compile the matching YAML with ESPHome and flash the resulting firmware over USB.
4. Choose a [Node-RED build](nodered/): a small one-sensor diagnostic flow or an always-on Raspberry Pi dashboard with SQLite history.
5. Connect sensors to power source
6. Connect to the Node-Red dashboard in your browser to monitor the temperatures. 

## Project Folders

  [firmware](firmware/) - ESPHome YAML templates, wiring pinouts, and USB/OTA installation instructions. ESPHome compiles the firmware; ESPHome Web flashes a compiled file. Private `secrets.yaml` and generated firmware are excluded by `.gitignore`.

  [Node-RED](nodered/) - Contains both a basic one-sensor diagnostic flow and a Raspberry Pi Zero 2W standalone build with multi-sensor SQLite history.

  [hardware](hardware/) - BOMs, Photos, and 3D STL files for the sensors

## Network and Ports

```text
Sensor
    |
    |
Home Network ------- Home Assistant (optional; instructions not included)
    |
    |
MQTT Broker
    |
    |
    |
Node-RED
```

| Service | Port | Used by |
| --- | --- | --- |
| MQTT broker | TCP `1883` | ESP32 devices and Node-RED connect to Mosquitto. |
| Node-RED editor and dashboard | TCP `1880` by default | Browser connects to the Node-RED computer; the port may differ if the user changed their Node-RED configuration. |
| ESPHome Device Builder | TCP `6052` by default | Browser connects to the firmware-building computer. |
| ESPHome OTA | TCP `3232` by default on ESP32 | ESPHome uploads firmware to a sensor with OTA enabled. |

For a standard local Node-RED installation, the editor is [localhost:1880](http://localhost:1880). The basic diagnostic flow uses `/dashboard/diagnostics`; the Raspberry Pi standalone build uses `/dashboard/temperature-history`. Use the port configured by your own installation if it differs. Use the server's LAN address from another device. No router port forwarding is required. Screen weather and time synchronization need internet access; local MQTT sensor collection does not.

## Tools and Software

- Firmware: ESPHome Device Builder; Docker is optional.
- DNode-Red Dashboard: a user-installed Node-RED instance plus the FlowFuse Dashboard add-on. Import the supplied diagnostic flow through the Node-RED editor.
- MQTT Broker: Mosquitto and its command-line clients.
- Enclosures: a 3D slicer of your choice.

Instructions for download and installation are found in each of the respective subfolders.


## Project Status and Disclosure

- I am not a programmer. Firmware, flow code, and documentation were developed with AI assistance. See [NOTICE.md](NOTICE.md) for attribution, third-party licenses, and testing limitations.

- This project is provided without warranty and is not intended for life-safety systems.

- This project is actively evolving. The screen enclosure is still in development, and the [firmware limitations](firmware/README.md#known-limitations) need review before calling the display firmware release-ready. Back up working configurations before upgrading.

- See the [publication review](AUDIT.md) for the checks performed, tested versions, and remaining hardware-validation work.

- Original project material is licensed under GNU GPL version 3 only (`GPL-3.0-only`). See [LICENSE](LICENSE) and [NOTICE.md](NOTICE.md). Third-party materials retain their own licenses.

# Project Thanks

- Huge thank you to my Sister for getting these components for me for Christmas 2025. Love you, sis.
- Thank you to my wife, for dealing with and embracing my nerdy endeavors. 
