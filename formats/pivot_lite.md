# pivot_lite.bin 格式（契约，v1）

- 生产方：A2（`data/pipeline/`）。消费方：A3（`core/src/pivot/`）。
- 用途：键盘扩展内 mmap 只读，支撑外语行与薄卡片。**内存预算**：mmap 后计入 phys_footprint 的增量 < 5 MB（M0 第 5 项）。
- 本文件只定"必须有什么"和"不变量"。具体字节布局由 A2、A3 在第一个 PR 里共同补全到本文件的「布局」一节，合入后冻结为 v1。

## 必须包含的信息

| 段 | 内容 | 用途 |
|---|---|---|
| 文件头 | magic `TRIPIVOT`、格式版本、生成时间、源数据版本、各段偏移 | 打开时校验；版本不符直接拒绝 |
| 词条表 | `entry_id → { zh, ja, en, kana, romaji, pos, gloss_zh, gloss_ja, gloss_en, example_id, flags }` | 薄卡片；flags 含 `has_keigo`、`has_full_entry` |
| 三个词头索引 | 每种语言一份：`词头 → [(entry_id, confidence)]`，按 confidence 降序 | 精确查找与正向最大匹配分词 |
| 例句表 | `example_id → (原文, 译文)` | 卡片里的 1 条例句 |
| 字符串池 | UTF-8，去重 | 节省体积 |

## 不变量

1. `entry_id` 从 1 开始，0 保留为"无"。**同一份 pivot 源生成的 pivot_lite.bin、pivot_full.sqlite、keigo.bin 使用同一套 entry_id。**
2. 字节序固定小端；所有偏移 32 位；文件整体 ≤ 64 MB。
3. 词头索引必须支持"从某字节位置起的最长前缀匹配"，不能要求整表读入内存（建议 marisa-trie 或排序数组 + 二分）。
4. 同一词头的多个义项中，confidence 最高者即"默认义项"；低于 `min_confidence`（配置，默认 0.6）的不显示。
5. 文件由 CI 生成，不手改。

## 布局

本节自 2026-10-26 起冻结为 **v1**。改动布局必须新开 `[contract]` PR 并升 `format_version`。

### 0. 总则

- 全部整数**小端**；所有偏移是**从文件起点算的绝对字节偏移**，均为 `u32`；文件 ≤ 64 MB。
- 每个段起始地址 4 字节对齐，段内记录也 4 字节对齐。读取方**不得假设**段顺序，一律按文件头里的偏移取。
- 字符串一律 UTF-8，**不做 NFC 归一化以外的改写**；生产方已做 NFC。读取方比较词头时按字节比较即可。
- **字符串引用统一用"字符串下标"（u32）**，下标 `0` 恒为空串（长度 0）。因此没有 `0xFFFFFFFF` 哨兵：**下标 0 就表示"无"**。
- 读取方打开文件时至少校验：`magic`、`format_version == 1`、`header_size == 128`、`file_size == 实际文件长度`、以及每个段 `off + size <= file_size`。任何一项不符返回打开失败（`TRI_E_INVALID_ARG` 一类的错误，具体由 A3 定），**不要**带着坏偏移继续读。

### 1. 文件头（128 字节，位于偏移 0）

| 偏移 | 长度 | 字段 | 说明 |
|---:|---:|---|---|
| 0 | 8 | `magic` | ASCII `TRIPIVOT`，无 NUL |
| 8 | 2 | `format_version` | `u16`，v1 = `1` |
| 10 | 2 | `header_size` | `u16`，= `128` |
| 12 | 4 | `flags` | `u32`：bit0 例句表非空；bit1 全部词条 `has_full_entry`；bit2 保留（生成方曾用过 LLM 校正）；bit3–31 保留，必须为 0 |
| 16 | 4 | `entry_count` | 词条条数 N |
| 20 | 4 | `example_count` | 例句条数 |
| 24 | 4 | `string_count` | 字符串条数（含下标 0 的空串） |
| 28 | 4 | `index_count` | 恒为 `3` |
| 32 | 8 | `source_hash` | `u64`，`source_version` 的 FNV-1a 64 位哈希；用于日志与跨平台比对 |
| 40 | 8 | `generated_at` | `u64`，Unix 秒（UTC） |
| 48 | 4 | `entry_off` | 词条表起点 |
| 52 | 4 | `entry_size` | 单条词条字节数，v1 = `48`；**读取方必须用它步进，不要写死 48** |
| 56 | 4 | `example_off` | 例句表起点（`example_count == 0` 时可为 0） |
| 60 | 4 | `example_size` | v1 = `16` |
| 64 | 4 | `indexdir_off` | 索引目录起点（3 条 × 24 字节） |
| 68 | 4 | `indexdir_size` | = `24` |
| 72 | 4 | `string_off` | 字符串偏移数组起点（`(string_count + 1)` 个 `u32`） |
| 76 | 4 | `pool_off` | 字符串池起点（UTF-8 原始字节） |
| 80 | 4 | `pool_size` | 字符串池字节数 |
| 84 | 4 | `source_str` | 字符串下标：源数据版本串，例如 `CC-CEDICT 2026-10-26; JMdict 2026-10-26; Tatoeba 2026-10-24` |
| 88 | 4 | `file_size` | 文件总字节数 |
| 92 | 2 | `min_confidence_milli` | `u16`，生成时用的置信度下限 ×10000（默认 `6000`）。**仅作记录**，实际过滤以 config 为准 |
| 94 | 2 | `reserved16` | 0 |
| 96 | 32 | `reserved` | 全 0 |

