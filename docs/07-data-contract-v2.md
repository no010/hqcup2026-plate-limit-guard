# 限行数据契约 v2（设计草案）

> 2026-10-27 · 设计草案，未冻结、未实现。本文定义发布方与设备之间的规则数据契约；不修改 v1 schema、`current.json` 或任何固件代码。

## 1. 目标与范围

v2 的目标是让设备在不解析网页、不自行推断节假日的前提下，获得足以做出三态判断的当日规则，并在数据不可信时明确返回未知。

- 显式表达每条规则的适用对象、规则域、时间窗口端点、区域、尾号、caveat 与法律依据。
- 用覆盖声明区分“确认不适用”和“依据不足”。
- 用签名、修订号、业务日期和有效期抵御篡改、重放与回滚。
- 保留来源、人工复核和撤回的审计链。

非目标：执法取证、判断进京证、判断车辆实际路线、其他城市、实时路况、道闸控制。

## 2. 信任模型

- 法律权威：北京市人民政府、北京市公安局公安交通管理局发布的通告。
- 发布方：本仓库的数据发布流程。发布方负责抓取、解析、人工复核、签名和发布，不代表官方背书。
- 设备：只信任内置公钥对应的签名包；不把 CDN、镜像域名或 HTTPS 证书当作内容真实性依据。
- 传输：HTTPS 只保护传输；镜像和仓库发布页都可能被替换或回放，必须验签。
- 时间：设备业务日期以 `Asia/Shanghai` 为准；时钟不可信时不得判断。

签名只证明包由持有私钥的发布方签发且未被篡改，不证明内容与官方通告一致；内容正确性由人工复核、来源哈希和审计负责。

威胁与对策：

| 威胁 | 对策 |
| --- | --- |
| 镜像篡改 | Ed25519 签名 + 载荷规范化 |
| 重放旧包 | `business_date` + `valid_from` / `valid_until` + 修订号 |
| 修订号回退 | 设备保存已接受的最大修订号 |
| 规则域部分覆盖被误读为不限行 | `coverage.completeness` + 不完整域必须返回未知 |
| 端点歧义 | `start_inclusive` / `end_inclusive` / `endpoint_verified` |
| 密钥泄露 | 双密钥重叠、密钥 ID、固件吊销与轮换流程 |
| 设备时钟被回拨 | 时钟可信性检查 + 业务日期匹配 |
| 发布方误发布 | 双人复核、自动校验、撤回包和审计 |

## 3. 包结构与示例

每个城市、每个业务日期发布一个包。包是不可变的：内容变化必须增加 `revision` 并重新签名。

### 3.1 信封

```json
{
  "envelope_version": 2,
  "payload": {},
  "signature": {
    "alg": "Ed25519",
    "key_id": "bj-2026-01",
    "value": "base64url..."
  }
}
```

- 签名对象是 `payload` 的规范化 JSON 字节（RFC 8785 JCS），不包含信封字段。
- `alg` 只允许 `Ed25519`。
- `key_id` 必须存在于设备的固定公钥表中；未知 ID 直接拒绝。
- `value` 使用 base64url 无填充编码。

### 3.2 载荷示例

以下示例对应 2026-10-26（周一）；示例假设端点语义已完成法务核验。未核验时 `endpoint_verified` 必须为 `false`，设备会返回未知。

