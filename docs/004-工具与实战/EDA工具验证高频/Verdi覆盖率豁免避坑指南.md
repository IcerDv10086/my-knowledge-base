# Verdi 覆盖率豁免避坑指南

> 来源：<https://www.cnblogs.com/cmyxjcc/p/19489535>
> 说明：按原文风格整理，保留踩坑过程与处理语气。

今天在 Verdi 里踩了一个很典型的覆盖率豁免坑。项目本身体量不算大，但离 Tape-out 越近 RTL 变化越频繁。前一轮已经整理好的 `.el`（Exclusion List）在新数据库上一套，结果大量失效。

如果你也在项目后期遇到 Waiver “看起来有、实际上不生效”的情况，这篇可以直接拿来对照处理。

## 1. 为什么 Waiver 会失效？

最常见的原因有三类：

- RTL 层次变化：模块被挪位置、实例名变了。
- 代码逻辑变化：同一路径对应的签名（signature）变化。
- 旧 Waiver 直接复用到新 coverage database。

这时 Exclusion Manager 里会出现 unresolved 异常图标，说明旧规则与新数据库出现映射不一致。

## 2. 异常图标的含义与处理建议

![Exclusion Manager 异常示例 1](assets/verdi-waiver-icon-example-1.png)

![Exclusion Manager 异常示例 2](assets/verdi-waiver-icon-example-2.png)

常见三类异常：

- 绿色问号（Matches Signature）
    - 通俗理解：逻辑像是同一个点，但路径位置变了。
    - 处理建议：数量少可 remap；为减少历史包袱，通常建议 reject 后在新路径重建。
- 红色问号（Mismatch Signature）
    - 通俗理解：位置变了，代码也变了。
    - 处理建议：直接 reject，旧 waiver 不再可信。
- 红色感叹号（Unmappable）
    - 通俗理解：旧目标已不存在或不可映射。
    - 处理建议：直接 reject。

一句话：绿色可以讨论，红色基本都应该重建。

## 3. 耗时数小时排查的“隐形坑”

最容易让人误判的一点是：界面看着处理完了，第二天回归报告却几乎没变化。

常见根因：

- 在 Exclusion Manager 里直接覆盖旧 `.el`，但目标文件可能只读、路径不对，或被工具版本细节影响，导致并未真正写入。

避坑动作：

1. 处理完异常项后，不要直接覆盖旧文件。
2. 必须 `Save As...` 到新文件，例如 `waivers_v2_fixed.el`。
3. 在回归脚本/Makefile 中切换到新 waiver 路径。
4. 异常项过多时，先按 Status 分组，再批量 reject，效率更高。

## 4. 总结

- Waiver 是“随 RTL 演进持续维护”的产物，不是一次性文件。
- 每次 RTL 大改后，先清 unresolved，再看覆盖率分数。
- 如果只记一个动作，就记住 `Save As New File`。
