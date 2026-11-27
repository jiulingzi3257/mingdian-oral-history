# 明典人生 · 长辈口述传记访谈系统 / Mingdian Life: Elder Oral-History Biography Interview System

> **语言说明 / Language Note**：本仓库文档主体为简体中文，便于国内研究者与使用者阅读；关键声明、核心术语提供中英双语对照。 The documentation in this repository is primarily in Simplified Chinese, with bilingual key statements and core terminology for international collaboration.

AI 以晚辈口吻陪长辈闲谈，按多尺度时间线采集生平素材，沉淀为带证据分级的人生档案，产出网页时间轴、Word 传记与结构化档案表。 A junior-tone AI companion conducts life-story interviews with elders, aggregates material onto a multi-scale timeline with evidence grading, and outputs a web timeline, a Word biography and a structured archive table.

**项目定位 / Positioning**：一个比 C 端产品更懂学术伦理，比学术工具更懂适老化交互的开源访谈框架。 An open-source interview framework that attends to research ethics more than consumer products, and to elder-friendly interaction more than academic tools.

> 规则与文献再严密，若交互像一台"审问机"，老人依然会觉得不好用；伦理约束最终要落到老人可感知的交谈分寸上。 However rigorous the rules and references, an interrogation-like interface will still feel wrong to elders; ethical guardrails must land in a tone and pace the elder can actually feel.

## 当前阶段 / Current Stage

MVP 骨架：确定性规则层承担动作门禁、证据标记与留档；模型仅负责自然语言生成，可整体退回本地 mock 模板。语音识别（ASR）与语音合成（TTS）仍为可替换占位。 An MVP skeleton: a deterministic rule layer owns action guardrails, evidence marking and archiving; the model only generates natural language and can be swapped for a local mock template. ASR/TTS remain replaceable placeholders.

- **模型对话 / Model dialogue**：已接入星火 `spark-x2.6`（OpenAI 兼容 chat 协议）；`MODEL_PROVIDER=mock` 可退回本地模板。 Connected to iFlytek Spark `spark-x2.6` (OpenAI-compatible chat); set `MODEL_PROVIDER=mock` to fall back to local templates.
- **方言 / Dialect**：仅 AI 口吻层面（称呼、语气词），方言语音转写接口预留。 Dialect affects the AI's tone of address and particles only; speech-transcription interfaces are reserved.
- **知情同意 / Informed consent**：访谈前强制签署，两档授权（家庭留存 / 对外公开，后者需单独勾选）；未签署不能进入会话，签署记录带版本号留档。 Consent is mandatory before a session, with two authorization tiers (family archive / public release, the latter opt-in); no session starts without a signed record, stored with a version number.
- **知识库 / Knowledge base**：老年心理学与口述史方法论文献已分级入库，见 `server/data/knowledge.json` 与 `docs/knowledge-base-evidence.md`. Graded references in gerontology and oral-history methodology are stored in `server/data/knowledge.json` and listed in `docs/knowledge-base-evidence.md`.

## 技术栈 / Tech Stack

- 后端 / Backend：Node.js 22 + Express 5；JSON 文件持久化（数据层接口可替换为 SQLite/Postgres）。 JSON-file persistence with a swappable data-layer interface.
- 前端 / Frontend：React 19 + Vite 8 + ECharts 6.
- 导出 / Export：`docx` 生成 Word；CSV 手写生成（带 BOM）。 Word via `docx`; hand-written CSV with BOM.

## 目录 / Repository Layout

