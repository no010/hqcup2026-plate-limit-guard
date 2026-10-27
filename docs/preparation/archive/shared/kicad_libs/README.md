# 共享 KiCad 库（华秋版）

## 使用说明

1. 安装 KiCad 华秋版：<https://www.eda.cn/>
2. 将本目录添加为项目符号库/封装库路径
3. 优先使用华秋库中已有的 symbols 和 footprints

## 目录结构

```text
kicad_libs/
├── symbols/        # 自定义符号库
├── footprints/     # 自定义封装库
├── 3d_models/      # 3D 模型
└── templates/      # 项目模板
```text
## 常用元器件清单

| 器件 | 封装 | 备注 |
|------|------|------|
| USB-C 16P | USB-C-16P | 电源+USB2.0 |
| SPX3819 | SOT-23-5 | 3.3V LDO |
| AMS1117 | SOT-223 | 5V/3.3V LDO |
| MPU6050 | QFN-24 | 6轴加速度计 |
| ADXL355 | LCC-14 | 低噪声加速度计 |
| WS2812B | 5050 | RGB LED |
| EC11 | 带柄编码器 | 旋转输入 |
| 麦克风 | MEMS-2.75mm | 数字/模拟麦克风 |

## 设计规则

- 两层板：最小线宽/间距 5mil（建议 6mil）
- 四层板：最小线宽/间距 4mil
- 过孔：外径 0.6mm，内径 0.3mm（通用）
- 丝印：字号 ≥ 0.8mm，避免焊盘上方
