# ESP32 BLE Proxy Enhanced

> An ESPHome-based Bluetooth Proxy firmware, offering a plug-and-play BLE gateway experience for Home Assistant  
> Reference hardware: `esp32-getway` (OSHWHub: [https://oshwhub.com/yanqisui/esp32-getway](https://oshwhub.com/yanqisui/esp32-getway))

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg?style=for-the-badge)](LICENSE)
[![ESPHome](https://img.shields.io/badge/HomeAssistant-ESPHome-blue.svg?style=for-the-badge)](https://esphome.io/)

[中文版本请点击这里](README_zh.md)


## 🕊️ Introduction

This hardware was originally released by Ai-Thinker in 2022, designed as a BLE gateway development board.  
Its design is elegant, with stable power, complete interfaces, and even a beautiful enclosure -- perfect for use as a BLE proxy.  

However, over time, the original firmware ecosystem has become outdated: old toolchains, lack of documentation, and no Home Assistant support.  
To most modern smart home users, it’s now little more than “a brilliant yet lonely piece of hardware.”  

...So let’s gently breathe new life into it\~ ✨  
This project is based on **ESPHome Bluetooth Proxy**, and reimagines this board as:  

* ✅ A modern, seamless bridge to Home Assistant
* ✅ With intuitive **LED indicators** and **button input**
* ✅ Selectable **wired (Ethernet)** or **wireless (Wi-Fi)** connectivity
* ✅ Powered by Home Assistant’s built-in `habluetooth` framework, supporting native link balancing and automatic load distribution across proxies

> ⚡ **Note**  
> This is an independent project, not affiliated with the original hardware vendor.  
> **Please do not request support from the original manufacturer.**  
> For questions, open an issue in this repository.


## ✨ Features

* **Dual firmware versions**:

  * `eth-version.yaml`: LAN8720 Ethernet version
  * `wifi-version.yaml`: 2.4GHz Wi-Fi version (with fallback AP support)
* **BLE indicator LED**:

  * Broadcast received → LED flashes for 50ms
  * GATT connection active → LED stays on
* **Network status indicators**:

  * `NET`: Indicates Home Assistant API connection status
  * `WIFI`: Indicates Wi-Fi connection status (Wi-Fi version only)
* **Button input**:

  * GPIO34 as button → usable as a trigger in Home Assistant automations
* **Web configuration panel**:

  * Optional ESPHome `web_server` for easy testing


## ⚙️ Hardware & Pinout

Pin mapping is as follows:

| Function     | GPIO   | Description                                    |
|--------------|--------|------------------------------------------------|
| BLE LED      | GPIO15 | Active LOW                                     |
| Wi-Fi LED    | GPIO2  | Active LOW (Wi-Fi version)                     |
| Ethernet LED | GPIO16 | Active LOW (Ethernet version)                  |
| Button input | GPIO34 | External pull-up, active LOW                   |
| OSC Enable   | GPIO17 | Pull HIGH to enable external 50 MHz oscillator |

### 💡 Ethernet Version Notes

* This firmware uses an **external 50 MHz oscillator** as RMII clock input. ESP32's internal clock is not used:

  * `clk.mode: CLK_EXT_IN` in YAML
  * RMII CLK input is GPIO0
  * GPIO17 is used to enable the oscillator (pull HIGH to power)
  * **Ensure that the oscillator circuit is properly soldered, and leave R16 (0Ω) unpopulated** to avoid conflict
  * See original open hardware reference for details


## 📁 File Structure

```
esp32-ble-proxy-enhanced/
├── README.md
├── LICENSE
├── eth-version.yaml
├── wifi-version.yaml
└── secrets.yaml.example
```

* `eth-version.yaml`: Ethernet connectivity
* `wifi-version.yaml`: Wi-Fi connectivity (with fallback AP)
* `secrets.yaml.example`: Secrets template (copy as `secrets.yaml` and fill in)

> ⚠️ **Note**
> **Wi-Fi and Ethernet cannot be enabled simultaneously** in ESPHome. Please choose only one.


## 🌙 Quick Start

1. **Prepare your environment**

   * Home Assistant (latest stable recommended)
   * Install the latest [ESPHome](https://esphome.io/)

2. **Configure your secrets**

   * Copy `secrets.yaml.example` to `secrets.yaml` and fill in:

     ```yaml
     wifi_ssid: "Your_WiFi_Name"
     wifi_password: "Your_WiFi_Password"
     api_key: "YOUR_ESPHOME_API_KEY"
     ota_password: "YOUR_OTA_PASSWORD"
     ```

3. **Choose your connection method**

   * Ethernet → use `eth-version.yaml`
   * Wi-Fi → use `wifi-version.yaml`

4. **Compile & flash**

   * Use ESPHome Dashboard or CLI to compile and flash the firmware
   * For first time, flash via USB; afterward, you can update via OTA

5. **Connect to Home Assistant**

   * The device will be auto-discovered or appear in the "ESPHome" integration
   * Once added, you can:

     * Trigger automations via the `binary_sensor`
     * Control or test LEDs via `switch.led_*`
     * Extend BLE range with Bluetooth Proxy functionality

6. **Done!**

   * You can view scanned BLE devices from the Bluetooth integration
   * Or monitor the current connection slot status

> 💡 **Tip**  
> Home Assistant's BLE integration automatically manages link balancing and slot allocation.  
> You can deploy multiple BLE proxies around your home for wider BLE coverage.  


## ⭐ LED Status Meaning

| LED    | State | Meaning                         |
|--------|-------|---------------------------------|
| `BLE`  | Blink | BLE broadcast received (50ms)   |
| `BLE`  | On    | Active BLE connection present   |
| `BLE`  | Off   | No broadcast, no active session |
| `NET`  | On    | Connected to Home Assistant API |
| `NET`  | Off   | Disconnected                    |
| `WIFI` | On    | Wi-Fi connected                 |
| `WIFI` | Off   | Wi-Fi disconnected              |

> 💡 **Tip**  
> These states are driven by `globals + interval + on_ble_advertise` for high responsiveness and non-blocking logic ✨


## ❓ FAQ

* **Q: Can I enable both Wi-Fi and Ethernet at the same time?**  
  A: No.

  * ESPHome does not support enabling both at once. Please choose one.

* **Q: How many BLE devices can it support?**  
  A: Unlimited broadcast devices.

  * BLE typically relies on passive advertisement; no slot is needed.  
  * However, active GATT connections (e.g., for querying device state) are limited by available connection slots (typically 3 concurrent).

* **Q: Can I increase the number of connection slots?**  
  A: Yes.

  * You can set `max_connections` in `esp32_ble_tracker`, but be cautious -- memory is limited.  
  * Removing `web_server` or unnecessary features can free memory and allow 6-7 connections.

* **Q: Does it support Classic Bluetooth?**  
  A: No. 
  
  * Only BLE is supported by ESPHome’s Bluetooth Proxy.

* **Q: Does it work with Xiaomi devices?**  
  A: Partially.

  * Some Xiaomi sensors are passively supported via broadcast.  
  * But Xiaomi’s smart devices often require proprietary encrypted communication, so this project cannot replace the Xiaomi gateway for those.

* **Q: Can I build my own BLE device?**  
  A: Definitely!  

  * Use the [BTHome protocol](https://bthome.io/) -- it’s official, simple, and compatible with Home Assistant.

* **Q: Can I actively control BLE devices?**  
  A: Yes, with custom development.  

  * Check out *my other project*: [PVVX Display Controller](https://github.com/Angelic47/pvvx_display)  
  * It adds a BLE service to control the screen of PVVX-based Xiaomi thermometers.


## 📜 License

* This project uses the **MIT License**
* No hardware design files included; all hardware rights belong to original creators
* Not affiliated with the OSHWHub project


## 🛠️ Development & Contribution

Any form of feedback, suggestions, or contributions is warmly welcomed 💛  
Feel free to submit Issues or Pull Requests -- I will do my best to respond in time.