```
mingdian-oral-history/
├── docs/knowledge-base-evidence.md   # 文献分级清单（人读）/ Graded reference list (human-readable)
├── server/
│   ├── data/knowledge.json           # 知识库结构化条目 / Structured knowledge entries
│   ├── src/
│   │   ├── index.js                  # Express 入口，托管前端 dist / Express entry, serves frontend dist
│   │   ├── db.js                     # 数据层（JSON 持久化 CRUD 抽象）/ Data layer (JSON persistence CRUD)
│   │   ├── providers/                # model / asr / tts 抽象 + mock / Provider abstractions + mocks
│   │   ├── engine/
│   │   │   ├── rules.js              # 访谈规则引擎（规范 + 安全回路）/ Interview rule engine
│   │   │   ├── memory.js             # 事件卡、线索、重复计数、时序聚合 / Event cards, leads, counts, timeline
│   │   │   ├── knowledge.js          # 知识库加载与分级过滤 / Knowledge loading and grading
│   │   │   ├── dialect.js            # 方言口吻包 / Dialect tone packs
│   │   │   └── prompts.js            # 模型提示词组装 / Prompt assembly
│   │   ├── routes/                   # interview / archive / export
│   │   └── export/                   # 传记组装、docx、csv / Biography assembly, docx, csv
└── web/                              # React 前端 / React frontend
```

## 启动 / Getting Started

```bash
# 安装前后端依赖 / Install both sides
npm run install:all

# 开发模式（两个终端）/ Development (two terminals)
npm run dev:server     # 后端 / backend, http://127.0.0.1:9000
npm run dev:web       # 前端 / frontend, Vite dev server proxies /api to 9000

# 生产：先构建前端，再由后端托管 / Production: build frontend, then serve via backend
npm run build
npm start
```

## 环境变量 / Environment Variables

未配置时默认走 mock。凭证只经环境变量注入，不进源码与交付包；样例见 `server/.env.example`. Falls back to mock when unset. Credentials are injected via environment variables only; see `server/.env.example`.

| 变量 / Variable | 说明 / Description |
|---|---|
| `MODEL_PROVIDER` | `xinghuo` 启用星火；`mock` 为本地模板。 `xinghuo` enables Spark; `mock` uses local templates. |
| `XFYUN_API_KEY` | 星火密钥（`ak-` 前缀），写入 `server/.env`，不进源码与交付包。 Spark key (`ak-` prefix), kept in `server/.env`, never committed. |
| `XFYUN_BASE_URL` | 星火接入点 / Spark endpoint. |
| `XFYUN_MODEL` | 模型标识 / Model identifier. |
| `ASR_PROVIDER` / `ASR_API_BASE` / `ASR_API_KEY` | 语音识别占位配置 / ASR placeholder config. |
| `TTS_PROVIDER` / `TTS_API_BASE` / `TTS_API_KEY` | 语音合成占位配置 / TTS placeholder config. |
| `PORT` / `HOST` | 端口与监听地址，默认 9000 / 0.0.0.0. Port and bind address, default 9000 / 0.0.0.0. |

## 访谈规则 / Interview Rules

**克制口径 / Restraint principle**：老人的每一句话都"无征"接纳；AI 的每一次追问、每一条入档必须"有征"克制。"征"是 AI 的自我约束，不是对老人的举证要求。 Every utterance from the elder is accepted without demanding proof; the AI's follow-up questions and archiving decisions are held to an evidence standard. The burden of restraint is on the AI, not on the elder's memory.

| 动作 / Action | 触发 / Trigger | 行为 / Behavior |
|---|---|---|
| `follow-up` | 上轮提取到人物/地点/食物实体 / Entities extracted last turn | 承接后从实体生长下一问 / Follow from the entity |
| `warmup` | 无可抓实体 / No entity to anchor on | 食物/旧物/时令具象切口兜底 / Concrete fallback opening |
| `echo`（父类语义 clarify） | 结构锚点含糊 / Structural anchor ambiguous | 回声确认，同一锚点最多一次；回声轮只复述锚点请其确认，不另开新话题 / One echo per anchor; confirmation only, no new topic |
| `respect` | 意愿拒绝信号 / Refusal signal | 留档为待访，会话内不二次强问 / Archive as deferred, do not re-press |
| `de-escalate` | 沉重情绪话题 / Heavy emotional topic | 先共情承接，再转回轻松日常；敏感含糊不澄清 / Empathize then shift to light topics; do not clarify sensitive ambiguity |
| `hold` | 高危表述 / High-risk statement | 不承接访谈，提示专业援助 / Do not continue the interview; prompt professional help |

