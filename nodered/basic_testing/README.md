# Node-RED and MQTT Setup

This folder contains a small diagnostic dashboard for the ESP32 Home Temperature Sensors project. It is deliberately separate from a full home-automation setup: its job is to prove that a sensor can publish temperature and humidity data, that the MQTT broker receives it, and that Node-RED can display it.

You do not need previous networking or Node-RED experience. Follow the sections in order.

For an always-on Pi dashboard with SQLite history and multiple sensors, use the [Raspberry Pi Zero 2 W standalone build](../pi_zero_2w_Standalone/) instead.

## Contents

- [How the data moves](#how-the-data-moves)
- [Before you begin](#before-you-begin)
- [Install Mosquitto](#install-mosquitto)
- [Configure and test the MQTT broker](#configure-and-test-the-mqtt-broker)
- [Install Node-RED and import the flow](#install-node-red-and-import-the-flow)
- [Configure the included flow](#configure-the-included-flow)
- [Use the diagnostic dashboard](#use-the-diagnostic-dashboard)
- [Troubleshooting](#troubleshooting)

## How the data moves

```text
ESP32 sensor -> Wi-Fi network -> Mosquitto MQTT broker -> Node-RED flow -> web dashboard
```

- The ESP32 sensor measures temperature and humidity, then publishes the reading.
- The Wi-Fi network carries that message. The sensor, MQTT broker, and Node-RED machine must be able to reach one another on the same local network. Do not use a guest Wi-Fi network that blocks connected devices from communicating.
- Mosquitto is the MQTT broker. Think of it as a local post office: sensors send it messages, and other programs subscribe to receive them.
- Node-RED subscribes to the broker, interprets the readings, and presents them in a browser dashboard.

Mosquitto and Node-RED can run on the same computer or Raspberry Pi. That is the easiest arrangement for this example.

Important: `localhost` always means the device running the software. It is correct for Node-RED when Mosquitto runs on the same machine. It is never correct in an ESP32 YAML file, because on the ESP32 `localhost` would mean the ESP32 itself. The sensor needs the local IP address or hostname of the machine running Mosquitto.

## Before you begin

Have these ready:

- A working ESP32 sensor that has joined your Wi-Fi network.
- A computer or Raspberry Pi that will run Mosquitto and Node-RED.
- A USB data cable for the ESP32's first firmware flash, if it has not already been flashed.
- The local IP address of the computer or Pi that will run Mosquitto. Use this address in the ESP32 MQTT configuration.

For the safest first test, use a trusted home or lab network. MQTT username/password authentication is optional for a temporary test, but should be enabled before leaving the broker running on a normal network.

The sensor uses TCP `1883` to reach Mosquitto. Your browser uses the port configured for your Node-RED installation, normally TCP `1880`. They are separate services. Allow these applications through the host firewall only for your trusted LAN; do not set up router port forwarding. This repository does not configure an editor login; before broader use, follow [Node-RED's security guide](https://nodered.org/docs/user-guide/runtime/securing-node-red). MQTT passwords on port `1883` are not encrypted in transit.

To find the broker computer's LAN address: run `ipconfig` in Windows PowerShell (active adapter's IPv4 address), `ipconfig getifaddr en0` on a typical Mac Wi-Fi connection (or look in System Settings > Network > Wi-Fi > Details > TCP/IP), or `hostname -I` on Linux/Pi (choose the address on the sensor's LAN, not a container/VPN address). An iPhone hotspot may restrict communication between clients; a successful internet connection alone does not prove local MQTT can connect.

## Install Mosquitto

Mosquitto is the MQTT broker. Install both the broker and its command-line tools; the tools let you prove the broker works before Node-RED is involved.

### Windows

1. Download and install Mosquitto from the official [Mosquitto download page](https://mosquitto.org/download/).
2. Open PowerShell after the installation completes.
3. Confirm the broker executable is available. If the installer did not add it to your PATH, use its usual installation location:

```powershell
& "C:\Program Files\mosquitto\mosquitto.exe" -h
```

### macOS

1. Install [Homebrew](https://brew.sh/) if you do not already use it.
2. Open Terminal and run:

```bash
brew install mosquitto
mosquitto -h
```

### Linux and Raspberry Pi OS

On Raspberry Pi OS, Ubuntu, Debian, and other Debian-based systems, run:

```bash
sudo apt update
sudo apt install -y mosquitto mosquitto-clients
```

The package may start a background Mosquitto service automatically. For this guide's foreground test, stop it first so the test broker can use port `1883`:

```bash
sudo systemctl stop mosquitto
```

See the [Mosquitto download page](https://mosquitto.org/download/) for other Linux distributions.

## Configure and test the MQTT broker

Start with an intentionally simple configuration so that you can verify the network path. Then add a username and password before treating the broker as a long-running home service.

### 1. Create a temporary test configuration

Create a file named `mosquitto.conf` in your home folder. Use this content:

An identical starting configuration is included as [mosquitto.example.conf](mosquitto.example.conf). Copy it into your home folder as `mosquitto.conf`, or create it below. Keep your real broker configuration and password file outside the repository.

```conf
listener 1883
allow_anonymous true
```

**Windows PowerShell**

```powershell
notepad "$HOME\mosquitto.conf"
```

**macOS, Linux, or Raspberry Pi**

```bash
nano ~/mosquitto.conf
```

Paste the configuration above, save the file, and close the editor. In Nano, press `Ctrl+O`, `Enter`, then `Ctrl+X`.

`1883` is the normal unencrypted MQTT port. `allow_anonymous true` means devices on your trusted local network can connect without a username or password. Do not use this setting on a network you do not trust.

### 2. Start the broker

Open a terminal and leave this command running. The `-v` option prints connection and message activity.

Only one broker can listen on port `1883`. If it is already in use, stop your test broker or the newly installed service first. On Windows, check the Mosquitto entry in Services; on macOS, check `brew services list` and use `brew services stop mosquitto` if you started it that way. Do not stop a broker serving other home devices without accounting for that interruption.

**Windows PowerShell**

```powershell
& "C:\Program Files\mosquitto\mosquitto.exe" -c "$HOME\mosquitto.conf" -v
```

**macOS**

```bash
mosquitto -c ~/mosquitto.conf -v
```

**Linux or Raspberry Pi**

```bash
mosquitto -c ~/mosquitto.conf -v
```

For now, run the Linux command as your normal user, without `sudo`, so it can read files in your home directory. A permanent system service uses a different configuration location and file permissions; see the service notes below after testing.

### 3. Prove the broker works before involving the sensor

Open a second terminal window. Run the subscribe command and leave it open.

**Windows PowerShell**

```powershell
& "C:\Program Files\mosquitto\mosquitto_sub.exe" -h localhost -p 1883 -t "test/temperature" -v
```

**macOS, Linux, or Raspberry Pi**

```bash
mosquitto_sub -h localhost -p 1883 -t "test/temperature" -v
```

Open a third terminal window and publish a test reading.

**Windows PowerShell**

```powershell
& "C:\Program Files\mosquitto\mosquitto_pub.exe" -h localhost -p 1883 -t "test/temperature" -m "72.4"
```

**macOS, Linux, or Raspberry Pi**

```bash
mosquitto_pub -h localhost -p 1883 -t "test/temperature" -m "72.4"
```

The subscribe terminal should print:

```text
test/temperature 72.4
```

This proves local publish/subscribe works on the broker computer. To test the network path, run the same publisher from a second computer on the sensor's network, replacing `localhost` with the broker's LAN address. Seeing its message in the subscriber confirms cross-device communication. The ESP32 connection test below is still necessary.

### 4. Optional: require an MQTT username and password

For a normal home deployment, use a password rather than anonymous access.

1. Create a password file. The command will prompt for a password instead of placing it in your terminal history.

`-c` creates a new file and overwrites an existing one. Omit `-c` when updating an account or adding another user to an existing file.

**Windows PowerShell**

```powershell
& "C:\Program Files\mosquitto\mosquitto_passwd.exe" -c "$HOME\mosquitto.passwd" example_sensor_user
```

**macOS, Linux, or Raspberry Pi**

```bash
mosquitto_passwd -c ~/mosquitto.passwd example_sensor_user
```

2. Replace the contents of `mosquitto.conf` with the following. Replace the password-file path with the full path to the file you just created. On Windows, use forward slashes in the path, such as `C:/Users/YOUR_WINDOWS_USERNAME/mosquitto.passwd`.

```conf
listener 1883
allow_anonymous false
password_file /full/path/to/mosquitto.passwd
```

3. Stop the running broker with `Ctrl+C`, then start it again using the command for your operating system above.
4. Retest with a username and password. Substitute your own password:

**Windows PowerShell**

```powershell
& "C:\Program Files\mosquitto\mosquitto_sub.exe" -h localhost -p 1883 -u example_sensor_user -P "YOUR_PASSWORD" -t "test/temperature" -v
```

**macOS, Linux, or Raspberry Pi**

```bash
mosquitto_sub -h localhost -p 1883 -u example_sensor_user -P "YOUR_PASSWORD" -t "test/temperature" -v
```

In another terminal, publish an authenticated test message:

**Windows PowerShell**

```powershell
& "C:\Program Files\mosquitto\mosquitto_pub.exe" -h localhost -p 1883 -u example_sensor_user -P "YOUR_PASSWORD" -t "test/temperature" -m "72.4"
```

**macOS, Linux, or Raspberry Pi**

```bash
mosquitto_pub -h localhost -p 1883 -u example_sensor_user -P "YOUR_PASSWORD" -t "test/temperature" -m "72.4"
```

Expect `test/temperature 72.4` again. The `-P` examples expose the password in command history and potentially the process list; use temporary test credentials, and do not share terminal output or screenshots containing them.

The ESP32 YAML and Node-RED broker settings must use the same port, username, and password. Mosquitto's documentation covers [password files](https://mosquitto.org/documentation/authentication-methods/) and [listener settings](https://mosquitto.org/man/mosquitto-conf-5.html) in more depth.

### 5. Connect the ESP32

Both ESP32 YAMLs already read the MQTT connection from private `firmware/secrets.yaml`. Copy [secrets.example.yaml](../../firmware/secrets.example.yaml) to that filename as described in the [firmware guide](../../firmware/README.md). Set `mqtt_broker` to the Mosquitto computer's LAN address. The public YAML should retain these references:

```yaml
mqtt:
  broker: !secret mqtt_broker
  port: 1883
  username: !secret mqtt_username
  password: !secret mqtt_password
```

Flash or update the sensor, then watch the Mosquitto terminal. A successful connection appears there. To see every topic from the sensor, use the version for your operating system below.

**Windows PowerShell**

```powershell
& "C:\Program Files\mosquitto\mosquitto_sub.exe" -h localhost -p 1883 -u example_sensor_user -P "YOUR_PASSWORD" -t "#" -v
```

**macOS, Linux, or Raspberry Pi**

```bash
mosquitto_sub -h localhost -p 1883 -u example_sensor_user -P "YOUR_PASSWORD" -t "#" -v
```

If you chose anonymous access for the temporary test, remove `-u example_sensor_user -P "YOUR_PASSWORD"` from the command.

For that anonymous test, also set `mqtt_username: ""` and `mqtt_password: ""` in private `secrets.yaml`, and leave the Node-RED broker Security fields empty. With authentication enabled, all three clients must use the account created above.

Expected sensor traffic is `<device_name>/telemetry` JSON every minute for ESPTHSC, or separate `<device_name>/sensor/temperature/state` and `humidity/state` readings every 15 minutes for ESPTH. The screen also publishes other ESPHome entity states. Seeing those messages in the subscriber confirms that the real sensor is reaching the broker.

### 6. Optional: a permanent Linux/Pi broker service

After the foreground test works, stop it with `Ctrl+C`. For a new dedicated broker, edit `/etc/mosquitto/conf.d/temperature.conf` using `sudo nano /etc/mosquitto/conf.d/temperature.conf`. Add the listener and authentication settings above, but use `password_file /etc/mosquitto/temperature.passwd`. Create that file and start the service:

```bash
sudo mosquitto_passwd -c /etc/mosquitto/temperature.passwd example_sensor_user
sudo chown root:mosquitto /etc/mosquitto/temperature.passwd
sudo chmod 640 /etc/mosquitto/temperature.passwd
sudo systemctl enable --now mosquitto
sudo systemctl restart mosquitto
sudo journalctl -u mosquitto -n 30 --no-pager
```

Use `-c` only for a new password file. Confirm `/etc/mosquitto/mosquitto.conf` includes `/etc/mosquitto/conf.d`, and avoid duplicate listeners if other broker configuration already exists. This service setup applies to Debian-based installations; it does not use the test `~/mosquitto.conf` automatically.

## Install Node-RED and import the flow

This repository supplies an example flow, not a complete Node-RED installation. Install, secure, and run Node-RED using the method that suits your computer or Raspberry Pi. The official [Node-RED installation guides](https://nodered.org/docs/getting-started/) cover Windows, macOS, Linux, and Raspberry Pi.

The flow needs the **FlowFuse Dashboard** add-on because it uses Dashboard 2.0 nodes such as `ui-base`, `ui-page`, and `ui-text`.

1. Install and start Node-RED using the official guide for your operating system.
2. Open its editor. A standard local installation uses `http://localhost:1880`. If yours uses a different port, use that address throughout this guide.
3. In the editor menu, choose **Manage palette**, open the **Install** tab, search for `@flowfuse/node-red-dashboard`, and install it. Restart Node-RED if prompted.
4. Choose the editor menu, then **Import**. Select [espth-diag-flows.json](espth-diag-flows.json) from this folder and import it.
5. The imported **Temperature Diagnostics** tab appears in the workspace. Importing adds this tab to your existing flows; it does not replace them.

The example uses the standard Node-RED dashboard route `/dashboard/diagnostics`. Its full address is normally `http://localhost:1880/dashboard/diagnostics`.

## Configure the included flow

Import [espth-diag-flows.json](espth-diag-flows.json) using the steps above, then configure it before clicking **Deploy**. The flow has no saved MQTT username or password.

### 1. Set the MQTT broker details

1. Open the Node-RED editor, normally at `http://localhost:1880`.
2. Find and double-click the node named **Sensor MQTT input**.
3. Select the topic for **one indoor sensor**. The default is `espthsc-1/telemetry` for the screen sensor. Use `espth-1/sensor/+/state` for the no-screen sensor. Replace the device name if you changed it in YAML. Avoid `#` here: the broker can carry readings from multiple devices and the display's outside weather sensors.
4. Next to the server field named **Local Mosquitto**, click the pencil icon.
5. Set the broker host:
   - Use `localhost` when Mosquitto and Node-RED run on the same computer or Pi.
   - Use the broker machine's local IP address or hostname when Mosquitto runs elsewhere.
6. Set the port to `1883`, unless you deliberately changed the Mosquitto listener to another port.
7. Open the security settings and enter the MQTT username/password if you enabled authentication.
8. Click **Update**, then **Done**, then **Deploy**.

The MQTT input node should show **connected** beneath it after deployment. If it does not, return to the terminal MQTT test before changing the flow further.

The broker client ID is left blank so Node-RED can generate one; if you set it manually, use a unique ID for every running client. Duplicate IDs can make clients disconnect each other. If you change the MQTT port, update Mosquitto's listener, both YAMLs, this broker configuration, and your test commands together. The Node-RED browser port is controlled by your own Node-RED installation and does not need to match the MQTT port.

### 2. What the flow expects

The parser accepts the indoor numeric state topics produced by the example YAMLs (the supplied numeric temperature states are Fahrenheit):

```text
espthsc-1/sensor/indoor_temperature/state 70.2
espthsc-1/sensor/indoor_humidity/state 51.4
espth-1/sensor/temperature/state 70.2
espth-1/sensor/humidity/state 51.4
```

It also accepts combined JSON on `<device_name>/telemetry`, which is the recommended screen input:

```json
{
  "timestamp": 1700000000,
  "temperature_f": 70.2,
  "humidity_percent": 51.4
}
```

Telemetry accepts `temperature_f` or `temperature_c`, `humidity_percent`, and an optional `timestamp` (Unix seconds, Unix milliseconds, or a parseable date string). It retains the latest temperature and humidity for the same device so separate messages produce one reading. Outside weather, debug/status messages, malformed JSON, and invalid values are ignored. When a different device supplies data, previous values are cleared to avoid combining two devices. Pressure and battery fields are not shown on this dashboard.

This is a one-device diagnostic view. To compare rooms simultaneously, extend the flow with separate device state and widgets. If you change the firmware's numeric state topics to Celsius, also update the parser's default unit; changing just the label does not convert a measurement.

## Use the diagnostic dashboard

Open:

```text
http://localhost:1880/dashboard/diagnostics
```

The Diagnostics page should show:

- The latest temperature reading.
- The latest humidity reading.
- The timestamp of the latest valid temperature reading. Telemetry uses its source timestamp when supplied; plain state topics use the Node-RED receive time. A humidity-only update does not advance this timestamp.

Before readings arrive, widgets may be empty; after a partial reading, missing values show `No reading yet`. Wait for the sensor's sample interval. The screen telemetry interval is one minute; the no-screen interval is 15 minutes. Enable **Raw MQTT (enable to inspect)** in the editor and open the Debug sidebar to inspect incoming messages. Disable it after testing, and scrub payloads before sharing screenshots or logs. The flow does not store history or mark stale readings automatically; retained MQTT state can be older than its receive timestamp.

## Troubleshooting

| Symptom | Check |
| --- | --- |
| `mosquitto_sub` does not receive the test message | Confirm the broker is still running, use the same port in both commands, and check the broker terminal for errors. |
| ESP32 cannot connect | Use the Mosquitto machine's LAN IP in the ESP32 YAML, not `localhost`. Confirm the ESP32 and broker are on the same non-guest network and allow Mosquitto through the computer firewall. |
| Node-RED MQTT node says disconnected | Confirm host, port, username, and password match the working terminal test. Use `localhost` only when Node-RED and Mosquitto run on the same machine. |
| Dashboard page does not open | Confirm Node-RED is running, FlowFuse Dashboard is installed, and use the port configured by your Node-RED installation. The usual default is `1880`. |
| Dashboard opens but has no readings | Repeat the OS-specific subscriber command above with your actual broker host, port, and credentials. Confirm the input topic matches the device name. Enable the supplied Debug node and wait for the sensor interval. |
| Another computer cannot open the editor/dashboard | Replace `localhost` in the browser URL with the Node-RED machine's IP address. Check that the firewall permits your configured Node-RED port, normally `1880`. |

Once this diagnostic setup works, you have a clean foundation for a larger dashboard, alerts, room labels, historical storage, and your broader home temperature-sensor system.

## Sharing and Attribution

The root `.gitignore` excludes Node-RED runtime configuration, credentials, sessions, and backups. They can contain private paths, passwords, or the key needed to decrypt `flows_cred.json`. Never upload a running Node-RED user directory wholesale. Review changes to [espth-diag-flows.json](espth-diag-flows.json) before committing because broker addresses and custom payloads can be saved there. Enter your broker login in your own Node-RED editor; credentials are not supplied by this project.

The flow and this guide were developed with AI assistance and use [GPL-3.0-only](../../LICENSE). FlowFuse Dashboard and Node-RED retain their upstream licenses. See [NOTICE.md](../../NOTICE.md).
