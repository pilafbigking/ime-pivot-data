# assoc_preset_zh.bin 格式（v1）

- 生产方：产品仓库的 `data/pipeline/build_assoc_data.py`。消费方：iOS `TriKit/PresetAssociationReader.swift`（macOS 接入中）。
- 用途：中文「下一词联想」的预置表，键盘内 mmap 只读，零额外内存占用。
- 只定"必须有什么"与不变量，字节布局与生产脚本一一对应，改格式必须升 `version`。

## 内容

词头（用户刚上屏的中文词）→ 最多 6 条候选（下一个词），按语料共现强度降序。

## 布局（全部小端）

```text
Header (32 B)
  magic         char[8] = "TRIASSC1"
  version       uint32   = 1
  entry_count   uint32
  index_offset  uint32   = 32
  data_offset   uint32   = 32 + entry_count * 8
  reserved      uint64   = 0

Index (entry_count * 8 B)             # 按词头 UTF-8 字节序升序
  headword_offset uint32              # 相对 data_offset
  record_offset   uint32              # 相对 data_offset

Data
  [headword]  u8 长度 + UTF-8 字节
  [record]    u8 条数 + (u8 长度 + UTF-8 字节) * 条数
```

## 不变量

1. 索引按词头的 UTF-8 字节序升序，支持二分查找（消费方不需要整表读入）。
2. `entry_count == 0` 时 `data_offset == 32`；空表的索引段长度为 0。
3. 长度字段是 **字节**数不是字符数，UTF-8 中文词 1 个字符 3 个字节；单字段上限 255 字节。
4. 每词头最多 6 条候选，去重，且不能包含词头自己。
5. 构建期已按黑名单剔除敏感对；消费方**不做**任何过滤，也不需要黑名单。
6. 文件由脚本生成、不手改。重建会改变文件内容 → 必须重新发布并更新 `manifest.json`。

## 版本

`version = 1` 自 2026-11 起未变。新增字段或改布局必须升版本，并同步改消费方（iOS / macOS 读表代码对版本不符一律拒绝）。
