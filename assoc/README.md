# assoc/ — Chinese next-word association preset

`assoc_preset_zh.bin` is the preset "next word" table shipped inside the iOS keyboard:
after the user commits a Chinese word, the keyboard offers up to 6 likely next words taken from this file.

## How it is built

| Input | License | Use |
|---|---|---|
| `zh-dict/tri_zh_pinyin.5w.dict.yaml` (this repository) | CC BY-SA 4.0 (derived from CC-CEDICT + Apache-2.0 / MIT sources) | the vocabulary and word frequencies |
| Tatoeba `cmn_sentences.tsv` | CC BY 2.0 FR (attribution only) | bigram co-occurrence counts from real sentences |
| OpenCC `STCharacters.txt` | Apache-2.0 | traditional → simplified normalisation before counting |

Because the vocabulary comes from the CC BY-SA 4.0 Chinese dictionary source, **this file is a derivative work
and is published under CC BY-SA 4.0**, like the rest of this repository. See [`../LICENSE`](../LICENSE).

## Build-time filtering

The table is filtered at build time by a blocklist that removes sensitive / offensive
`(headword, suggestion)` pairs (profanity, sexual, disease/death, violence, discriminatory terms).
Filtered pairs never enter the file, so there is no runtime cost and no blocklist inside the app.
The 2026-11-24 build removed 201 pairs: 176 by the multi-character term rule and 25 judged by hand
(neutral headword → a violent causative such as 下令→杀死, and vulgar collocations).
Headwords whose every suggestion was blocked drop out of the table entirely.

## Evaluation hold-out (since 2026-11-24)

Some Tatoeba sentences are **not counted**, so the table can be measured on sentences it has never seen:

- every sentence whose Tatoeba id satisfies `id % 100 == 0` or `id % 100 == 50` (1,777 sentences);
- 734 further sentences that also appear in the product's private evaluation set.

86,700 of 89,211 sentences were counted. The second list is not published, so a rebuild from the public inputs
alone gives a table that differs slightly from this file.

## Contents

- Format: `../formats/assoc_preset.md`
- 29,869 headwords / 86,491 pairs / 953,230 bytes
- Built 2026-11-24 (previous build 2026-11-13: 30,132 headwords / 87,458 pairs); sha256 in `../manifest.json`

## Rebuilding

The build script lives in the (private) product repository (`data/pipeline/build_assoc_data.py`);
the inputs listed above, plus the hold-out rule, reproduce it up to the unpublished evaluation sentences. Rebuilds change the byte content, so re-publish
this file and update `manifest.json` whenever the table is regenerated.
