# KiCad 工程说明

## 当前状态

- `plate-limit-guard.kicad_pro`：工程设置（默认网络类、层叠、DRC 规则待补）。
- `plate-limit-guard.kicad_sch`：空白原理图，含标题栏与 TODO 说明。
- `plate-limit-guard.kicad_pcb`：空白 PCB，含 120×80mm 占位板框。

**这是骨架，不是设计。** 在 [docs/02-hardware-spec.md](../../docs/02-hardware-spec.md) 的待确认清单关闭前，不要开始布线。

## 版本兼容

现有工程以 KiCad 8 格式初始化。项目选定的设计工具为**官方 KiCad 10**；能否直接打开、是否提示升级以及保存后的兼容性均待验证。本文仅进行设计审查，不打开、转换或升级工程文件。项目后续应统一主版本，并在转换前保留可追溯的原始版本。

## 计划的功能页

| 页 | 内容 | 关键网络 |
| --- | --- | --- |
| `01_power` | 12V 输入、5V/3.8V/3.3V/1.8V、保护、电源指示 | `12V`、`5V_X5`、`3V8_LTE`、`3V3`、`1V8` |
| `02_rdk_x5` | 板对板连接器、SD 卡、RTC、风扇、USB、I2S、UART | `USB_`、`SD_`、`I2S_`、`UART_` |
| `03_lte` | ML307N、SIM、天线、PWRKEY/RESET、电平转换 | `LTE_`、`SIM_` |
| `04_audio` | ML307N TTS 路径、X5 I2S 路径、功放、扬声器 | `AUD_` |
| `05_io` | LED、按键、扩展座、DI/继电器、测试点 | `GPIO_`、`DI_`、`RLY_` |

## 库策略

- 优先使用 KiCad 官方符号库与封装库；自定义符号/封装放在工程内 `libs/`（待建），并注明来源与许可证。
- 华秋版元件库可选；如使用，记录库版本与料号，避免封装与实物不一致。
- 不直接把厂商参考设计的工程文件复制进仓库；只参考接口与电源设计，最终原理图自研。

## ERC / DRC

KiCad 8+ 命令行示例：

```bash
kicad-cli sch erc plate-limit-guard.kicad_sch -o erc.rpt
kicad-cli pcb drc plate-limit-guard.kicad_pcb -o drc.rpt
```

门禁：

- ERC：0 error；warning 必须逐条说明（如未连接引脚是故意预留）。
- DRC：0 error；未连接网络 0；差分对/阻抗规则按规格配置。
- 生成 Gerber 前再次跑 DRC，并做华秋 DFM。

## 提交前检查

- [ ] 标题栏：项目名、版本、日期、贡献者标识（不写真实姓名）、许可证。
- [ ] 板框与安装孔符合结构件。
- [ ] 所有接口有丝印名称与极性。
- [ ] 电源轨、UART、USB、音频、PWRKEY/RESET 有测试点。
- [ ] 天线 keep-out、SIM 走线、USB 差分、电源环路按规格检查。
- [ ] BOM 与原理图一致；关键料有替代料。
- [ ] Gerber、钻孔、BOM、坐标文件导出到 `../fab/`。