### 2. 词条表（每条 48 字节，按 `entry_id` 升序）

**`entry_id` 不保证连续**（5 万档是 20 万档的子集，id 沿用全集编号），读取方用二分查找定位，不要用 `entry_off + (id-1)*entry_size`。

| 偏移 | 长度 | 字段 | 说明 |
|---:|---:|---|---|
| 0 | 4 | `entry_id` | ≥ 1，全表唯一、升序 |
| 4 | 4 | `zh` | 字符串下标；中文词头（简体优先，无简体时用繁体） |
| 8 | 4 | `ja` | 日文词头（汉字表记优先，无汉字时用假名） |
| 12 | 4 | `en` | 英文词头（默认义项的英文释义短语） |
| 16 | 4 | `kana` | 假名读音（平假名） |
| 20 | 4 | `romaji` | 罗马字（由假名规则生成） |
| 24 | 4 | `pos` | 词性，与 `pivot_full.sqlite` 的 `entry.pos` 同值，如 `名詞` / `動詞` / `形容詞` / `副詞`；未知为空串 |
| 28 | 4 | `gloss_zh` | 薄卡片的中文释义（默认义项，短） |
| 32 | 4 | `gloss_ja` | 薄卡片的日文释义（默认义项，短） |
| 36 | 4 | `gloss_en` | 薄卡片的英文释义（默认义项） |
| 40 | 4 | `example_id` | 例句 id，`0` = 无例句（对应例句表下标 = `example_id - 1`） |
| 44 | 2 | `flags` | `u16`：bit0 `has_keigo`；bit1 `has_full_entry`；bit2 `has_example`；bit3–15 保留 0 |
| 46 | 2 | `confidence` | `u16`，`round(默认义项置信度 × 10000)`，范围 0–10000 |

### 3. 例句表（每条 16 字节，`example_id = 下标 + 1`）

| 偏移 | 长度 | 字段 | 说明 |
|---:|---:|---|---|
| 0 | 4 | `text` | 字符串下标：原文（日文句优先） |
| 4 | 4 | `translation` | 字符串下标：译文（中文优先，缺中文时用英文） |
| 8 | 4 | `source` | 字符串下标：来源，如 `Tatoeba #12345` |
| 12 | 4 | `reserved` | 0 |

例句不做去重池以外的压缩：同一条例句可被多个词条引用（共用 `example_id`）。

### 4. 索引目录（3 条 × 24 字节，顺序固定：zh、ja、en）

| 偏移 | 长度 | 字段 | 说明 |
|---:|---:|---|---|
| 0 | 1 | `lang` | `0` = zh，`1` = ja，`2` = en |
| 1 | 1 | `reserved8` | 0 |
| 2 | 2 | `max_term_bytes` | 该索引里最长词头的 UTF-8 字节数（读取方可用来剪枝） |
| 4 | 4 | `bucket_count` | 前缀桶数 |
| 8 | 4 | `bucket_off` | 桶目录起点（`bucket_count × 16`） |
| 12 | 4 | `term_count` | 词头条数 |
| 16 | 4 | `term_off` | 词头表起点（`term_count × 16`） |
| 20 | 4 | `post_off` | 倒排区起点 |

### 5. 前缀桶目录（每条 16 字节，按 `key` 升序）

桶用来把"最长前缀匹配"压到小范围扫描，避免整表读入。`key` 取词头的**前 3 个 UTF-8 字节**：

