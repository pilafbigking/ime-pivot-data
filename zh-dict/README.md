# zh-dict · 中文输入词库源（CC BY-SA 4.0）

三语输入法的中文拼音词库源文件，含 CC-CEDICT 派生内容，因此整体按 **CC BY-SA 4.0** 公开
（<https://creativecommons.org/licenses/by-sa/4.0/>）。

| 文件 | 说明 |
|---|---|
| `tri_zh_pinyin.5w.dict.yaml` | rime 词典源，随 App 分发的 5 万档（实际 149,072 条：单字 + 主库词 + jieba 常用词 + 补词）；列：词 TAB 拼音（无声调，ü 记 v）TAB 权重 |
| `cedict_supplement.tsv` | 其中的补词表（80,151 条）：CC-CEDICT 的词头与拼音，加项目自建的现代词；同样的列 |
| `licenses/` | 各上游许可原文 |

## 来源与署名

| 来源 | 许可 | 用途 |
|---|---|---|
| CC-CEDICT（<https://cc-cedict.org/>，© MDBG 与贡献者，2026-09-27 快照，sha256 `05bb7cf9…a95a`） | CC BY-SA 4.0 | 补词：词头与人工整理的拼音。**本目录是它的修改版**：只取词头与拼音（不含英文释义）、去声调、拼音转 rime 写法（`u:` → `v`，儿化 `r5` → `er`）、加权重；不再过滤专名 / 异体 / 姓氏条目 |
| rime-pinyin-simp（Android PinyinIME 派生） | Apache-2.0 | 主库词条、拼音、词频 |
| jieba `dict.txt` | MIT | 常用长尾词与词频（权重封顶 100） |
| mozillazg/pinyin-data | MIT | 长尾词的逐字读音 |

Apache-2.0 与 MIT 的内容并入本 CC BY-SA 作品时，其原有版权声明与许可文本保留在 `licenses/`。

只含随 App 分发的 5 万档；10 万 / 20 万档是内存对照实验档，不随包，也不在此发布。
不含：代码、英文词库、编译产物（`.bin`）。校验值见仓库根目录的 `manifest.json`。
