# 网站设计与功能优化计划

## Summary

本计划用于优化当前生日网站的页面设计、交互体验、功能稳定性、可访问性和代码可维护性。项目现状是纯静态、无构建、无框架的生日互动网站，核心由 `index.html`、`jingxi.html`、`photo.html`、`admin.html` 和 `assets/` 中的图片资源组成。

本次优化应以 `index.html` 和 `jingxi.html` 为主，不引入 React、Vue、Tailwind、Vite 或其他构建链，不拆掉现有自包含结构。优化目标是在保留当前核心体验的基础上，让网站更精致、更稳定、更适合移动端和键盘访问。

需要保留的核心体验包括：

- `index.html`：粒子文字、照片粒子、蛋糕搭建、麦克风吹蜡烛、祝福打字机、留言板、主题切换。
- `jingxi.html`：心形祝福弹窗、普通祝福弹窗、Canvas 烟花、心形烟花、最终祝福卡、二维码弹窗、留言板、主题切换。

## Current State Analysis

### 技术结构

当前项目没有 `package.json`、测试框架、构建工具或前端框架。页面样式和脚本都以内联方式写在 HTML 文件中，这符合现有仓库规则，应继续保持。

`AGENTS.md` 和 `CLAUDE.md` 均说明该网站是生日祝福网站，并明确两个主要页面：

- `index.html` 是主页面，包含粒子文字、蛋糕场景、吹蜡烛检测和祝福揭示。
- `jingxi.html` 是惊喜页，包含心形弹窗祝福和烟花结尾。

### 页面设计现状

当前视觉风格以浅米色背景、蓝色强调色、玻璃拟态卡片和柔和动画为主。整体温柔、可爱，但存在以下问题：

- 蓝色玻璃卡片、弹窗、按钮、输入框的样式重复度较高，层次略单一。
- 中文标题主要使用系统字体，仪式感不够强。
- `jingxi.html` 中粉色按钮和蓝色主视觉同时出现，强调色略不统一。
- 深色模式依赖较宽泛的 `[class*="..."]` 选择器和 `!important`，后续维护风险高。
- 背景质感偏平，缺少纸感、微纹理或细腻的空间层次。

### 功能现状

现有核心功能价值较高，不应重写：

- `index.html` 的粒子文字、照片粒子、蛋糕搭建、吹蜡烛和祝福揭示是主记忆点。
- `jingxi.html` 的心形弹窗、普通祝福弹窗、烟花和最终祝福卡是第二层高潮。

但有几个功能稳定性问题需要优先处理：

- `jingxi.html` 烟花阶段计数疑似使用全局发射计数对比每阶段上限，可能导致后续阶段不再继续发射，心形烟花不稳定出现。
- `index.html` 麦克风授权成功但用户吹不灭时，缺少清晰的手动兜底路径。
- `index.html` 吹灭蜡烛后停止了 `MediaStream`，但没有明确关闭 `AudioContext`。
- 留言功能在前端混淆并还原 GitHub token，这不能作为安全机制；若网站公开部署，这是安全边界风险。

### 交互与可访问性现状

当前页面已有 hover、active、disabled、toast、弹窗遮罩关闭和移动端媒体查询，但还不够完整：

- 部分可点击元素使用 `div` 或 `span`，缺少原生按钮语义。
- 主题切换控件不是语义化按钮，缺少明确状态。
- 弹窗缺少 `role="dialog"`、`aria-modal`、标题关联和焦点管理。
- 按钮、关闭按钮、主题切换、跳过按钮等缺少统一 `:focus-visible`。
- `index.html` 的 viewport 使用 `user-scalable=no`，不利于低视力用户。
- 动态效果很多，但没有 `prefers-reduced-motion` 降级策略。

### 代码质量现状

`index.html` 中存在一个明显结构问题：`theme-toggle` 位于 `</head>` 和 `<body>` 之间，应移动到 `<body>` 内。两页还存在较多内联 `style=""`，不利于主题和状态统一。

