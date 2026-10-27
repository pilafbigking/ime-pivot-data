# 中日对照数据（Chinese–Japanese pivot）

**本仓库只有一个用途：按 CC BY-SA 4.0 提供「中日对照词对」数据本身。**
它不含产品代码、不含构建工具、也不含任何面向终端用户的功能。

数据由 **CC-CEDICT**（中→英）与 **JMdict**（日→英）派生而来，因此本仓库全部内容
**必须**以 [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/deed.zh)
授权：可以商用、可以修改，但**分发修改后的版本必须沿用同一协议**，并保留下面的署名。

## 文件

| 文件 | 大小 | 内容 |
|---|---|---|
| `pivot_candidates.tsv.gz` | 85.5 MB | **源形态**：1,540,474 条候选对（TSV，gzip）。117,678 个中文词头 / 111,280 个日文词头 |
| `pivot_lite_200k.bin` | 37.3 MB | 20 万档二进制，200,000 条 |
| `pivot_lite_100k.bin` | 21.9 MB | 10 万档二进制，100,000 条 |
| `pivot_lite.bin` | 12.0 MB | 5 万档二进制，50,000 条 |
| `pivot_full.sqlite.gz` | 33.3 MB | 厚词典（SQLite，gzip）：200,000 词条 / 200,000 义项 / 225,240 活用 / 198,787 例句 |
| `manifest.json` | — | 每个文件的字节数、sha256、源数据版本、生成时间 |
| `formats/pivot_lite.md` | — | 二进制格式说明（字段、索引、置信度语义） |
| `formats/pivot_full.md` | — | SQLite schema 说明 |

**建议从 `pivot_candidates.tsv.gz` 入手**：它是可读、可改的源形态，列名见下；
`.bin` 与 `.sqlite` 是给运行时直接 mmap/查询用的，格式在 `formats/` 里有完整文档。

```bash
gunzip -c pivot_candidates.tsv.gz | head -2
# entry_id  zh      ja       en               kana   romaji  pos   zh_trad  confidence  score   ...  cooc  senses  ...
```

## 校验

```bash
sha256sum -c <(python3 -c "import json;m=json.load(open('manifest.json'));[print(v['sha256']+'  '+k) for k,v in m['files'].items()]")
```

## 署名（必须保留）

| 来源 | 用于 | 许可 |
|---|---|---|
| [CC-CEDICT](https://cc-cedict.org/) | 中文词头、拼音、英文释义 | CC BY-SA 4.0 |
| [JMdict](https://www.edrdg.org/jmdict/j_jmdict.html)（**EDRDG**） | 日文词头、假名、词性、释义 | CC BY-SA 4.0 |
| [Tatoeba](https://tatoeba.org/) | 例句与句对证据（**逐句保留 sentence id，可回溯原作者**） | CC BY 2.0 FR（部分 CC0） |
| [OpenCC](https://github.com/BYVoid/OpenCC) | 简繁字表（仅构建期使用） | Apache-2.0 |
| [Unihan](https://www.unicode.org/charts/unihan.html) | 字形变体（仅构建期使用） | Unicode License v3 |

**EDRDG 的署名要求**：使用 JMdict 派生的数据时须注明来源为 EDRDG，并保留其许可声明。

## 数据版本

```
CC-CEDICT      2026-09-27
JMdict         2026-09-26
Tatoeba        2026-10-24
OpenCC         1.1.9
Unihan         16.0.0
```

这批产物生成于 2026-10-27（UTC），各文件 sha256 见 `manifest.json`。

**注意**：重新生成**不保证**逐字节一致——条目编号的排序键里存在并列，
不同机器或不同并行度下并列名次可能翻转，导致后续条目整体位移。
要复现某一份产物，请直接用 `manifest.json` 里对应 sha256 的文件。

## 不包含什么

- **代码**：产品与构建工具都不在本仓库，也不在本仓库的任何历史里。
- **敬语（敬語）数据**：`keigo.bin` 是另一份自有数据，不在 CC BY-SA 覆盖范围内，单独授权。
- **评测集与标注**：同样是自有资产，不随本仓库分发。
- **源数据本体**：CC-CEDICT / JMdict 请各自到上游获取；本仓库只提供派生结果。

## 反馈

数据问题（错配、缺词、读音错误）欢迎开 issue 指出来，附上 `entry_id` 或词对即可。
