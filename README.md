# Chinese–Japanese Pivot Data

**This repository has a single purpose: to provide, under CC BY-SA 4.0, the dictionary data used by an input method (IME)** —
the Chinese–Japanese word pairs (repository root) and the Chinese input dictionary source (`zh-dict/`).
It contains no product code, no build tools, and no end-user features.

The data is derived from **CC-CEDICT** (Chinese→English) and **JMdict** (Japanese→English). `zh-dict/` uses only CC-CEDICT, plus Apache-2.0 / MIT sources. As a result, everything in this repository
**must** be licensed under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/deed.en):
commercial use and modification are allowed, but **modified versions must be distributed under the same license**, and the attributions below must be kept.

## Files

**Large files are release assets of [Release `data-2026-11-05`](https://github.com/pilafbigking/ime-pivot-data/releases/tag/data-2026-11-05)**
(to avoid bloating git history); small files live in the repository. This batch is the one shipped with the app: the sha256 of `pivot_lite.bin` and `pivot_full.sqlite`
matches the files installed on the phone.

| Location | File | Size | Contents |
|---|---|---|---|
| Release | `pivot_lite.bin` | 15.1 MB | 50k-tier binary (shipped with the keyboard): 58,179 entries / 45,756 Chinese headwords / 18,786 Japanese headwords |
| Release | `pivot_full.sqlite.gz` | 34.6 MB | Full dictionary (SQLite, gzip, shipped with the host app): 200,000 entries / 200,000 senses / 227,755 inflections / 198,761 example sentences |
| Release | `pivot_candidates.tsv.gz` | 77.5 MB | **Source form**: 1,539,158 candidate pairs (TSV, gzip) |
| Release | `pivot_refined.tsv.gz` | 30.4 MB | Refined results: both `pivot_lite` and `pivot_full` are built from this |
| Repo | `examples/pivot_examples.tsv.gz` | 5.1 MB | Example-sentence cache: 198,761 sentences, each with its Tatoeba sentence id |
| Repo | `tables/` | 224 KB | Small hand-made / derived build-input tables: `pairs/` (pairings), `bridges/` (English bridge keys), `overrides/` (manual review verdicts), `seed/` (business vocabulary) |
| Repo | `zh-dict/` | ~5 MB | **Chinese input dictionary source** (50k-tier rime dictionary source + supplementary word list + upstream license texts); see [`zh-dict/README.md`](zh-dict/README.md) |
| Repo | `assoc/` | 941 KB | **Chinese next-word association preset** shipped inside the keyboard (30,132 headwords / 87,458 next-word pairs), derived from the 50k-tier dictionary above + Tatoeba; see [`assoc/README.md`](assoc/README.md) |
| Repo | `manifest.json` | — | Per-file byte size, sha256, location, download URL, provenance, and reproduction notes |
| Repo | `formats/` | — | `pivot_lite.md` (binary format), `pivot_full.md` (SQLite schema), `assoc_preset.md` (association table format) |

**Start with `pivot_candidates.tsv.gz`**: it is the readable, editable source form.
`.bin` and `.sqlite` are meant to be mmap'd / queried directly at runtime; their formats are fully documented in `formats/`.

```bash
# Download the large files (or click them on the release page)
for f in pivot_lite.bin pivot_full.sqlite.gz pivot_candidates.tsv.gz pivot_refined.tsv.gz; do
  curl -LO "https://github.com/pilafbigking/ime-pivot-data/releases/download/data-2026-11-05/$f"
done
gunzip -c pivot_candidates.tsv.gz | head -2
```

## Verification

```bash
# Verify the files present in the current directory (small repo files by relative path; release assets in the current directory)
python3 - <<'PY'
import json, hashlib, os
for name, v in json.load(open('manifest.json'))['files'].items():
    path = name if os.path.exists(name) else os.path.basename(name)
    if not os.path.exists(path): continue
    ok = hashlib.sha256(open(path, 'rb').read()).hexdigest() == v['sha256']
    print('OK ' if ok else 'BAD', name)
PY
```

## Previous versions

The first release of 2026-10-27 (`pivot_lite*.bin`, `pivot_candidates.tsv.gz`, `pivot_full.sqlite.gz`) no longer matches the app's source data.
It was removed from the current file tree on 2026-11-05 but can still be found in git history (commit `e67510f`); the rights
of anyone who already obtained that version under CC BY-SA 4.0 are unaffected.

## Attribution (must be kept)

| Source | Used for | License |
|---|---|---|
| [CC-CEDICT](https://cc-cedict.org/) | Chinese headwords, pinyin, English glosses | CC BY-SA 4.0 |
| [JMdict](https://www.edrdg.org/jmdict/j_jmdict.html) (**EDRDG**) | Japanese headwords, kana, parts of speech, glosses | CC BY-SA 4.0 |
| [rime-pinyin-simp](https://github.com/rime/rime-pinyin-simp) | `zh-dict/` main-dictionary entries, pinyin, word frequencies | Apache-2.0 |
| [jieba](https://github.com/fxsjy/jieba) | `zh-dict/` long-tail words and frequencies | MIT |
| [pinyin-data](https://github.com/mozillazg/pinyin-data) | `zh-dict/` per-character readings | MIT |
| [OpenMultilingualWordNet](https://github.com/omwn/omw-data) (NICT Japanese WordNet, COW Chinese Open WordNet) | Chinese–Japanese sense-alignment evidence (build time only) | WordNet-style permissive license (commercial use allowed; no share-alike required) |
| [Tatoeba](https://tatoeba.org/) | Example sentences and sentence-pair evidence (**sentence ids kept per sentence, traceable to original authors**) | CC BY 2.0 FR (partly CC0) |
| [OpenCC](https://github.com/BYVoid/OpenCC) | Simplified/Traditional character tables (build time only) | Apache-2.0 |
| [Unihan](https://www.unicode.org/charts/unihan.html) | Glyph variants (build time only) | Unicode License v3 |

**EDRDG attribution requirement**: data derived from JMdict must credit EDRDG as the source and retain its license notice.

## Data versions

```
CC-CEDICT      2026-09-27
JMdict         2026-09-26
Tatoeba        2026-10-24
OpenCC         1.1.9
Unihan         16.0.0
```

`pivot_lite.bin` was generated on 2026-11-02, `pivot_full.sqlite` on 2026-10-31, and `zh-dict/` on 2026-11-05.
See `manifest.json` for each file's sha256.

**Reproducibility**: rebuilding from the released `pivot_candidates` + `pivot_refined` + `pivot_examples` and `tables/`,
`pivot_lite.bin` differs from the released version only in 3 bytes at header offsets 40–42 (the build timestamp); `pivot_full.sqlite`
matches row for row in every table, differing only in `meta.generated_at`. However, **re-running the full alignment from the raw dictionaries is not guaranteed to be byte-identical** —
the sort key used for entry numbering contains ties, and tie ordering can flip across machines or degrees of parallelism, shifting all subsequent entries; also, Tatoeba's
weekly exports cannot be pinned. So please start from the candidates / refined files published here rather than re-running the alignment.

## What is not included

- **Code**: the product and build tools are not in this repository, nor anywhere in its history.
- **Honorific (keigo) data**: `keigo.bin` is a separate proprietary dataset, not covered by CC BY-SA, and licensed separately.
- **Items not shipped**: `pivot_ids.tsv` (used only for honorific alignment, outside the v1 scope), the 100k / 200k tiers (comparison-experiment tiers), archived LLM trial runs, and WikiMatrix pairs (not incorporated into the artifacts).
- **Evaluation sets and annotations**: also proprietary assets, not distributed with this repository.
- **Raw source data**: get CC-CEDICT / JMdict from their upstream projects; this repository provides derived results only.

## Feedback

Data issues (wrong pairings, missing words, incorrect readings) are welcome as GitHub issues — just include the `entry_id` or the word pair.