两个主要页面的留言板逻辑有重复，但考虑到项目当前强调自包含页面，不建议默认拆成外部 JS 文件。更合适的方式是在各自页面内部整理结构、统一命名和状态处理。

## Proposed Changes

### `index.html`

#### 修复 HTML 结构

将当前位于 `</head>` 与 `<body>` 之间的 `#themeToggle` 移入 `<body>` 内，并改为语义化按钮。

建议目标：

```html
<button class="theme-toggle" id="themeToggle" type="button" aria-label="切换深浅主题" aria-pressed="false">
```

这样做的原因是当前 HTML 结构无效，且主题切换是可交互控件，应具备按钮语义、键盘访问能力和状态表达。

#### 移除移动端缩放限制

将 viewport 从：

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no" />
```

改为：

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
```

这样保留响应式布局，同时允许用户缩放页面。

#### 优化吹蜡烛兜底流程

保留 `enableBlowDetect()` 的麦克风检测逻辑，不重写音频分析流程。新增统一完成函数，例如 `completeCandleWish()`，让吹灭成功、麦克风拒绝后的跳过、麦克风已启动但用户吹不灭时的手动完成入口，都走同一条收束路径。

实施要点：

- 新增 `wishCompleted` 状态，防止重复触发。
- 吹灭成功后统一隐藏火焰、停止粒子文字、进入祝福卡。
- 拒绝麦克风后显示跳过按钮。
- 麦克风已启动后也允许出现“手动完成愿望”入口。
- 成功或跳过后停止 `MediaStream` 并关闭 `AudioContext`。

这样可以避免用户卡在吹蜡烛场景，也避免重复进入祝福页或重复播放打字机祝福。

#### 改善可点击元素语义

优先把以下可点击元素改成 `<button>`，或在不便修改结构时补齐 `role`、`tabindex` 和键盘事件：

- `#enterLink`
- `#skipBlow`
- `#themeToggle`
- 留言板关闭按钮
- 管理入口按钮

按钮改造后需要保留原视觉样式。对于原本依赖 `display:none` 的控件，可以改用 `hidden` 属性或继续用现有显示逻辑，但脚本要同步调整。

#### 增加弹窗可访问性

为留言板弹窗补充：

- `role="dialog"`
- `aria-modal="true"`
- `aria-labelledby`
- 打开后聚焦到弹窗或第一个可操作元素
- Esc 关闭弹窗
- 关闭后焦点回到触发按钮

现有点击遮罩关闭行为应保留。

#### 增加统一焦点样式

在页面 CSS 中加入统一的键盘焦点样式：

```css
:where(button, a, input, textarea, [role="button"], [tabindex]):focus-visible {
    outline: 2px solid var(--accent);
    outline-offset: 3px;
    box-shadow: 0 0 0 6px rgba(59, 130, 246, 0.14);
}
```

如果调整颜色变量时新增 `--accent-rgb`，则把阴影中的 RGB 改为变量。

#### 优化视觉变量与排版

保留当前米色和蓝色方向，但降低“通用蓝色玻璃”的模板感。建议建立更细的变量：

```css
:root {
    --bg: #eee7dd;
    --paper: rgba(255, 252, 247, 0.68);
    --paper-strong: rgba(255, 252, 247, 0.86);
    --ink: #243044;
    --muted: #6f7a8b;
    --accent: #3f7fbe;
    --accent-rgb: 63, 127, 190;
    --candle: #d69b43;
    --heart: #c96f8f;
    --line: rgba(63, 127, 190, 0.16);
    --shadow-soft: 0 24px 70px rgba(86, 102, 126, 0.12);
}
```

中文标题建议使用更有仪式感的系统衬线字体栈：

```css
--font-display: "Songti SC", "Noto Serif CJK SC", "Source Han Serif SC", "STSong", serif;
--font-body: "PingFang SC", "Microsoft YaHei", "Segoe UI", system-ui, sans-serif;
```

应用到祝福标题、倒计时标题、卡片标题等，不要影响粒子文字 canvas 的字体回退逻辑。

#### 增加背景细节

用 `body::before` 或类似低透明伪元素增加轻微纸感纹理，避免纯平背景。该伪元素必须：

