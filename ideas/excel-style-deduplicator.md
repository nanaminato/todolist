# Excel / OOXML 样式去重与工作簿优化工具

## 背景

长期编辑、跨工作簿复制粘贴或由某些工具生成的 Excel 文件，可能积累大量重复或无用的单元格格式（XF），导致文件异常膨胀，并出现保存、复制粘贴、编辑时明显卡顿甚至 Excel 无响应。

实际排查案例中，一个实际使用区域很小的工作簿，其 `xl/styles.xml` 解压后接近 10 MB，`cellXfs` / `cellStyleXfs` 数量达到约 48,845，明显存在样式膨胀的可能。

该问题适合做成一个完全本地、离线运行的诊断和修复能力。未来可考虑加入 RelaxCat，也可以独立成其他工具或复用到其他项目中。

## 目标

提供一个针对 `.xlsx` / `.xlsm`（OOXML）的安全分析和优化工具：

1. 默认只分析，不修改原文件。
2. 统计 `styles.xml` 中的 fonts、fills、borders、cellStyleXfs、cellXfs 等数量。
3. 扫描各 worksheet 实际引用的 style ID。
4. 识别完全重复的 XF、未引用 XF，以及可安全去重的样式。
5. 输出不包含单元格文本内容的统计报告，方便处理公司/敏感文档时避免泄露业务数据。
6. 修复模式下建立 `old style id -> new style id` 映射，并同步更新 worksheet 中的 `s` 引用。
7. 保持单元格视觉效果、公式、值、合并单元格等业务内容不变。
8. 始终输出到新文件，不覆盖原始工作簿；后续可增加显式覆盖选项。

## 建议实现阶段

### Phase 1: Analyze

只读分析 OOXML ZIP：

- `xl/styles.xml` 大小
- fonts / fills / borders 数量
- cellStyleXfs / cellXfs 数量
- 唯一 XF 数量
- 完全重复 XF 数量
- 实际被 worksheet 引用的 style ID 数量
- 未引用 style ID 数量
- 各 worksheet 的 UsedRange / dimension（仅坐标，不读取或输出业务文本）

示例输出：

```text
CellXfs total:       48845
Unique XFs:          320
Duplicate XFs:       48525
Referenced XFs:      115
Unreferenced XFs:    48730
Potential reduction: 99%+
```

### Phase 2: Safe deduplication

第一版只合并语义完全相同、可证明等价的 XF：

- 对 XF 进行规范化后计算结构键/hash
- 构建旧 ID 到新 ID 的映射
- 重建 `cellXfs`
- 必要时处理 `cellStyleXfs` 及相关引用
- 更新 `xl/worksheets/*.xml` 中 `<c s="...">` 的 style ID
- 保持其他 OOXML part 不变
- 输出新的 `.xlsx` / `.xlsm`

不要在第一版尝试把“看起来一样但 XML 不一样”的格式强行合并，以降低格式变化风险。

### Phase 3: Integration

可考虑：

- 集成 RelaxCat，作为 Excel 文件分析/优化功能
- 独立 CLI / GUI 小工具
- 批量扫描目录中的 Excel 文件
- 优化前后文件大小、XF 数量和预计风险对比
- `--analyze` / `--optimize` / `--dry-run`
- 可选备份和恢复

## 安全要求

- 核心处理完全离线，不上传工作簿。
- 分析报告默认不得输出单元格值、公式文本、批注等业务内容。
- `.xlsm` 优化时必须保留 VBA project 及其他未知 OOXML parts，不重新生成整个工作簿。
- 修改前进行 OOXML 结构校验；修改后重新打开/校验关键关系和 style 引用。
- 对无法确认安全性的格式不自动合并。

## 技术方向

优先考虑直接处理 OOXML ZIP/XML，而不是通过 Excel COM 自动化逐单元格修改。这样更适合跨平台、批处理，也更容易精确控制 style ID 的重映射。

如果集成 RelaxCat，可将核心逻辑做成独立库，CLI / GUI 只作为调用层，以便未来其他项目复用。

## 状态

Idea / TODO。暂不确定最终归属项目；候选包括 RelaxCat 或独立工具。