```
key = (b0 << 16) | (b1 << 8) | b2
```

不足 3 字节的（只有 1 字节或 2 字节的词头）缺位补 0。UTF-8 的续字节恒在 `0x80..0xBF`、ASCII 恒 ≥ `0x01`，所以补 0 不会与真实字节冲突，key 唯一。

| 偏移 | 长度 | 字段 | 说明 |
|---:|---:|---|---|
| 0 | 4 | `key` | 见上式 |
| 4 | 4 | `term_lo` | 词头表下标（含） |
| 8 | 4 | `term_hi` | 词头表下标（不含）；`term_hi > term_lo` |
| 12 | 4 | `reserved` | 0 |

### 6. 词头表（每条 16 字节，全局按词头 UTF-8 字节升序）

按 UTF-8 字节序排序后，同 key 的词头天然相邻，所以桶只记区间即可。桶内实测约几十条（20 万词档最大桶 < 2000），读取方可以桶内线性扫描，也可以桶内二分。

| 偏移 | 长度 | 字段 | 说明 |
|---:|---:|---|---|
| 0 | 4 | `term` | 字符串下标：词头 |
| 4 | 4 | `first_post` | 倒排区下标（含），绝对位置 = `post_off + first_post * 8` |
| 8 | 2 | `post_count` | 该词头的倒排条数，≥ 1 |
| 10 | 2 | `term_bytes` | 词头 UTF-8 字节数 |
| 12 | 4 | `reserved` | 0 |

### 7. 倒排（每条 8 字节，同一词头内按 confidence 降序）

| 偏移 | 长度 | 字段 | 说明 |
|---:|---:|---|---|
| 0 | 4 | `entry_id` | ≥ 1 |
| 4 | 2 | `confidence` | `u16`，×10000 |
| 6 | 1 | `flags` | bit0 = 该义项是所属词条的默认义项 |
| 7 | 1 | `reserved` | 0 |

### 8. 字符串池

- `string_off` 处是 `string_count + 1` 个 `u32`，相对 `pool_off` 的偏移，`off[0] = off[1] = 0`（下标 0 是空串）。
- 第 i 个字符串 = `pool[off[i] .. off[i+1])`，长度 = `off[i+1] - off[i]`，**无终止符**。
- 内容相同的字符串必须去重（生产方保证），读取方不得依赖下标顺序。

### 9. 读取方要实现的三件事（对齐 `core/src/pivot/pivot_lite.h`）

1. `lookup(term, lang)`：算桶 key → 二分桶目录 → 桶内二分词头 → 读倒排（已按 confidence 降序）。返回 `[(entry_id, confidence)]`。
2. `longest_prefix(text, byte_offset, lang)`：
   - 取 `text[byte_offset..]` 的前 1/2/3 个字节，得到三个候选 key（可用字节数不足就跳过）；
   - 命中的桶里找**最长**的、且等于 `text[byte_offset..]` 前缀的词头，返回其字节长度；都没有返回 0。
   - 注意 1–2 字节词头落在各自更短的 key 桶里，所以三个 key 都要探（这是本节唯一容易写错的地方）。
   - 匹配到词头后要检查 `byte_offset + len` 是否落在 UTF-8 字符边界上（字节比较已足够，但返回长度不能让调用方从半个字符继续切分）。
3. `entry(entry_id)`：词条表按 id 二分。

### 10. 存放位置与档位

- 默认产物 `build/data/pivot_lite.bin` = **5 万档**（v1 随包分发的就是它，A9 的 `scripts/copy_data.sh` 只认这个文件名）。
- 另外产出 `build/data/pivot_lite_100k.bin`、`build/data/pivot_lite_200k.bin`，仅供 M0 内存测试。
- 三档条目数分别是 vocabulary 里置信度/词频排序后的前 5 万、10 万、20 万；`entry_id` 不重新编号，所以三档之间同一个词的 `entry_id` 相同。

### 11. 一致性自测（A3 可直接用）

`python -m pipeline.build_lite --fixtures`（在 `data/pipeline/` 下）会用仓库里的 200 条迷你词典生成
`build/data/fixtures/pivot_lite_mini.bin` 和同目录的 `pivot_lite_mini.expected.json`（每条词条的
`lookup` / `longest_prefix` / `entry` 期望值）。A3 的单元测试读这对文件即可验证解读是否正确，
不需要真的词典。格式每次改动，这份 fixture 的 `format_version` 会一起变。