```json
{
  "schema_version": 2,
  "min_engine_version": 2,
  "city": "beijing",
  "timezone": "Asia/Shanghai",
  "business_date": "2026-10-26",
  "revision": 3,
  "generated_at": "2026-10-25T21:06:00+08:00",
  "valid_from": "2026-10-26T00:00:00+08:00",
  "valid_until": "2026-10-27T00:00:00+08:00",
  "status": "active",
  "withdrawn_reason": null,
  "source": {
    "url": "https://www.beijing.gov.cn/zhengce/zhengcefagui/202603/t20260323_4564097.html",
    "title": "北京市人民政府关于实施工作日高峰时段区域限行交通管理措施的通告",
    "published_at": "2026-03-23",
    "effective_from": "2026-03-30",
    "effective_until": "2027-03-28",
    "retrieved_at": "2026-10-25T21:05:00+08:00",
    "content_sha256": "..."
  },
  "review": {
    "reviewed_by": "release-reviewer-a",
    "approved_by": "release-approver-b",
    "reviewed_at": "2026-10-25T21:08:00+08:00",
    "checklist_version": "2026-10-27",
    "approval_id": "bj-2026-10-26-r3"
  },
  "coverage": [
    {
      "domain": "local_tail",
      "scope": {"vehicle_origin": "local", "plate_types": ["blue"]},
      "time_scope": "full_day",
      "completeness": "complete",
      "unresolved": []
    },
    {
      "domain": "nonlocal_peak",
      "scope": {"vehicle_origin": "nonlocal", "plate_types": ["blue"]},
      "time_scope": "full_day",
      "completeness": "partial",
      "unresolved": ["permit_status", "vehicle_class"]
    },
    {
      "domain": "nonlocal_tail",
      "scope": {"vehicle_origin": "nonlocal", "plate_types": ["blue"]},
      "time_scope": "full_day",
      "completeness": "partial",
      "unresolved": ["permit_status", "vehicle_class"]
    }
  ],
  "rules": [
    {
      "rule_id": "bj-local-tail-2026-10-26",
      "domain": "local_tail",
      "priority": 100,
      "applies_to": {
        "vehicle_origin": "local",
        "plate_types": ["blue"],
        "vehicle_class": "passenger"
      },
      "windows": [
        {
          "start": "07:00",
          "end": "20:00",
          "start_inclusive": true,
          "end_inclusive": false,
          "endpoint_verified": true,
          "area": "inside_5th_ring_excluding"
        }
      ],
      "match": {"tail_numbers": ["5", "0"]},
      "effect": "restricted",
      "caveats": ["exemptions_not_device_verifiable", "vehicle_class_assumed"],
      "exemptions": [
        {
          "code": "emergency_and_service_vehicles",
          "description": "通告列明的豁免车辆",
          "device_verifiable": false,
          "handling": "caveat"
        }
      ],
      "legal_basis": {
        "source_url": "https://www.beijing.gov.cn/zhengce/zhengcefagui/202603/t20260323_4564097.html",
        "clause": "限行时间与轮换表"
      }
    }
  ]
}
```

注意：示例中的 `reviewed_by` 是角色标识，不是个人信息；真实发布时使用发布系统的审批 ID。

## 4. 字段定义

### 4.1 载荷顶层

| 字段 | 类型 | 必填 | 约束 |
| --- | --- | --- | --- |
| `schema_version` | integer | 是 | 固定为 `2` |
| `min_engine_version` | integer | 是 | 设备规则引擎低于此值时必须返回未知 |
| `city` | string | 是 | 首版固定 `beijing` |
| `timezone` | string | 是 | 固定 `Asia/Shanghai` |
| `business_date` | string | 是 | `YYYY-MM-DD`，本地业务日期 |
| `revision` | integer | 是 | 正整数；同一 `city` + `business_date` 单调递增，从 1 开始 |
| `generated_at` | string | 是 | ISO 8601，带时区 |
| `valid_from` | string | 是 | ISO 8601，带时区；不晚于业务日期 00:00 |
| `valid_until` | string | 是 | ISO 8601，带时区；不早于业务日期次日 00:00 |
| `status` | string | 是 | `active` 或 `withdrawn` |
| `withdrawn_reason` | string/null | 是 | `withdrawn` 时必填 |
| `source` | object | 是 | 官方来源、抓取时间与内容哈希 |
| `review` | object | 是 | 人工复核与批准记录 |
| `coverage` | array | 是 | 每个规则域的覆盖声明；`active` 时至少 1 项，`withdrawn` 时可为空 |
| `rules` | array | 是 | 规则列表；不限行日或 `withdrawn` 时为空数组 |

`source` 字段：

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `url` | string | 是 | 官方通告 URL |
| `title` | string | 是 | 通告标题 |
| `published_at` | string | 是 | 官方发布日期 |
| `effective_from` | string | 是 | 通告生效日期 |
| `effective_until` | string | 是 | 通告失效日期 |
| `retrieved_at` | string | 是 | 实际抓取时间 |
| `content_sha256` | string | 是 | 抓取原文的 SHA-256，用于审计 |

`review` 字段：

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `reviewed_by` | string | 是 | 复核角色或审批人 ID，不写个人敏感信息 |
| `approved_by` | string | 是 | 第二复核/批准人，不能与 `reviewed_by` 相同 |
| `reviewed_at` | string | 是 | 复核时间 |
| `checklist_version` | string | 是 | 复核清单版本 |
| `approval_id` | string | 是 | 发布审批单号，用于追溯 |

### 4.2 覆盖声明

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `domain` | string | `local_tail`、`nonlocal_peak`、`nonlocal_tail` |
| `scope` | object | `vehicle_origin` 与 `plate_types`；只覆盖列出的对象 |
| `time_scope` | string | 首版固定 `full_day` |
| `completeness` | string | `complete` 或 `partial` |
| `unresolved` | string[] | `partial` 时必须非空，列出无法确认的条件代码 |

