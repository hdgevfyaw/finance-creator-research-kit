# 采集与适配器契约（本项目自己的示例，不是第三方API文档）

**选配，仅供以后自行接入工具。** 第一次手工研究不需要实现本契约、不需要 API 或模型；直接用 [一批研究单](FIRST_BATCH_TEMPLATE.md)。下列字段不是要求新手逐项建数据库，示例占位值也不是可直接调用的服务。

## 1. 最小记录

| 字段 | 约束 |
|---|---|
| source_id / canonical_url / publisher / author_identity_status | 稳定ID优先；不靠标题认身份；规范化保留有语义的查询参数 |
| published_at_raw / published_at / timezone_status | 保留原文；规范时间可空；未知时区不能假定北京时间 |
| first_verified_public_at / recorded_at / fetched_at_utc / as_of | 首次可证实公开、录制、实际取得、分析截止分开；使用ISO 8601时间；每种时间另带状态与依据，未知留空并说明，文章录制可不适用 |
| source_type / record_layer / evidence_kind / access_status | 原片/公告/报表等；record_layer 用通用 L1/L2/L3 归属，evidence_kind 区分作者/独立事实/研究推导材料；访问状态为成功/部分/失败/未尝试 |
| locator / quote_short / paraphrase | 页码、段落或秒数；短摘与转述分开；不存版权全文到公开仓库 |
| event_period / market_trade_date / currency / unit | 事件和报告期不等于发布日；金额单位必须显式 |
| sha256 / content_version / parser_version | 只有合法取得实际字节后才填哈希；哈希不证明真实性或许可 |
| claim / scope / conditions / counterevidence / unknowns | 条件词、对象、期限与反证不可省略 |
| rights_basis / redistribution_allowed | 默认未知/否；可访问不自动变成许可 |

**纯合成输入示例（不代表任何真实人物、市场值或检查结果）：**

```json
{
  "source_id": "SYNTHETIC-example-001",
  "canonical_url": "https://example.org/public-item",
  "publisher": "<publisher>",
  "author_identity_status": "unverified",
  "published_at_raw": null,
  "published_at": null,
  "timezone_status": "unknown",
  "first_verified_public_at": null,
  "recorded_at": null,
  "time_status": {"first_verified_public_at": "unknown", "recorded_at": "unknown", "fetched_at_utc": "unknown"},
  "time_basis": "尚未取得材料；时间待核",
  "fetched_at_utc": null,
  "as_of": "<ISO-8601-cutoff>",
  "access_status": "not_attempted",
  "record_layer": "L1",
  "evidence_kind": "author_claim",
  "locator": {"page": null, "start_seconds": null, "end_seconds": null},
  "quote_short": null,
  "claim": "<待核实观点>",
  "conditions": [],
  "counterevidence": [],
  "unknowns": ["尚未取得原始材料"],
  "sha256": null,
  "rights_basis": "unknown",
  "redistribution_allowed": false
}
```

行情记录另加 `instrument_id / exchange / trade_date / session / close / return_type / adjustment / constituent_as_of / valuation_basis`。缺一关键口径不可自动横比。公司事件另加 `event_id / share_class / stage / stage_effective_at / announcement_url / amount_or_per_share / tax_basis`。

## 2. 统一适配器边界

以下是**自定义抽象**，不声称网络上存在这些服务：

- `collect(source, cutoff) → records + access_observation + coverage + errors`：只读；不成功也返回可审计状态。
- `extract(record, schema_version) → proposed_claims + locators + unknowns`：可用本地模型或经批准的供应商；输出是待审建议。
- `verify(proposed_claim, independent_sources) → supported / contradicted / unresolved`：记录理由与核验人，不靠模型置信度替代证据。
- `publish(reviewed_cards) → metadata_only_bundle`：只包含有权公开的内容；证据与归纳可反向追踪。

配置只给占位：`MODEL_PROVIDER=<provider>`、`MODEL_NAME=<model>`、`MODEL_ENDPOINT=<documented-endpoint-if-needed>`、`MODEL_CREDENTIAL_REF=<secret-store-reference>`、`PRIVATE_STORAGE_DIR=./data-private`。不要把真实凭证、账户或原件放进仓库；模型访问接口必须按所选供应商当前官方文档实现，这里不编造请求地址或字段。

## 3. 已使用过的公开地址：观察而非稳定API承诺

完整实际地址与日期见 `SOURCES.csv` 的CSI-PERF、SZSE-HISTORY、HSI-HISTORY、HSTECH-HISTORY。它们来自2026-10-25既有公开源采集日志，不是推测出来的端点；未获得稳定性、批量许可、SLA或全历史覆盖保证。

| 来源ID | 旧采集器观察到的响应结构 | 迁移时必须重新验证 |
|---|---|---|
| CSI-PERF | `data[]`中`tradeDate / close / changePct` | 空数组、日期排序、币种、指数代码、百分比单位；不要直接选最新行忽略截止日 |
| SZSE-HISTORY | `data.picupdata`二维数组；旧脚本以索引0/2/6读取日期/收盘/涨跌幅 | 位置字段脆弱，必须与官方页面核列；不能将索引当长期公开规范 |
| HSI-HISTORY / HSTECH-HISTORY | `data[0].valueHistory[]`中`date / close` | 检查序列与相邻交易日；差价收益与含股息收益区分；旧脚本计算涨跌幅不代表官方字段 |

这些结构仅是旧脚本使用事实，**本次没有取得三个代表行情地址的可解析响应**；因此不可据本包直接标“接通”。同域官方PDF能打开也不能证明API可用。

新浪作者feed旧解析器依赖`a.post-link`和URL中的条目ID；DOM或日期格式变化必须停止并重验。它只覆盖一次可见页面，不保证全集。其他作者先人工确认官方主页/原始发布入口。

巨潮、SSE、SZSE、HKEXnews、PBOC、CSRC及公司IR未提供经验证的通用采集API契约；本包只列人工网页入口或具体公开文档地址。具体文档ID不能按规律猜造；从官方检索结果跟随链接。对于需要正式授权的数据接口，另按服务商合同与官方文档接入。
