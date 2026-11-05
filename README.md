# 中日对照数据（Chinese–Japanese pivot）

**本仓库只有一个用途：按 CC BY-SA 4.0 提供输入法用到的词典数据本身**——
「中日对照词对」（根目录）和「中文输入词库源」（`zh-dict/`）。
它不含产品代码、不含构建工具、也不含任何面向终端用户的功能。

数据由 **CC-CEDICT**（中→英）与 **JMdict**（日→英）派生而来（`zh-dict/` 只用到 CC-CEDICT，另并入 Apache-2.0 / MIT 来源），因此本仓库全部内容
**必须**以 [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/deed.zh)
授权：可以商用、可以修改，但**分发修改后的版本必须沿用同一协议**，并保留下面的署名。

## 文件

**大文件放在 [Release `data-2026-11-05`](https://github.com/pilafbigking/ime-pivot-data/releases/tag/data-2026-11-05) 的附件里**
（避免历史膨胀），小文件在仓库里。这一批就是随 App 分发的那一份：`pivot_lite.bin` 与 `pivot_full.sqlite`
的 sha256 与装进手机的文件一致。

| 位置 | 文件 | 大小 | 内容 |
|---|---|---|---|
| Release | `pivot_lite.bin` | 15.1 MB | 5 万档二进制（随键盘分发），58,179 条 / 45,756 个中文词头 / 18,786 个日文词头 |
| Release | `pivot_full.sqlite.gz` | 34.6 MB | 厚词典（SQLite，gzip，随宿主 App 分发）：200,000 词条 / 200,000 义项 / 227,755 活用 / 198,761 例句 |
| Release | `pivot_candidates.tsv.gz` | 77.5 MB | **源形态**：1,539,158 条候选对（TSV，gzip） |
| Release | `pivot_refined.tsv.gz` | 30.4 MB | 校正结果：`pivot_lite` 与 `pivot_full` 的构建都读它 |
| 仓库 | `examples/pivot_examples.tsv.gz` | 5.1 MB | 例句缓存：198,761 句，每句带 Tatoeba 句子 id |
| 仓库 | `tables/` | 224 KB | 构建输入的人工 / 派生小表：`pairs/`（配对）、`bridges/`（英文桥接键）、`overrides/`（人工复核结论）、`seed/`（商务词表） |
| 仓库 | `zh-dict/` | 约 5 MB | **中文输入词库源**（5 万档 rime 词典源 + 补词表 + 上游许可原文），详见 [`zh-dict/README.md`](zh-dict/README.md) |
| 仓库 | `manifest.json` | — | 每个文件的字节数、sha256、所在位置、下载地址、来源与复现说明 |
| 仓库 | `formats/` | — | `pivot_lite.md`（二进制格式）、`pivot_full.md`（SQLite schema） |

**建议从 `pivot_candidates.tsv.gz` 入手**：它是可读、可改的源形态；
`.bin` 与 `.sqlite` 是给运行时直接 mmap / 查询用的，格式在 `formats/` 里有完整文档。

```bash
# 下载大文件（也可以在网页上点）
for f in pivot_lite.bin pivot_full.sqlite.gz pivot_candidates.tsv.gz pivot_refined.tsv.gz; do
  curl -LO "https://github.com/pilafbigking/ime-pivot-data/releases/download/data-2026-11-05/$f"
done
gunzip -c pivot_candidates.tsv.gz | head -2
```

## 校验

```bash
# 校验当前目录里存在的文件（仓库里的小文件按相对路径，Release 附件放在当前目录）
python3 - <<'PY'
import json, hashlib, os
for name, v in json.load(open('manifest.json'))['files'].items():
    path = name if os.path.exists(name) else os.path.basename(name)
    if not os.path.exists(path): continue
    ok = hashlib.sha256(open(path, 'rb').read()).hexdigest() == v['sha256']
    print('OK ' if ok else 'BAD', name)
PY
```

## 旧版本

2026-10-27 的第一版发布（`pivot_lite*.bin`、`pivot_candidates.tsv.gz`、`pivot_full.sqlite.gz`）与当前 App 已不同源，
2026-11-05 起从当前文件树里删除，仍可在 git 历史里找到（提交 `e67510f`）；已经取得那一版的人，
其依 CC BY-SA 4.0 获得的权利不受影响。

## 署名（必须保留）

| 来源 | 用于 | 许可 |
|---|---|---|
| [CC-CEDICT](https://cc-cedict.org/) | 中文词头、拼音、英文释义 | CC BY-SA 4.0 |
| [JMdict](https://www.edrdg.org/jmdict/j_jmdict.html)（**EDRDG**） | 日文词头、假名、词性、释义 | CC BY-SA 4.0 |
| [rime-pinyin-simp](https://github.com/rime/rime-pinyin-simp) | `zh-dict/` 主库词条、拼音、词频 | Apache-2.0 |
| [jieba](https://github.com/fxsjy/jieba) | `zh-dict/` 长尾词与词频 | MIT |
| [pinyin-data](https://github.com/mozillazg/pinyin-data) | `zh-dict/` 逐字读音 | MIT |
| [OpenMultilingualWordNet](https://github.com/omwn/omw-data)（NICT 日本語 WordNet、COW 中文开放 WordNet） | 中日义项对齐证据（仅构建期使用） | WordNet 式宽松许可（可商用，不要求同协议公开） |
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

`pivot_lite.bin` 生成于 2026-11-02，`pivot_full.sqlite` 生成于 2026-10-31，`zh-dict/` 生成于 2026-11-05。
各文件 sha256 见 `manifest.json`。

**复现说明**：用本发布的 `pivot_candidates` + `pivot_refined` + `pivot_examples` 与 `tables/` 重建，
实测 `pivot_lite.bin` 与发布版只有文件头偏移 40–42 的 3 个字节（构建时间戳）不同；`pivot_full.sqlite`
所有表逐行一致，只有 `meta.generated_at` 不同。但**从原始词典重新跑全量对齐不保证逐字节一致**——
条目编号的排序键里存在并列，不同机器或并行度下并列名次可能翻转，导致后续条目整体位移，而且 Tatoeba
的每周导出无法固定。所以请以这里发布的 candidates / refined 为起点，不要重跑对齐。

## 不包含什么

- **代码**：产品与构建工具都不在本仓库，也不在本仓库的任何历史里。
- **敬语（敬語）数据**：`keigo.bin` 是另一份自有数据，不在 CC BY-SA 覆盖范围内，单独授权。
- **没有随包的东西**：`pivot_ids.tsv`（只供敬语对齐，v1 范围外）、10 万 / 20 万档（对照实验档）、LLM 试跑归档与 WikiMatrix 配对（未套进产物）。
- **评测集与标注**：同样是自有资产，不随本仓库分发。
- **源数据本体**：CC-CEDICT / JMdict 请各自到上游获取；本仓库只提供派生结果。

## 反馈

数据问题（错配、缺词、读音错误）欢迎开 issue 指出来，附上 `entry_id` 或词对即可。