`scope` 字段：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `vehicle_origin` | string | `local` 或 `nonlocal` |
| `plate_types` | string[] | `blue`、`green`、`yellow` 的非空子集 |

规则域覆盖语义：

- `complete` 表示该域在声明范围内、整个业务日期内，所有影响判断的条件都已表达；设备可以在没有规则命中时返回“不适用”。
- `partial` 表示至少一个条件未确认；该域返回未知，不能返回“适用”或“不适用”。发布方若希望某类车辆可判断，应拆出独立的 `complete` scope。
- 没有匹配的覆盖声明（例如绿牌没有对应 scope）时，返回未知，不得按“没有规则”推断不适用。
- 多个覆盖声明同时匹配且完整度或条件冲突时，返回未知。
- `unresolved` 使用封闭词汇表；设备遇到未知代码时返回未知，不能忽略。

首版条件代码：`permit_status`（进京证）、`vehicle_class`（载客/载货等）、`green_plate_exemption`（绿牌豁免）、`window_endpoint_semantics`（端点语义）、`temporary_adjustment`（临时调整）。

### 4.3 规则

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `rule_id` | string | 是 | 稳定标识；同一业务日期内唯一 |
| `domain` | string | 是 | 必须与某个覆盖声明的 `domain` 一致 |
| `priority` | integer | 是 | 发布方冲突检测用；设备不据此选择规则 |
| `applies_to` | object | 是 | 车辆归属、号牌类型、车辆类别 |
| `windows` | array | 是 | 至少 1 个时间窗口；只决定通知时间，不参与日级适用性匹配 |
| `match` | object | 是 | 首版为尾号匹配 |
| `effect` | string | 是 | 首版固定 `restricted`；其他值使整包无效 |
| `caveats` | string[] | 是 | 不阻断判断、但播报必须提示的条件代码 |
| `exemptions` | array | 是 | 豁免说明与设备处理方式；可为空 |
| `legal_basis` | object | 是 | 官方来源与条款 |

`applies_to` 字段：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `vehicle_origin` | string | `local` 或 `nonlocal` |
| `plate_types` | string[] | `blue`、`green`、`yellow` 的非空子集 |
| `vehicle_class` | string | `passenger`；由发布方声明，设备不核验，命中时必须带 `vehicle_class_assumed` caveat |

`windows` 字段：

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `start` | string | 是 | `HH:MM`；不允许 `24:00` |
| `end` | string | 是 | `HH:MM`；允许 `24:00` 表示当日结束 |
| `start_inclusive` | boolean | 是 | 起点是否包含 |
| `end_inclusive` | boolean | 是 | 终点是否包含 |
| `endpoint_verified` | boolean | 是 | 端点语义是否已法务核验；`false` 时该规则不可用于判断 |
| `area` | string | 是 | `inside_5th_ring_excluding`、`inside_5th_ring_including`、`all_city` |

`match` 字段：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `tail_numbers` | string[] | 命中尾号，元素为唯一 `"0"`–`"9"`；空数组表示匹配全部尾号 |

`exemptions` 字段：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `code` | string | 豁免代码 |
| `description` | string | 说明 |
| `device_verifiable` | boolean | 首版固定 `false` |
| `handling` | string | `caveat`（继续判断并提示核对）或 `unknown`（返回未知） |

`legal_basis` 字段：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `source_url` | string | 官方通告 URL |
| `clause` | string | 规则所在条款或章节 |

首版对本地蓝牌尾号规则采用“passenger 假设 + caveat”，以便对核心场景给出提醒；外埠、黄牌和绿牌不采用该假设，相关条件未确认即为未知。

规则冲突包括：重复 `rule_id` 且内容不同；同一规则内窗口重叠；规则对象不在覆盖 scope 内；同一对象在同一时间被要求落在不同 `area`。出现冲突则整包无效。

`caveats` 与 `unresolved` 的区别：

- `caveat` 不改变规则是否适用，但必须在播报中提示用户核对；例如设备无法核验的豁免。
- `unresolved` 改变规则是否适用；只要出现在匹配的覆盖声明中，就必须返回未知。
- `caveats` 使用封闭词汇表，首版允许 `exemptions_not_device_verifiable` 与 `vehicle_class_assumed`。规则声明 `vehicle_class` 或含不可核验豁免时，发布方必须写入对应 caveat，否则包无效。

