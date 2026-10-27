# 硬件设计

> 状态：v0.1 空白工程骨架，尚未开始原理图与布线。
> 规格依据：[docs/02-hardware-spec.md](../docs/02-hardware-spec.md)

## 目录

| 路径 | 内容 |
| --- | --- |
| `kicad/` | KiCad 工程：原理图、PCB、工程设置 |
| `kicad/README.md` | 工程约定、版本兼容、ERC/DRC 门禁 |
| `fab/` | Gerber、钻孔、BOM、坐标文件、装配图（发布时提交） |

## 设计流程

1. **资料齐套**：RDK X5 Module Datasheet、Hardware Design Guide、Pinout、Carrier Board V1P1 参考设计；ML307N 硬件手册与 AT 手册。
2. **规格冻结**：完成 `docs/02-hardware-spec.md` 的待确认清单，锁定连接器、电源、音频路径、板框。
3. **原理图**：按功能分页；每页完成后跑 ERC；关键网络留测试点。
4. **评审**：电源、4G、音频、接口、保护五类交叉评审；记录问题与结论。
5. **PCB**：4 层板；先摆接口与模块，再布电源与射频；跑 DRC 与华秋 DFM。
6. **打样**：导出 Gerber、BOM、坐标文件；使用大赛打样券；保留订单记录。
7. **回板调试**：先测电源轨，再测 4G、音频、摄像头；记录测试数据并回写文档。

## 命名约定

- 工程名：`plate-limit-guard`
- 网络前缀：`5V_X5`、`5V_PERIPH`、`3V8_LTE`、`3V3`、`1V8`、`LTE_`、`AUD_`、`CAM_`
- 功能页：`01_power`、`02_rdk_x5`、`03_lte`、`04_audio`、`05_io`
- 版本：`v0.1-draft` → `v0.2-draft` → `v1.0`（打样冻结）

## 质量门禁

| 阶段 | 门禁 | 证据 |
| --- | --- | --- |
| 原理图 | ERC 0 error；warning 逐条说明 | `kicad-cli sch erc` 输出 |
| PCB | DRC 0 error；未连接网络为 0 | `kicad-cli pcb drc` 输出 |
| 制造 | 华秋 DFM 无阻断项 | DFM 报告截图 |
| 开源 | 原理图、PCB、Gerber、BOM 齐全 | 仓库文件清单 |
| 合规 | 第三方库/参考设计许可证已记录 | `docs/05-competition-compliance.md` |

## 版本说明

工程以 **KiCad 8 格式**初始化，官方 KiCad 8/9/10 均可打开；首次保存时如提示升级，按项目统一版本另存即可。大赛推荐的 KiCad 华秋版为可选工具，不强制使用。
