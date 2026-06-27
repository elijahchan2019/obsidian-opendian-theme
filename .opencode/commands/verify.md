---
description: Opendian 改动的可视化验证清单（DEV_VAULT reload + 截图 + light/dark 双查）
---

刚改完 `theme.css`。带我走完整可视化验证，别跳步。

未提交改动概览：

!`git diff --stat`

改动命中的区段（帮我确认没误伤无关区段）：

!`git diff -U0 theme.css | grep -E '^@@' | head -40`

请按顺序执行并逐项汇报：

1. 确认 `../DEV_VAULT/.obsidian/appearance.json` 的 `cssTheme` 是 `Opendian-dev`；
   不是就提示我切换。symlink 已挂载，改动已生效，只需在 Obsidian 里 `Cmd+R` reload。
2. 列出本次改动**应该肉眼检查**的 DEV_VAULT 测试文档（依据改动区段，从
   `表格测试.md / 按钮体系测试.md / 可访问性测试.md / Anthropic Design Reference.md`
   中挑相关的）。
3. 提醒我分别在 **light** 和 **dark** 外观下各看一遍——重点是改了颜色/token 的地方。
4. 若改动涉及 §18 MOBILE，提醒用窄窗口（<600px）验证。
5. 给一句总结：这次改动需要重点盯的视觉风险点是什么。

$ARGUMENTS
