# zh-dict · 中文输入词库源（CC BY-SA 4.0）

三语输入法的中文拼音词库源文件，含 CC-CEDICT 派生内容，因此整体按 **CC BY-SA 4.0** 公开
（<https://creativecommons.org/licenses/by-sa/4.0/>）。

| 文件 | 说明 |
|---|---|
| `tri_zh_pinyin.5w.dict.yaml` / `.10w.` / `.20w.` | rime 词典源。5 万档为 App 默认；列：词 TAB 拼音（无声调，ü 记 v）TAB 权重 |
| `cedict_supplement.tsv` | 其中取自 CC-CEDICT 的补词（61,439 条），同样的列 |
| `licenses/` | 各上游许可原文 |

## 来源与署名

| 来源 | 许可 | 用途 |
|---|---|---|
| CC-CEDICT（<https://cc-cedict.org/>，© MDBG 与贡献者，2026-09-27 快照，sha256 `05bb7cf9…a95a`） | CC BY-SA 4.0 | 补词：词头与人工整理的拼音。**本目录是它的修改版**：筛选（去专名、姓氏、异体、儿化条目）、去声调、拼音转 rime 写法、加权重 |
| rime-pinyin-simp（Android PinyinIME 派生） | Apache-2.0 | 主库词条、拼音、词频 |
| jieba `dict.txt` | MIT | 长尾词与词频 |
| mozillazg/pinyin-data | MIT | 长尾词的逐字读音 |

Apache-2.0 与 MIT 的内容并入本 CC BY-SA 作品时，其原有版权声明与许可文本保留在 `licenses/`。

不含：代码、英文词库、编译产物（`.bin`）。校验值见仓库根目录的 `manifest.json`。