- `position: fixed`
- `pointer-events: none`
- 不遮挡 canvas、弹窗、烟花和按钮
- 在深色模式下不产生脏污感

#### 支持减少动态效果

增加 `@media (prefers-reduced-motion: reduce)`，降低弹幕、彩纸、闪烁和非核心装饰动画。不要直接跳过生日主流程，而是降低动画频率、缩短转场、减少粒子数量。

JS 中可读取：

```js
const reduceMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
```

用于减少彩纸、弹幕或其他高频装饰数量。

### `jingxi.html`

#### 修复烟花阶段计数

保留 `heartCurve()`、`startFireworks()`、`explodeRocket()` 和 `fireworkLoop()` 的整体职责，不重写烟花系统。重点修复每阶段发射上限判断。

当前风险是用全局 `fwLaunchCount` 与当前阶段的 `maxForPhase` 对比，阶段 0 发射数超过后，后续阶段可能不再继续发射。建议新增每阶段计数：

```js
let fwPhaseLaunchCounts = [0, 0, 0, 0];
```

在 `startFireworks()` 中重置：

```js
fwPhaseLaunchCounts = [0, 0, 0, 0];
```

在发射判断中使用当前阶段计数：

```js
if (
    fwPhaseLaunchCounts[fwPhase] < maxForPhase &&
    timestamp - fwLastLaunchTime > interval * 1000
) {
    fwRockets.push(createRocket());
    fwLastLaunchTime = timestamp;
    fwLaunchCount++;
    fwPhaseLaunchCounts[fwPhase]++;
}
```

心形烟花触发条件可以继续依赖总发射数和阶段，也可以调整为“进入阶段 2 且尚未触发”后在合适位置触发。必须确保心形烟花稳定出现。

#### 统一惊喜页强调色

保留粉色作为心形、祝福和烟花情绪点缀，但不让粉色成为常规按钮的第二主色。`next-btn` 可继续有温柔粉色倾向，但应降低饱和度、透明度，并与蓝色主变量协调。

目标是让 `index.html` 和 `jingxi.html` 看起来像同一套生日设计系统，而不是两个拼接页面。

#### 改善弹窗语义和焦点管理

为二维码弹窗、留言板弹窗、最终祝福卡相关可交互区域补充必要语义：

- `role="dialog"`
- `aria-modal="true"`
- `aria-labelledby`
- 关闭按钮使用 `<button>`
- 打开时聚焦，关闭时恢复焦点
- Esc 可关闭可关闭弹窗

心形祝福弹窗和普通祝福弹窗主要是动态展示内容，不一定都要做成可交互 dialog，但应避免抢焦点。最终祝福卡如果包含按钮，则按钮必须可键盘访问。

#### 修正可点击控件

优先处理：

- `#qrClose`
- `#msgClose`
- `#themeToggle`
- 复制链接或保存图片相关控件
- 留言按钮和关闭按钮

所有按钮保留原有视觉，但补齐 hover、active、focus-visible 和 disabled 状态。

#### 深色模式整理

减少大范围 `[class*="card"]`、`[class*="popup"]`、`[class*="modal"]` 与 `!important` 的使用。改为通过明确变量和关键组件选择器控制深色模式。

建议定义深色变量：

```css
body.dark {
    --bg: #12151c;
    --paper: rgba(28, 32, 42, 0.72);
    --paper-strong: rgba(34, 39, 51, 0.9);
    --ink: #f4efe7;
    --muted: #aeb7c8;
    --line: rgba(255, 255, 255, 0.14);
    --shadow-soft: 0 24px 70px rgba(0, 0, 0, 0.32);
}
```

这样能减少误伤未来组件，也能让二维码、留言板、祝福卡和按钮在深色模式下更一致。

#### 支持减少动态效果

增加 CSS 和 JS 双层降级：

- CSS 降低闪烁、弹跳、星星、背景装饰动画。
- JS 减少烟花粒子数、弹窗数量或动画密度。
- 保留“开启祝福 → 祝福弹窗 → 烟花 → 最终卡片”的完整流程。