含糊分流四态 / Ambiguity subtypes (`subType`)：

| 子类型 / Subtype | 含义 / Meaning | 处理 / Handling |
|---|---|---|
| `anchor_ambiguous` | 年代/人名/地点等结构锚点含糊 / Anchor (date/name/place) unclear | 回声一次，收敛即止 / Echo once, stop when settled |
| `detail_ambiguous` | 主体清楚、细节含糊 / Subject clear, detail unclear | 不追问，直接存疑入档 / Archive as uncertain, no follow-up |
| `sensitive_ambiguous` | 沉重语境中的含糊 / Ambiguity in heavy context | 不澄清，走 de-escalate / De-escalate, do not clarify |
| `refusal_signal` | 意愿拒绝信号 / Willingness-to-stop signal | 立即停止，标 `refused` / Stop immediately, mark refused |

> "记不清/忘了/不记得"属记忆提取失败，**不是拒绝**：只落 `uncertain`，不标 `refused`. "Can't recall / forgot / don't remember" are memory-retrieval failures, **not refusals**; they are marked `uncertain`, never `refused`.

**证据等级 / Evidence levels**：`single` 单次自述 → `repeated` 同簇提及 ≥3 次（重点叙事）；`uncertain` 存疑、`refused` 意愿拒绝；`verified` 为人工复核确认，规则层不自动转正。档案表中存疑条目可由家属/经办人点「确认属实」人工转 `verified`，并留痕复核时间与经办人（`reviewedAt` / `reviewedBy`）；`refused` 条目不进入复核。事件卡 `versions[]` 并列各版原话，不覆盖、不合并成定论。 `verified` is granted only by human review; the rule layer never promotes automatically. Uncertain entries can be manually confirmed in the archive table, leaving reviewer and timestamp traces; refused entries are excluded from review. `versions[]` keeps every wording side by side, never overwritten into a single verdict.

### 规则溯源 / Rule Provenance

每条规则在 `rules.js` 的 `RULES` 表标明依据类型，可叠加。 Each rule in `rules.js` carries one or more provenance tags:

- `literature`：有核验文献支撑，`refs` 挂 KB 条目 id。 Backed by a verified reference, linked to a KB entry id.
- `ethics`：伦理底线（受访者自主、安全边界）。 Ethics baseline (autonomy, safety boundaries).
- `engineering`：产品交互与计数取舍（如回声上限、一轮一问），不由文献背书具体数值。 Engineering trade-off (echo cap, one question per turn); specific values are not literature-backed.

沉默给补充空间的时长（默认 10 秒）当前标为 engineering 经验值，待补沉默时长实证文献；意愿信号即停原则挂 ethics + 文献（KB-M01、KB-P06）。 The silence-to-space threshold (default 10 s) is an engineering default pending empirical evidence; the stop-on-refusal principle rests on ethics plus KB-M01/KB-P06.

## 数据模型要点 / Data Model Notes

- **事实分级**（事件卡）与**知识分级**（知识库 L1/L2/L3）分库存储，互不串味。 Fact grading (event cards) and knowledge grading (L1/L2/L3) are stored separately.
- 时序三层：事件卡 → 年代段 → 人生阶段；每条事件可回溯会话与原话摘录。 Three timeline layers: event card → decade segment → life phase; each event traces back to its session and original wording.
- 知识库每条含 `applicable_population`（适用人群与外推限制）、`evidence_age`（年代与时效）、`action_scope`（受控动作词表）、`evidence_ref`（DOI/PMID 短键）。 Each KB entry carries population scope, evidence age, controlled action scope, and DOI/PMID reference key.

