# pivot_full.sqlite 格式（契约，v1）

- 生产方：A2。消费方：A3（`core/src/dict/`，通过 `tri_dict_*` 暴露给宿主 App）。
- 用途：宿主 App 内的完整词条（无内存上限）。与 pivot_lite.bin 共用 entry_id。

## 必须包含的表（列详情见下面「布局」一节，该节自 2026-10-26 起冻结为 v1）

| 表 | 内容 |
|---|---|
| `meta` | 格式版本、源数据版本、生成时间 |
| `entry` | entry_id、三语词头、假名、罗马音、词性 |
| `sense` | entry_id、序号、三语释义、是否默认义项、置信度 |
| `conjugation` | entry_id、活用形名称、写法（动词 / 形容词） |
| `example` | entry_id、原文、译文、来源（Tatoeba 等）、许可 |
| `source` | 每条数据的来源词典与版本（用于许可声明页） |
| `fts` | 三语词头与释义的全文索引（宿主 App 搜索，CJK 逐字切分，见「布局」§2） |

## `tri_dict_get_entry_json` 返回的 JSON

```json
{
  "entry_id": 123,
  "zh": "吃饭", "ja": "ご飯", "en": "meal",
  "kana": "ごはん", "romaji": "gohan", "pos": "名詞",
  "senses": [{ "zh": "米饭；饭；餐", "en": "rice; meal", "default": true }],
  "conjugations": [],
  "examples": [{ "text": "ご飯を食べましょう。", "translation": "我们吃饭吧。", "source": "Tatoeba #12345" }],
  "keigo": null,
  "sources": ["JMdict 2026-10", "CC-CEDICT 2026-10"]
}
```

## 布局

本节自 2026-10-26 起冻结为 **v1**（`PRAGMA user_version = 1`）。改动表结构必须新开 `[contract]` PR 并升 `user_version`。

### 0. 总则

- 单个文件，只读打开。**表名、列名、类型、可空性、行语义都是契约**；只允许新增列/表（向后兼容），不允许改名或改语义。
- 文本一律 UTF-8；时间一律 ISO 8601 UTC（`2026-10-26T00:00:00Z`）；置信度 `REAL`，范围 `0.0–1.0`。
- `entry_id` 与 `pivot_lite.bin`、`keigo.bin` 是同一套编号，从 1 开始，0 保留为"无"。**`pivot_full.sqlite` 是全量（20 万档全集），`pivot_lite.bin` 是它的子集。**
- 生成方写完数据后执行一次 `VACUUM` + `ANALYZE`，产物里不留空闲页；读取方不要假设 `page_count` 有冗余。

### 1. 表结构（DDL 即契约）