## 5. 判断语义

设备按以下顺序得到一个三态结果：

1. 校验包（见第 7 节）。任一项失败：未知。
2. 规范化车牌与尾号：取最后一位字符，数字直接用，字母按 0；无法解析：未知（识别失败）。
3. 确定车辆属性：`vehicle_origin`、`plate_type`；任一项未知：未知。
4. 找到匹配的覆盖声明：没有匹配、多个匹配冲突或 `partial`：未知。
   - `partial` 的 `unresolved` 条件即使看似与当前车辆无关，设备也不做主观推断；发布方应拆出更精确的 `complete` scope。
5. 在该域内匹配规则（只看 `applies_to` 与 `match`，不看当前是否在窗口内）：
   - 任一规则的适用对象与尾号条件满足：结果为“适用”；合并 `caveats`；若命中的 `exemptions` 中存在 `handling=unknown`，结果为未知。
   - 多条规则命中：合并 caveats；`priority` 只用于发布方冲突检查，不改变设备判定。
   - 包中出现未支持的 `effect` 值：整包无效，返回未知。
   - 没有规则命中：`complete` 返回“不适用”；`partial` 返回未知。
6. 时间窗口：
   - `status` 表示规则在业务日期内是否适用于该车辆，不因当前时刻在窗口外而改变。
   - 当前时刻只用于上层通知决策：窗口内为当前提醒，窗口开始前为预提醒，所有窗口结束后不播报。
   - `endpoint_verified=false`：该规则不可用，返回未知。
7. 区域判断：
   - 设备配置的安装区域必须能映射到 `area`；无法映射：未知。
   - 设备在区域外：若覆盖为 `full_day` 且 `complete`，返回“不适用”；否则未知。
8. 通知决策不在本契约内，由软件设计定义；本契约只提供三态结果、窗口和 caveats。

“适用”只表示规则在当日适用于该车辆，不表示当前正在限行，也不表示车辆必然违法；播报必须使用核对措辞，不输出执法结论。

## 6. 时间、区域与对象

- 业务日期固定为 `Asia/Shanghai` 的日历日期；不以 UTC 日期或设备启动时间推断。
- `valid_from` 必须不晚于业务日期 00:00，`valid_until` 必须不早于业务日期次日 00:00；否则该包不得声明 `full_day` 覆盖，按无效包处理。
- 端点包含性由 `start_inclusive` / `end_inclusive` 显式表达；官方原文“7时至20时”不自动等价于任一端点。
- `area` 使用封闭枚举；设备安装区域由配置提供，不从车牌或 GPS 推断。
- 车辆归属和号牌类型来自识别结果；识别不确定时返回未知。
- `vehicle_class` 首版按发布方声明和设备 caveat 处理，不通过摄像头推断载客/载货。

## 7. 设备校验顺序

设备必须在一次完整校验通过后才替换可用包；任何一步失败都保留旧包，但旧包只有在自身再次通过校验且业务日期匹配时才能用于判断。

1. 解析信封与 JSON，限制最大尺寸；未知字段拒绝。
2. 校验 `envelope_version=2`、`signature.alg=Ed25519`、`key_id` 已知。
3. 规范化 `payload` 并验签；失败拒绝。
4. 校验 `schema_version=2`、`min_engine_version` 不超过本机引擎版本。
5. 校验 `city=beijing`、`timezone=Asia/Shanghai`。
6. 校验设备时钟可信；不可信返回未知，不尝试“猜测日期”。
7. 校验 `business_date` 等于本地业务日期，且在 `source.effective_from` 至 `source.effective_until` 之间。
8. 校验 `valid_from <= now < valid_until`。
9. 校验 `revision` 大于已接受的最大修订号；相同修订号仅接受完全相同的载荷哈希。
10. 校验 `status`：`withdrawn` 时停止使用该业务日期的规则并返回未知；已接受 `withdrawn` 后，该业务日期不再接受任何 `active` 包。
11. 校验覆盖声明与规则：域一致、scope 合法、规则对象在 scope 内、`partial` 必须有 `unresolved`、同一规则内窗口不重叠、无规则冲突、尾号合法。`endpoint_verified=false` 的规则必须使对应 coverage 为 `partial` 且包含 `window_endpoint_semantics`；设备不使用该规则判断。
12. 原子写入：临时文件 + 校验哈希 + `fsync` + 重命名；失败保留旧包。包、修订号和哈希一起持久化；修订号状态损坏时按未知处理，不猜测。

