# AGENTS.md — Opendian

Obsidian theme. Single-file CSS, no build step. Aesthetic: **OpenCode / terminal**
— mono-forward type, sharp consistent radii, monochrome accents, flat tinted
states (no drop shadows). When in doubt, choose the more restrained, more
"developer-tool" option.

## 纪律（必须遵守）

1. **只改 `theme.css`，且只在本目录（`Opendian-dev`）。** 这是 `dev` 分支的工作区。
   - 旁边的 `../Opendian` 是 **`main` 分支**（发布产物）。**禁止直接编辑**它。
   - 发布流程是 `dev` → `main` 的合并，不是手动拷贝文件。
2. **改前先读。** 这是 2800+ 行的单文件，定位到目标区段（见下表）再动手，不要整文件重排。
3. **最小改动。** 不重构无关代码、不加装饰性注释、不"顺手优化"未要求的部分。
4. **改完即验**（见下）。纯 CSS 看不出对错，必须肉眼确认。
5. Obsidian 约定：**优先用官方 CSS 变量**（`--background-primary`、`--text-normal`、
   `--interactive-accent`…），自造 token 用 `--oc-*` 前缀。
6. **light / dark 双覆盖**：design token 分两套（`:root` 亮色 + `.theme-dark`）。
   改颜色相关的东西，两套都要顾到，否则暗色模式会破。

## 可视化验证回路

DEV_VAULT 已通过 symlink 把本主题挂进 `../DEV_VAULT/.obsidian/themes/Opendian-dev`，
改完 `theme.css` 后：

1. 在 Obsidian 里 reload（`Cmd+R`）或在外观设置里切走再切回主题。
2. 打开 `../DEV_VAULT` 里的测试文档对照：`表格测试.md`、`按钮体系测试.md`、
   `可访问性测试.md`、`Callout`/`Anthropic Design Reference.md` 等。
3. 桌面截图比对前后（Obsidian 是 Electron，浏览器类 MCP 用不上，用系统截图）。
4. 确认 light 和 dark 两种外观都正常。

## theme.css 区段地图

行号是近似值，**大改后会漂移**——重新生成用：
`awk '/====================/{getline t; if(t!~/=/) printf "%d: %s\n", NR, t}' theme.css`

| 行号 | 区段 |
|------|------|
| 16   | OPENCODE DESIGN TOKENS（`:root` 亮色 ~20，`.theme-dark` ~201：调色板 / 语义别名 / `--oc-*` 短 token / callout / 圆角 / 字体栈） |
| 377  | 1. BASE（`body`、prose 节奏） |
| 418  | 2. SIDEBAR & LOGO（左栏、vault 双色 wordmark、ribbon、侧栏图标对齐、设置图标） |
| 786  | 3. TABS（沉浸式单栏 tab，浮动 pill 几何，pin 标记） |
| 985  | 4. HEADINGS & TYPOGRAPHY |
| 1057 | 5. TABLE（关掉原生斑马纹、行 hover、紧凑表格） |
| 1137 | 6. CALLOUT（Starlight aside 风：中性面 + 语义色轨/标题/图标） |
| 1349 | 7. CODE BLOCKS（语言标签栏、复制按钮——刻意禁用） |
| 1457 | 8. CHECKBOXES & LISTS（普通勾选变量驱动，其余任务态自定义） |
| 1637 | 9. SETTINGS & MODALS（toggle pill、buttons、select/dropdown） |
| 1905 | 10. TAGS |
| 1951 | 11. RIGHT SIDEBAR / OUTLINE / BACKLINKS |
| 2130 | 12. SCROLLBARS |
| 2157 | 13. PROMPT / COMMAND PALETTE |
| 2211 | 14. TOOLTIPS / MENUS / DROPDOWNS（含 notice/toast 终端气泡） |
| 2300 | 15. STATUS BAR & TITLE BAR |
| 2394 | 16. SYNTAX HIGHLIGHTING（基础回退） |
| 2477 | 17. EDITOR / SOURCE MODE PARITY（源码模式：标题、格式标记、链接、inline code、选区） |
| 2746 | 18. MOBILE / NARROW LAYOUT（`max-width: 600px`） |

## 视觉品味（CSS 通用，迁移自 taste-skill）

写/审样式时主动规避 AI 设计 tells——本主题是终端/OpenCode 美学，更要克制：

- **不用纯黑/纯白。** 用 off-black、charcoal 这类带微色温的中性色。
- **强调色克制：最多一个，饱和度压住。** 禁 "AI 紫/霓虹蓝" 发光渐变。本主题是 monochrome 取向，accent 单一，别引入抢戏的彩色。
- **层级靠字重+颜色，不靠一味放大字号。**
- **状态用扁平染色填充（flat tinted fill），不用阴影/外发光。** 这正是本主题 active tab、icon hover 的既定做法，保持一致。
- **动画只动 `transform` 和 `opacity`**，绝不动 `top/left/width/height`——避免重排掉帧。
- **圆角走统一的 radius scale**（§TOKENS 已定义），别每处自造角值。
- **色板全局统一**，颜色走 `--oc-*` / 官方变量，别在组件里散写魔法色值。

## 提交前

- `manifest.json` 的 `version` 是否需要 bump（`versions.json` 维护 minAppVersion 映射）。
- light + dark 都验过。
- 没有顺手改到无关区段。