```sql
PRAGMA user_version = 1;

-- 键值元信息。必有这些 key：
--   format_version / source_version / generated_at / entry_count
--   pipeline_version / min_confidence / fts / license_summary
CREATE TABLE meta (
  key   TEXT PRIMARY KEY,
  value TEXT NOT NULL
);

-- 来源词典。App 的「词典来源与许可」页直接读这张表。
CREATE TABLE source (
  source_id   INTEGER PRIMARY KEY,
  name        TEXT NOT NULL,                -- "CC-CEDICT" / "JMdict" / "Tatoeba" / "pivot (derived)"
  version     TEXT NOT NULL,                -- 数据版本，如 "2026-10-26"
  license     TEXT NOT NULL,                -- 如 "CC-BY-SA-4.0" / "CC-BY-2.0-FR"
  url         TEXT NOT NULL,
  attribution TEXT NOT NULL                 -- 许可页要原样显示的署名文本
);

CREATE TABLE entry (
  entry_id         INTEGER PRIMARY KEY,     -- ≥1
  zh               TEXT,                    -- 中文词头（简体优先）
  ja               TEXT,                    -- 日文词头（汉字表记优先）
  en               TEXT,                    -- 英文词头（默认义项英文短释义）
  kana             TEXT,                    -- 平假名读音
  romaji           TEXT,                    -- 罗马字
  pos              TEXT,                    -- 词性（日文用日文名：名詞/動詞/形容詞/副詞/…）
  freq_rank        INTEGER NOT NULL DEFAULT 0,  -- 词频序位，1 = 最高频；0 = 未知。抽检分层用它
  default_sense_id INTEGER,                 -- → sense.sense_id，NULL 表示无义项
  confidence       REAL    NOT NULL DEFAULT 0,
  has_keigo        INTEGER NOT NULL DEFAULT 0,  -- A4 的 keigo.bin 里有记录时为 1
  source_id        INTEGER NOT NULL REFERENCES source(source_id)
);
CREATE INDEX idx_entry_zh ON entry(zh);
CREATE INDEX idx_entry_ja ON entry(ja);
CREATE INDEX idx_entry_freq ON entry(freq_rank);

CREATE TABLE sense (
  sense_id   INTEGER PRIMARY KEY,
  entry_id   INTEGER NOT NULL REFERENCES entry(entry_id),
  seq        INTEGER NOT NULL,              -- 词条内顺序，0 起
  zh         TEXT,                          -- 中文释义
  ja         TEXT,                          -- 日文释义
  en         TEXT,                          -- 英文释义（对齐用的 bridge gloss，原样保留）
  is_default INTEGER NOT NULL DEFAULT 0,    -- 0/1；每个 entry 恰好一行 =1
  confidence REAL    NOT NULL DEFAULT 0,
  source_id  INTEGER NOT NULL REFERENCES source(source_id)
);
CREATE INDEX idx_sense_entry ON sense(entry_id, seq);
CREATE UNIQUE INDEX idx_sense_default ON sense(entry_id) WHERE is_default = 1;

CREATE TABLE conjugation (
  entry_id  INTEGER NOT NULL REFERENCES entry(entry_id),
  form      TEXT NOT NULL,                  -- 辞書形/ます形/て形/た形/ない形/…
  written   TEXT NOT NULL,                  -- 该活用形的写法（汉字表记）
  reading   TEXT,                           -- 该活用形的假名读音
  source_id INTEGER NOT NULL REFERENCES source(source_id),
  PRIMARY KEY (entry_id, form)
);

CREATE TABLE example (
  example_id       INTEGER PRIMARY KEY,
  entry_id         INTEGER NOT NULL REFERENCES entry(entry_id),
  text             TEXT NOT NULL,           -- 原文（日文句优先）
  translation      TEXT,                    -- 译文（中文优先，缺中文时英文）
  translation_lang TEXT,                    -- "zh" / "en" / NULL
  source           TEXT NOT NULL,           -- "Tatoeba #12345"
  license          TEXT NOT NULL            -- "CC-BY-2.0-FR" / "CC0-1.0"
);
CREATE INDEX idx_example_entry ON example(entry_id);

-- 逐条溯源：每个 entry 的候选对来自 CC-CEDICT 的哪条、JMdict 的哪条。
-- 用于抽检、许可公示，以及"这个对照是怎么来的"的内部追查（不对外暴露）。
CREATE TABLE evidence (
  entry_id INTEGER NOT NULL REFERENCES entry(entry_id),
  dict     TEXT NOT NULL,                   -- "cedict" / "jmdict" / "tatoeba" / "llm"
  dict_key TEXT NOT NULL,                   -- cedict: "简体/繁体"；jmdict: ent_seq；tatoeba: 句 id
  note     TEXT,                            -- 桥接用的英文 token、LLM 说明等
  weight   REAL NOT NULL DEFAULT 0
);
CREATE INDEX idx_evidence_entry ON evidence(entry_id);

-- 全文索引（三语词头 + 默认义项）。CJK 逐字切分后建索引，见 §2。
CREATE VIRTUAL TABLE entry_fts USING fts5(
  zh, ja, en, kana, romaji, gloss,
  content='',                               -- 不冗余存原文，只存索引
  tokenize='unicode61'
);
```