校验失败原因码（用于日志和测试）：`SIGNATURE_INVALID`、`SCHEMA_INVALID`、`UNKNOWN_FIELD`、`UNSUPPORTED_VERSION`、`CITY_MISMATCH`、`CLOCK_UNTRUSTED`、`BUSINESS_DATE_MISMATCH`、`NOT_YET_VALID`、`EXPIRED`、`REVISION_ROLLBACK`、`WITHDRAWN`、`COVERAGE_INVALID`、`RULE_CONFLICT`、`ENDPOINT_UNVERIFIED`、`STORAGE_WRITE_FAILED`。

## 8. 签名、密钥与发布

### 8.1 密钥

- 设备固件内置至少两个 Ed25519 公钥及其 `key_id`，支持重叠轮换。
- 私钥不进入通用 CI；签名在隔离环境或硬件安全模块中完成。
- 轮换：发布新 `key_id` 的包前，先通过固件更新把新公钥下发到设备；旧密钥在重叠期后停用。
- 吊销：密钥泄露时发布固件更新移除该公钥，并通过运维通知；离线设备在更新前仍可能接受旧密钥签名的包，这是已知残余风险。
- `key_id` 未知、签名算法不支持或签名不匹配一律拒绝。
- 设备存储被清空后，重放仍在有效期内的旧包无法仅靠修订号检测；影响限于当天已签名内容，需通过安全存储或运维审计降低。

### 8.2 发布流程

1. 定时抓取官方通告，保存原文、URL、抓取时间和 SHA-256。
2. 解析为 v2 草稿；解析失败不发布猜测值。
3. 自动校验：Schema、日期、尾号、窗口、端点标志、覆盖度、规则冲突、来源哈希。
4. 人工复核：逐日核对业务日期、尾号、窗口、适用对象和豁免；双人复核后批准。
5. 在隔离环境签名，生成信封、发布清单和审计记录。
6. 发布不可变包到仓库发布页和镜像；更新指针文件时保留旧版本。
7. 监控设备拉取与验签失败率；异常时撤回或修正。

### 8.3 撤回与修正

- 修正（撤回前）：发布更高 `revision` 的 `active` 包；设备按修订号替换。
- 撤回：发布更高 `revision` 的 `withdrawn` 包，填写 `withdrawn_reason`；设备停止使用该业务日期的规则并返回未知，不得回退到旧修订号。`withdrawn` 是该业务日期的终态，之后不再发布同日期的 `active` 包；如需人工恢复，属于运维例外，必须重新评估并记录审批。
- 不删除历史包；审计链保留至少一个轮换周期。

## 9. 与 v1 的关系

- v1 的 `limit-day.schema.json` 和 `current.json` 保留为历史草案，不用于设备判断。
- v2 不从 v1 自动转换；发布方重新生成 v2 包并人工复核。
- 只支持 v1 的固件不得把 v1 数据当作 v2 的降级替代；无法验签或版本不匹配时返回未知。
- v2 实现后，仓库可保留 v1 文件用于追溯，但 README、数据说明和设备配置必须明确 v2 是唯一判断来源。

## 10. 测试要求

至少覆盖：

- 正常包：签名、日期、有效期、覆盖度、规则命中与不命中。
- 篡改：修改载荷、签名、`key_id`、`revision`、`business_date`、`valid_until`、`source.effective_until`。
- 重放：旧 `revision`、昨日包、未来包、跨日包。
- 覆盖：`partial` 不得返回不适用；无匹配 scope 返回未知；多 scope 冲突返回未知。
- 端点：`endpoint_verified=false` 返回未知；包含/不包含四种组合。
- 规则：多条命中、重复 `rule_id`、窗口/对象矛盾、`priority` 相同/不同、尾号空数组、字母尾号。
- 时钟：不可信、回拨、无 RTC、NTP 跳变。
- 存储：断电写入、半包、哈希不匹配、旧包保留。
- 撤回：`withdrawn` 后停止判断，不回退。
- 密钥：未知 `key_id`、轮换重叠、吊销后的固件行为。

## 11. 待决事项

1. 签名私钥的保管方式（HSM、离线机器、双人控制）和发布审批人。
2. 官方窗口端点（20:00 是否包含）及绿牌、外埠豁免范围的法务核验责任人。
3. 外埠进京证、车辆类别是否进入首版规则域，还是保持 `partial` 并返回未知。
4. 是否需要发布未来多日预览包；若需要，预览包不得替代当日包。
5. 密钥吊销在离线设备上的最大可接受窗口。
6. 规则域词汇表和条件代码的扩展流程。