## 边界与已知局限 / Boundaries and Limitations

- 本产品定位口述素材整理，**不是心理治疗**；识别临床抑郁/创伤/高危信号时提示寻求专业帮助。 Oral-material organization only, **not psychotherapy**; high-risk signals prompt referral to professionals.
- 实体提取为规则粗匹配（MVP）：人物靠亲属称谓、地点靠后缀字、物件仅限食物词表；旧物/场所/地名覆盖有限。 Entity extraction is coarse rule matching in the MVP; coverage of objects and places is limited.
- 高危 `hold` 分支由模型生成安全确认问句，规则层只保证不承接访谈往事。 The `hold` branch's safety wording is model-generated; the rule layer only guarantees the interview does not continue.
- 知识库证据多来自西方/机构样本，岭南社区认知正常老人的直接证据有限（见 `docs/` gaps）。 Most evidence comes from Western/institutional samples; direct evidence for cognitively intact elders in Lingnan communities is limited.

## 部署现状 / Deployment Status

当前为**本地可运行的全栈原型**（后端 Express 托管前端 `dist`，数据落 `server/data/db.json`），适合本机或内网有状态环境运行。 This is a locally runnable full-stack prototype (Express serves the frontend `dist`; data lives in `server/data/db.json`), suited to a stateful machine or intranet environment.

公网部署前需满足：① 数据层由本地文件库改为外部托管存储，移除运行时磁盘写；② 密钥与模型配置走平台环境变量/密钥管理，不进交付包；③ 补齐构建校验后再部署。 Before public deployment: (1) move the data layer to external managed storage and remove runtime disk writes; (2) keep credentials in platform environment variables/secret management, out of the package; (3) complete build validation before deployment.

## 适老化要点 / Accessibility for Elders

- 基础字号 19px、对话气泡 19px、按钮最小点击区 50px；窄屏单列、线索队列默认折叠。 Base text 19px, bubbles 19px, minimum button hit area 50px; single-column on narrow screens, lead queue collapsed by default.
- 同意书分步卡片逐条勾选，大白话措辞；签署称呼/关系必填，校验失败逐条提示。 Consent is a step-by-step card flow in plain language; name and relation are required, with per-item validation hints.
- 前端带错误边界，单页渲染异常给出可读提示与重试，不整树白屏。 An error boundary keeps a readable message and retry instead of a full white screen.

## 引用本项目 / Cite This Project

软件被引元数据见 [`CITATION.cff`](CITATION.cff)。 The machine-readable citation metadata is in [`CITATION.cff`](CITATION.cff).

**GB/T 7714—2025（草稿）**

> jiulingzi3257. 明典人生：长辈口述传记访谈系统: v0.1.0[CP/OL]. (2026). https://github.com/jiulingzi3257/mingdian-oral-history.

**APA 7th**

> jiulingzi3257. (2026). *Mingdian life: Elder oral-history biography interview system* (Version 0.1.0) [Computer software]. https://github.com/jiulingzi3257/mingdian-oral-history

## 开源协议 / License

本项目采用 **Apache-2.0** 协议开源，可依条款自由使用、修改与分发。 This project is released under the **Apache-2.0** license; you may use, modify and distribute it under its terms. See [`LICENSE`](LICENSE).

## 后续路线 / Roadmap

1. 方言 ASR/TTS 接入；实体抽取与情绪识别交由模型，规则层保留门禁与计数。 Dialect ASR/TTS; model-based entity extraction and sentiment, with the rule layer keeping guardrails and counts.
2. 中文老年心理/口述史文献补检；口述史方法论专著版本核验入库。 Further Chinese gerontology/oral-history references; verified editions of methodology monographs.
3. 数据层迁移到外部托管存储后，评估公网部署。 After migrating to managed storage, evaluate public deployment.
