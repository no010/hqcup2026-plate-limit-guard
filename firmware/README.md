# 固件与端侧软件

> 状态：v0.1 尚未实现。本文是开发计划与接口约定，不是可用代码。
> 设计依据：[docs/03-software-design.md](../docs/03-software-design.md) · [数据契约 v2](../docs/07-data-contract-v2.md)

## 运行环境

| 项目 | 选型 |
| --- | --- |
| 硬件 | RDK X5 Module（4GB 起）+ ML307N Cat.1 |
| 系统 | Ubuntu 22.04（地瓜官方镜像） |
| 语言 | Python 3.10（首版）；性能瓶颈再考虑 C++ |
| Python 管理 | `uv` + `pyproject.toml`（计划） |
| 视觉 | OpenCV、V4L2；候选 ONNX Runtime / hobot_dnn / rdk_model_zoo |
| 串口 | pyserial |
| 音频 | ALSA；ML307N AT TTS 或本地 TTS |
| 服务 | systemd |
| 测试 | pytest；规则引擎纯函数测试 |

## 目录规划

```text
firmware/
├─ device/
│  ├─ capture/       # 摄像头采集与重连
│  ├─ vision/        # 车牌检测与字符识别
│  ├─ tracking/      # 多帧去重与冷却
│  ├─ rules/         # 限行规则引擎（纯函数）
│  ├─ data/          # 限行数据拉取、校验、缓存
│  ├─ notify/        # 播报决策与 TTS
│  ├─ modem/         # ML307N AT 状态机
│  ├─ config/        # 配置加载与校验
│  └─ service/       # systemd、看门狗、健康检查
├─ tests/            # 单元与集成测试
├─ tools/            # 开发辅助脚本
├─ config.example.yaml
└─ README.md
```

## 开发顺序

先做不依赖硬件的部分，再做硬件联调：

1. **规则引擎**：纯函数 + 单元测试，覆盖规则域、尾号、窗口端点、覆盖度、通知策略与外埠未知分支。
2. **数据校验**：按 [数据契约 v2](../docs/07-data-contract-v2.md) 实现签名、业务日期、有效期、修订号与覆盖度校验；v1 schema 只作历史参考。
3. **ML307N 冒烟**：串口 `AT` → SIM → 注网 → HTTPS → TTS，逐项记录。
4. **摄像头与识别**：先在开发板上跑通 1080p 取帧与 5fps，再评估模型帧率。
5. **端到端**：识别 → 规则 → 播报 → 日志，单车道实测。
6. **系统化**：systemd 服务、看门狗、日志轮转、配置校验、断网降级。

## 硬件接口约定

| 接口 | 约定 |
| --- | --- |
| 摄像头 | `/dev/video0`，USB 2.0 UVC，1080p MJPEG |
| ML307N | UART，默认 115200 8N1，1.8V 电平经底板转换 |
| 音频 A | ML307N 离线 TTS，AT 指令下发文本 |
| 音频 B | RDK X5 I2S → MAX98357A → 扬声器 |
| 状态灯 | PWR / SYS / LTE / ERR，GPIO 或模组 NET_LED |
| 扩展 | DI 触发、继电器、I2C/UART 扩展座 |

## 质量门禁

| 门禁 | 命令/方式 | 通过标准 |
| --- | --- | --- |
| 静态检查 | `ruff check .` | 0 error |
| 类型检查 | `ty check` | 0 error |
| 单元测试 | `pytest` | 全部通过；规则引擎覆盖边界 |
| 数据校验 | JSON Schema + 合理性检查 | 通过 |
| 集成测试 | 开发板 + 摄像头 + 4G + 扬声器 | 冒烟清单全部通过 |
| 文档 | markdownlint | 0 error |

## 首个可验收目标

- [ ] 规则引擎对 2026-10-27（周二，尾号 1/6）返回正确结果。
- [ ] 07:00/20:00 端点按规则包的包含标志判断。
- [ ] 断网时仅当天有效包可判断；跨日无新包返回未知。
- [ ] ML307N 能拉取 v2 签名包并验签通过。
- [ ] 摄像头 1080p 连续取帧 10 分钟无断流。
- [ ] 识别到符合提醒条件的车辆后 1.5s 内开始播报。
- [ ] 日志默认脱敏，不保存图片。

## 待确认

1. ML307N 子型号与 AT 指令集（PDP/HTTP/TTS）。
2. RDK X5 上可用的车牌识别模型与许可证。
3. 音频路径选择：ML307N TTS 还是 X5 I2S。
4. 摄像头型号与夜间补光方案。
5. systemd 服务名、日志目录、配置目录的最终约定。
