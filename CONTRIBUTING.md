# 参与贡献

欢迎 Issue、Pull Request 和复现反馈。这是一个参赛作品，维护者响应可能不及时，但会认真看每一条。

## 可以贡献什么

- 车牌识别模型、预处理、夜间补光的实测数据。
- 北京限行规则的边界案例与官方依据。
- ML307N AT 指令、TTS、HTTP 的调试经验。
- RDK X5 上的性能优化与 BPU 模型转换经验。
- 硬件设计评审：电源、4G、音频、EMC、DFM。
- 文档、复现步骤、错别字。

## 工作流

1. 先开 Issue 描述问题或建议，避免大改动白做。
2. 从 `main` 拉分支：`feat/...`、`fix/...`、`docs/...`。
3. 小步提交，说明“为什么”而不是只写“改了什么”。
4. PR 描述写清变更、验证方式、影响范围。
5. 硬件变更必须附 ERC/DRC 结果；数据变更必须附 Schema 校验结果。

## PR 检查清单

- [ ] 文档通过 markdownlint。
- [ ] JSON 文件语法正确；数据变更符合 Schema。
- [ ] Python 代码通过 ruff、ty、pytest。
- [ ] 硬件变更附 ERC/DRC 结果与 BOM 更新。
- [ ] 没有提交真实车牌、人脸、密钥、WiFi 密码、个人地址。
- [ ] 第三方代码/模型/图片的许可证已核验并记录。
- [ ] 更新了相关文档与变更记录。

## 许可证

- 硬件设计：CERN-OHL-S-2.0。
- 固件与软件：Apache-2.0。
- 文档与图片：CC BY 4.0。

提交 PR 即表示你同意按对应许可证授权你的贡献。引用第三方内容时，请保留原许可证与出处。

## 隐私与安全

- 测试图片必须打码，不得包含真实车牌、人脸、门牌号。
- 不要在公开 Issue 中粘贴密钥、SIM ICCID、设备序列号、家庭地址。
- 发现安全问题时，请先私下联系维护者，不要在公开 Issue 贴利用细节。

## 提交信息

建议使用 Conventional Commits：

```text
feat(rules): support nonlocal peak-hour ban
fix(modem): retry PDP activation after network loss
docs(hardware): freeze ML307N power sequence
```

中文提交也可以，保持前缀与范围清晰即可。对事不对人，欢迎直接指出设计问题。