### `photo.html`

本次计划不默认修改 `photo.html`，除非执行过程中发现 `index.html` 的“进入拍照留念”链接或交互语义必须同步调整。若需要同步，只做低风险修正：

- 确认 `index.html` 跳转目标仍有效。
- 不改变拍照页业务逻辑。
- 不引入新依赖。

### `admin.html`

本次计划不默认修改 `admin.html`。如果 `index.html` 的管理入口语义化影响跳转或密码入口，需要只做兼容性调整，不改变管理页功能。

### `assets/`

不需要新增图片资源。可继续使用：

- `assets/birthday-friends.jpg`
- `assets/birthday-friends-particle.jpg`
- `qrcode.png`

背景纸感、焦点、按钮、卡片层次应优先通过 CSS 实现，避免引入额外文件。

## Assumptions & Decisions

- 继续保持纯 HTML / CSS / JavaScript，不引入框架、不引入构建工具。
- 不把 `index.html` 和 `jingxi.html` 拆成外部 CSS / JS 文件，因为当前仓库规则强调两个页面自包含。
- 不重写粒子系统、吹蜡烛系统、弹窗队列和烟花系统，只做局部增强。
- 留言功能的 GitHub token 暴露问题需要明确记录为安全边界；在用户没有要求增加服务端代理前，不默认改变静态架构。
- 视觉方向继续保持“温柔生日、米色纸感、蓝色主调、少量烛光与心形点缀”，不改成完全不同风格。
- 优先修复影响体验闭环的问题，再做视觉细节。
- 所有交互控件改造必须保持现有外观和主要行为。
- 减少动态效果模式下不跳过核心祝福流程，只降低高频动画和装饰密度。

## Verification steps

### `index.html`

执行后需要直接在浏览器打开 `index.html` 验证：

- 页面可以正常加载，没有控制台语法错误。
- 倒计时或入口流程正常显示。
- 粒子文字、照片粒子、线性过渡仍能按顺序执行。
- 蛋糕搭建动画、蜡烛和火焰正常出现。
- 允许麦克风后，吹气进度条变化，达到阈值后进入祝福卡。
- 拒绝麦克风后，跳过入口出现并可进入祝福卡。
- 麦克风已授权但用户不吹或吹不灭时，手动完成路径可用。
- 祝福打字机正常播放，不重复触发。
- 留言板可以打开、关闭、提交，失败时有明确反馈。
- 主题切换可点击，也可通过键盘 Tab 聚焦并触发。
- 所有主要按钮和关闭按钮有清晰 `focus-visible` 样式。
- 移动端页面可缩放，布局不破。
- 深色模式下文字、弹窗、输入框、按钮对比正常。
- 开启减少动态效果偏好时，流程仍可完成，装饰动画明显减少。

### `jingxi.html`

执行后需要直接在浏览器打开 `jingxi.html` 验证：

- 点击“开启祝福”后控制面板正常隐藏。
- 心形祝福弹窗完整出现并淡出。
- 普通祝福弹窗队列正常出现。
- 弹窗结束后烟花启动。
- 烟花按多个阶段持续发射，不只停留在第一阶段。
- 心形烟花稳定出现。
- 烟花结束后最终祝福卡显示。
- 保存图片、二维码弹窗、留言板仍正常工作。
- 二维码弹窗和留言板可通过关闭按钮、遮罩和 Esc 关闭。
- 主题切换、保存、留言、关闭等控件可键盘访问。
- 深色模式下二维码弹窗、留言板、祝福卡和按钮保持足够对比。
- 减少动态效果模式下仍能完成“弹窗 → 烟花 → 最终卡片”的流程。

### 回归检查

需要额外检查：

- `index.html` 到其他页面的链接仍有效。
- `jingxi.html` 不因烟花计数修复产生无限动画。
- 页面没有新增外部依赖。
- 控制台没有未捕获异常。
- 没有删除现有生日祝福内容、照片资源、二维码资源和留言功能。
- `AGENTS.md` 与 `CLAUDE.md` 中要求的核心页面职责仍被满足。