### 2. FTS 的约定与降级

**为什么不用 trigram**：trigram 分词器要求查询串**至少 3 个字符**，而中文最常见的查词典输入
就是两个字（"吃饭"、"学校"）。用它等于让最常用的一类查询全部落回 `LIKE`。

**做法**：用 `unicode61` 分词器，**建索引前把 CJK 逐字切开**（每个汉字/假名一个 token），
ASCII 字母数字保持整词：

```
"吃饭"      → "吃 饭"
"USB接口"   → "USB 接 口"
"ご飯を食べる" → "ご 飯 を 食 べ る"
```

六列都按这个规则切（`zh / ja / en / kana / romaji / gloss`；`gloss` 是默认义项三语释义
用 `" / "` 拼起来的一段文本）。查询时把用户输入**按同样规则切分后包成短语**：

```
查询 "吃饭"    → MATCH '"吃 饭"'      相邻才命中（不会命中"吃饭团"）
查询 "吃"      → MATCH '"吃"'         单字也能命中
查询 "usb接口" → MATCH '"usb 接 口"'  大小写不敏感（unicode61 默认）
```

- `entry_fts.rowid = entry.entry_id`，`content=''`（只存索引，查询要 join `entry`）。
- 查询串切不出任何 token（纯标点）时直接返回空结果，不要拼 `MATCH ''`——那会报语法错。
- SQLite 的版本不需要 3.34+（`unicode61` 是很早就有的分词器）；FTS5 不可用时把
  `meta.fts` 读成 `none`，全部走 `LIKE '%q%'`，功能不降级、只是慢。

### 3. `tri_dict_search_json` 返回的 JSON

```json
{
  "query": "吃饭",
  "total": 3,
  "results": [
    { "entry_id": 123, "zh": "吃饭", "ja": "ご飯", "en": "meal",
      "kana": "ごはん", "pos": "名詞", "freq_rank": 812, "brief": "米饭；饭；餐" }
  ]
}
```

`total` 是命中总数（可以大于 `results` 的长度）；`brief` 取默认义项的中文释义，没有则英文。

### 4. `tri_dict_get_entry_json` 的字段对应

| JSON 字段 | 来源 |
|---|---|
| `entry_id` / `zh` / `ja` / `en` / `kana` / `romaji` / `pos` | `entry` 同名列 |
| `senses[]` | `sense` 按 `seq` 升序；`default` = `is_default = 1`；`en` / `zh` / `ja` 原样返回，另附 `confidence` |
| `conjugations[]` | `conjugation`，顺序固定：辞書形、ます形、て形、た形、ない形、可能形、受身形、使役形、意志形、命令形 |
| `examples[]` | `example`，最多 3 条，附 `translation_lang` |
| `keigo` | 恒为 `null`——敬语在 `keigo.bin`，由 A4/A3 的接口取 |
| `sources[]` | `source.name || ' ' || source.version`，按 source_id 升序 |
| `confidence` / `freq_rank` | `entry` 同名列（v1 追加字段，App 可不用） |

### 5. 产物位置

- 只有一份：`build/data/pivot_full.sqlite`（20 万档全集）。宿主 App 包内 `TriData/pivot_full.sqlite`。
- 体积预估：20 万词条 × （义项 ~1.4 + 例句 ~0.2 + 溯源 2 行）≈ 60–120 MB。宿主 App 无内存上限，不受 40 MB 约束。

### 6. 一致性自测（A3 可直接用）

`python -m pipeline.build_full --fixtures`（在 `data/pipeline/` 下）用仓库里的 200 条迷你词典生成
`build/data/fixtures/pivot_full_mini.sqlite` 与 `pivot_full_mini.expected.json`（若干 `entry_json` /
`search_json` 的期望输出）。A3 的单元测试按本文件的 JSON 形状比对即可，不需要真词典。
