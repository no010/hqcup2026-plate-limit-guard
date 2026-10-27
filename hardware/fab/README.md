# 制造文件

> 状态：待生成。当前目录只有说明，没有 Gerber/BOM。

## 发布内容

每次打样或发布时，在此目录下建立版本子目录：

```text
fab/
└─ v0.1-20261109/
   ├─ gerber/PLG-v0.1-gerber.zip
   ├─ drill/PLG-v0.1-drill.zip
   ├─ PLG-v0.1-BOM.xlsx
   ├─ PLG-v0.1-CPL.csv
   ├─ PLG-v0.1-assembly.pdf
   ├─ PLG-v0.1-schematic.pdf
   └─ README.md
```

## 检查清单

- [ ] Gerber 层名与华秋 CAM 兼容；钻孔文件单独导出。
- [ ] BOM 含位号、型号、封装、数量、替代料、供应商料号。
- [ ] 坐标文件含顶层/底层，单位与旋转角正确。
- [ ] 原理图 PDF 清晰，含标题栏与版本。
- [ ] 装配图标注极性器件、连接器方向。
- [ ] 与仓库中的 KiCad 工程版本一致。
- [ ] 打样订单号、打样券金额记录在版本 README。
- [ ] 不包含厂商保密资料或未授权文件。

## 许可

硬件制造文件按 CERN-OHL-S-2.0 发布；引用或再制造请保留出处与许可证。
