# 共享工具脚本

## 说明

辅助开发、测试、部署的脚本工具。

## 脚本清单

| 脚本 | 功能 | 用法 |
|------|------|------|
| `flash.sh` | 一键烧录固件 | `./flash.sh <elf/bin>` |
| `log_parser.py` | 解析串口日志 | `python log_parser.py <port>` |
| `model_converter.py` | TFLite 模型转换/量化 | `python model_converter.py <model>` |
| `plot_sensor.py` | 实时绘制传感器数据 | `python plot_sensor.py <port>` |
| `generate_bom.py` | 从 KiCad 生成 BOM | `python generate_bom.py <pcb_file>` |

## 环境依赖

```bash
pip install pyserial matplotlib numpy pandas
```
