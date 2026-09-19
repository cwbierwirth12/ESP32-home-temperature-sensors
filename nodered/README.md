# Node-RED Builds

This folder has two different Node-RED paths. Both receive ESP32 readings through Mosquitto on TCP `1883`, but they serve different purposes.

| Build | Best for | What it includes |
| --- | --- | --- |
| [Basic testing](../nodered/basic_testing/README.md) | Confirming one sensor, broker, and dashboard work | A small importable diagnostic flow and beginner MQTT guide |
| [Raspberry Pi Zero 2 W standalone dashboard](../nodered/pi_zero_2w_Standalone/README.md) | A permanent, whole-home local monitor | Pi OS setup, Mosquitto, SQLite history, a multi-sensor Dashboard 2.0 flow, and optional OpenWeather data |

Start with **Basic testing** when you are new to MQTT or Node-RED. Use the Raspberry Pi build when you want the dashboard and history database to run continuously without leaving a computer on.

Both flows are imported into a user-installed Node-RED instance. Neither flow contains private Wi-Fi details, MQTT credentials, database files, or API keys. Review a flow before exporting it from your own Node-RED installation.

Original project files are offered under [GPL-3.0-only](../LICENSE). See [NOTICE.md](../NOTICE.md) for attribution and AI-assistance disclosure.
