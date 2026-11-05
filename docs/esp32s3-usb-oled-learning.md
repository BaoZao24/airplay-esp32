# ESP32-S3 AirPlay USB 声卡与 OLED 学习记录

这个分支保存 ESP32-S3、TOPPING E1x2 OTG USB 声卡与 SSD1306 I2C OLED 的实际使用配置。AirPlay 1 经 Wi-Fi 接收音频，由 ESP32-S3 的 USB Host 输出至声卡；OLED 显示设备与播放信息。

## 接线

- ESP32-S3 的 COM 口连接电脑，用于供电、烧录和串口日志；板载 USB/OTG 口连接声卡 OTG 数据口。声卡按其要求另外供电。
- SSD1306 OLED：VCC 接 3.3 V，GND 接 GND，SDA 接 GPIO8，SCL 接 GPIO9；默认 I2C 地址为 `0x3C`。
- 使用其他开发板前，请先检查 GPIO 和供电接线是否适用。

## 构建

克隆时同时获取项目依赖的子模块：

```sh
git clone --recurse-submodules -b esp32s3-usb-oled https://github.com/BaoZao24/airplay-esp32.git
cd airplay-esp32
pio run -e esp32s3-e1x2otg-oled
```

`esp32s3-e1x2otg-oled` 为带 OLED 的构建环境；不接 OLED 可使用 `esp32s3-e1x2otg`。环境设置可见 `platformio.ini` 与 `config/`。固件和 SPIFFS 资源烧录方法见项目原有文档；首次使用需自行配网。本分支不保存 Wi-Fi 密码。

## 本次改动和验证

- 修正 USB 音频 PCM 缓冲区按完整音频帧填充及读取，避免 24 位立体声数据错位产生持续噪声。
- 接收异步 USB 声卡的显式时钟反馈，调整发送的数据包大小，缓解时钟偏差引起的偶发爆音。
- 为通用 ESP32-S3 开发板增加 I2C 显示总线初始化，并添加可选 OLED 配置。
- 直连声卡的配置关闭 USB Hub 支持，用于避开当前设备与 ESP-IDF 5.5.3 组合中出现的探测问题；使用其他声卡或集线器时应重新评估。

这套硬件已完成 AirPlay 1、USB 声卡与 OLED 联调。48 kHz、24 位是当时声卡的 USB 输出格式；AirPlay 1 的 44.1 kHz 音频在此配置下经过重采样，并非端到端逐位直通。其他声卡和异常场景需要另行测试。
