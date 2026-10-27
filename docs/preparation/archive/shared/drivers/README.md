# 共享驱动代码

## 说明

各项目通用的传感器/模块驱动，统一放在此处，具体项目通过软链接或复制使用。

## 已有驱动清单

| 器件 | 接口 | 状态 | 路径 |
|------|------|------|------|
| SSD1306 OLED | I2C/SPI | 待移植 | `display/ssd1306/` |
| ST7789 LCD | SPI | 待移植 | `display/st7789/` |
| MPU6050 | I2C | 待移植 | `sensor/mpu6050/` |
| ADXL355 | SPI | 待移植 | `sensor/adxl355/` |
| INA219 电流 | I2C | 待移植 | `sensor/ina219/` |
| SHT30 温湿度 | I2C | 待移植 | `sensor/sht30/` |
| MAX9814 麦克风 | 模拟 | 待移植 | `audio/max9814/` |
| PAM8302 功放 | I2S/PWM | 待移植 | `audio/pam8302/` |
| EC11 编码器 | GPIO | 待移植 | `input/ec11/` |
| ML307N 4G | UART | 待移植 | `comm/ml307n/` |

## 编码规范

- 接口统一使用 `xxx_init()`, `xxx_read()`, `xxx_write()`, `xxx_deinit()`
- 返回码：0 = 成功，负值 = 错误码
- 所有驱动提供 RT-Thread 设备框架适配层（可选）
