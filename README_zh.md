# ESP32 BLE Proxy Enhanced

> 基于 ESPHome 的 BLE 代理固件（Bluetooth Proxy），为 Home Assistant 提供即插即用的蓝牙网关体验  
> 开源硬件参考：`esp32-getway`（OSHWHub：https://oshwhub.com/yanqisui/esp32-getway）  

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg?style=for-the-badge)](LICENSE)
[![ESPHome](https://img.shields.io/badge/HomeAssistant-ESPHome-blue.svg?style=for-the-badge)](https://esphome.io/)

(English version available in [README.md](README.md))


## 🕊️ 介绍

这块硬件最早由 Ai-Thinker 于 2022 年发布，原始用途是蓝牙网关开发板。  
其硬件设计优雅，供电稳固、接口齐全，结构也非常适合作为 BLE 网关使用…… 甚至拥有一个漂亮的外壳。  

但随着时间推移，原固件生态早已失修，工具链陈旧，文档稀缺，且不支持 Home Assistant ——  
对如今的智能家居用户而言，几乎等同于“只剩一块优秀却孤独的硬件”。

......所以，让我们将其核心温柔地重塑\~ ✨  
本项目基于 **ESPHome Bluetooth Proxy**，为它注入新的生命，使其：

- ✅ 能以现代方式无缝接入 Home Assistant
- ✅ 提供简洁直观的 **LED 指示灯** 与 **按钮输入**
- ✅ 支持 **有线（以太网）与无线（Wi-Fi）** 二选一连接
- ✅ 利用 Home Assistant 自带的 `habluetooth` 框架，原生支持链路切换、BLE 多设备自动负载均衡

> ⚡ **注意**  
> 本项目为非官方实现，与原始硬件作者无合作关系。**原作者未参与开发，请勿向其索要技术支持**。  
> 如需支持，请前往本项目仓库提出 Issue


## ✨功能特性

- **双版本配置文件**：
  - `eth-version.yaml`：LAN8720 以太网版
  - `wifi-version.yaml`：2.4GHz Wi-Fi 版（含 Fallback AP 热点）
- **BLE 状态指示灯**
  - 接收到 BLE 广播 → 灯亮 50ms
  - 存在主动 GATT 连接 → 常亮
- **网络状态指示灯**
  - `NET`：表示 Home Assistant API 是否连接成功
  - `WIFI`：表示 Wi-Fi 是否连接成功（仅 Wi-Fi 版）
- **按钮输入**
  - GPIO34 接入按钮 → 可用作 Home Assistant 自动化触发器
- **Web 配置面板**
  - 内建 ESPHome 的 `web_server`（可选）


## ⚙️ 硬件与引脚

引脚映射如下：

| 功能         | 引脚     | 说明                     |
|--------------|----------|--------------------------|
| BLE 指示灯    | GPIO15   | 低电平点亮               |
| WIFI 指示灯   | GPIO2    | 低电平点亮（Wi-Fi 版）    |
| ETH 指示灯    | GPIO16   | 低电平点亮（以太网版）    |
| 按钮输入      | GPIO34   | 外部上拉，按下接地         |
| 外部晶振使能  | GPIO17   | 若有外部 50 MHz OSC，需要拉高此脚上电使能 |

### 💡 以太网版注意事项

- 本项目采用 **外部 50 MHz 有源晶振** 为 RMII 时钟源，**未使用 ESP32 自带时钟输出**：
  - YAML 中设置 `clk.mode: CLK_EXT_IN`
  - CLK 输入引脚为 GPIO0
  - GPIO17 被用作外部有源晶振使能引脚（拉高使能）
  - **请焊接开发板的晶振部分电路，并将开发板上的 R16（0Ω）留空**，以避免冲突
  - 详见原始开源硬件项目


## 📁 文件结构

```

esp32-ble-proxy-enhanced/
├── README.md
├── LICENSE
├── eth-version.yaml
├── wifi-version.yaml
└── secrets.yaml.example

````

- `eth-version.yaml`：以太网接入
- `wifi-version.yaml`：Wi-Fi 接入（含 AP 回退）
- `secrets.yaml.example`：密钥配置模板（复制为 `secrets.yaml` 后填写）

> ⚠️ **注意**  
> ESPHome 中 **Wi-Fi 与 Ethernet 不能同时启用**，请根据需要二选一。


## 🌙 快速上手指南

1. **准备环境**
   - Home Assistant（建议使用最新稳定版）
   - 安装最新版本 [ESPHome](https://esphome.io/)

2. **配置密钥**
   - 复制 `secrets.yaml.example` 为 `secrets.yaml`，填写如下字段：
     ```yaml
     wifi_ssid: "Your_WiFi_Name"
     wifi_password: "Your_WiFi_Password"
     api_key: "YOUR_ESPHOME_API_KEY"
     ota_password: "YOUR_OTA_PASSWORD"
     ```

3. **选择接入方式**
   - 有线 → 使用 `eth-version.yaml`
   - 无线 → 使用 `wifi-version.yaml`

4. **编译 & 刷写**
   - 使用 ESPHome Dashboard 或命令行编译并刷写
   - 首次可通过串口（USB）刷入，后续用 OTA 升级

5. **接入 Home Assistant**
   - 刷写完成后设备将自动被发现，或者出现在 Home Assistant 的“ESPHome”集成中
   - 添加后即可：
     - 使用按钮 `binary_sensor` 触发自动化
     - 使用 `switch.led_*` 控制或测试指示灯
     - 使用 BLE Proxy 功能扩展蓝牙接收范围

6. **完成!**
   - 可在 Home Assistant 的蓝牙集成中查看已扫描的设备（广告监视器）
   - 也可以在查看当前设备的连接槽使用状态

> 💡 **小提示**  
> Home Assistant 的蓝牙集成会自动处理链路选择与负载平衡，原生支持多个 ESP32 BLE 代理设备，  
> 如您需要，可以在家中多个位置放置此设备，覆盖更广的 BLE 设备范围。

## ⭐ 指示灯语义说明

| 指示灯 | 状态 | 含义 |
|--------|------|------|
| `BLE`  | 闪烁 | 收到广播包（50ms） |
| `BLE`  | 常亮 | 存在主动连接 |
| `BLE`  | 熄灭 | 无广播、无连接 |
| `NET`  | 常亮 | Home Assistant 已连接（API） |
| `NET`  | 熄灭 | 连接断开 |
| `WIFI` | 常亮 | 成功连接 Wi-Fi |
| `WIFI` | 熄灭 | Wi-Fi 未连接 |

> 💡 **小提示**  
> 这些逻辑由 `globals + interval + on_ble_advertise` 驱动，确保效率高、无阻塞、状态反应快速 ✨


## 常见问题（FAQ）

- **Q：Wi-Fi 与以太网能否同时启用？**  
  A：不可以。
  - ESPHome 的以太网与 Wi-Fi 组件不可并存，您需要二选一。

- **Q：最大 BLE 设备数是多少？**  
  A：理论上可接收无限设备的广播。   
  - BLE 通常采用设备广播、ESP32 被动接收的方式，不受连接槽数量的限制。  
  但是，如果某项操作需要主动连接到设备的 GATT 服务，进行主动控制、状态轮询等，则受限于 ESP32 的连接槽数量（通常为同时并发 3 个）。  

- **Q: 连接槽数量可以修改吗？**  
  A：可以。
  - ESPHome 的 `esp32_ble_tracker` 组件允许通过 `max_connections` 参数修改最大连接槽数量，  
  但请注意，ESP32的内存有限，过多的连接可能导致性能下降或系统不稳定。  
  建议在实际使用中根据需要调整，通常为 3 - 5 个连接槽，删除 Web 服务器等不必要的功能可以释放更多内存，可达到 6 - 7 个连接槽。

- **Q: 支持Classic Bluetooth 吗？**  
  A：不支持。ESPHome 的 BLE 代理功能仅支持 BLE（低功耗蓝牙），不支持 Classic Bluetooth。

- **Q: 是否支持小米设备？**  
  A：理论上支持，
  - Home Assistant 自带了一些小米设备的集成（例如小米温湿度计），  
  但由于小米设备并不开源且存在加密机制，Home Assistant 也缺少对于主动控制小米（例如小米智能开关）设备的支持，  
  因此，该项目暂时无法替代小米网关.  

- **Q: 我可以自己开发自己的 BLE 设备吗？**  
  A：当然可以！
  - 如果您只需要被动广播，请直接使用 [BTHome协议](https://bthome.io/)，  
  该协议是 Home Assistant 官方推荐的 BLE 广播协议，简单易用。  

- **Q：可以做 BLE 主动控制吗？**  
  A：可以，但需要额外开发。
  - 如果您需要主动发起 GATT 请求，请参考*我的另一个项目*：[PVVX显示控制集成](https://github.com/Angelic47/pvvx_display) 。  
  该项目通过创建新的 *服务* ，并且向 BTHome 的设备附加一个 *动作* 的方式，实现了对 PVVX 固件的小米温度计的显示屏自定义内容控制。  


## 📜 许可证

- 项目代码使用 **MIT License**
- 不包含硬件设计文件，原始硬件归各作者所有
- 与 OSHWHub 原项目无官方关联


## 🛠️ 开发与贡献
欢迎任何形式的反馈、建议与贡献 💛  
你可以通过 Issue 或 Pull Request 提交您的创意，我也会尽力给予及时的回应。
