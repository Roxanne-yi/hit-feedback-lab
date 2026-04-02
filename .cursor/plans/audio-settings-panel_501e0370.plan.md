---
name: audio-settings-panel
overview: 在现有单文件 `index.html` 中新增独立的“音效设置面板”（非 Debug 面板），提供武器选择、音量控制与高级资源路径，并与 Attack/Auto-Hit 逻辑和一组武器 Smart Presets 联动。
todos:
  - id: locateAttackAndRangeHooks
    content: 定位普通攻击/技能 range 与命中结算入口，确定武器 preset 的参数落点（range、速度、hit-stop、曲线）。
    status: completed
  - id: addAudioUI
    content: 在 `index.html` 顶层新增 `#btnAudio` 与 `#audioPanel` DOM 结构，并添加 scoped CSS（尺寸、半透明、blur、边框、过渡、dismiss）。
    status: completed
  - id: implementAudioBus
    content: 实现 `AudioBus`（weapon+volume+paths）与“新音效中断旧音效”的 play 行为，接入 Attack/Auto-Hit 调用点。
    status: completed
  - id: weaponPresets
    content: 实现 `applyWeaponPreset`，将枪械/短剑/钝器映射到 range、速度倍数、hit-stop、曲线预设与 muzzle flash（最小可用实现）。
    status: completed
  - id: verifyUX
    content: 跑通 hit-and-run + 连击 + Auto-Hit 场景，确认 UI/音效/预设联动行为一致。
    status: completed
isProject: false
---

## 目标与约束

- 目标：新增一个独立于 Debug 面板的 Audio Settings Panel：右上角 `🔊` 图标按钮切换显示；面板约 220×180，暗色半透明+blur+细边框；包含武器选择、音量滑杆、（可折叠）资源路径输入；播放策略为“新音效中断旧音效”（每次 play 将 `currentTime=0`）。
- 约束：当前工程是单一 `[e:\siqi.yi\agenticDesigner\ingameEnvironment\index.html](e:\siqi.yi\agenticDesigner\ingameEnvironment\index.html)`，无 React 运行时；因此用原生 JS 以“组件化模块（可 mount/unmount、封装 state、scoped CSS）”实现。

## 代码落点梳理（用于联动）

- 右上角现有按钮：`#btnSettings`（用于打开/关闭 Debug 面板）。
- 攻击/自动连击入口：`performAttackOnce()`、`beginAttackHold()`、`attackRepeatTick()` 相关逻辑在 `index.html` 的 Phase 2/3 HUD Attack 段。
- 攻击命中范围：`tryHitEnemy(range, damage, label)` 以及 `SKILL_RANGES[...]`（如 Skill3 使用 `SKILL_RANGES[2]`）。
- Hit-stop：常量 `HIT_STOP_MS` 与攻击返回延迟计算（`returnDelay`）以及飘字触发的 `floatingTextHitStopUntil`。
- 曲线预设：`hitFeedbackParams.floatingTextPreset`、以及 `motion.speedCurve/scale.sizeCurve/fade.alphaCurve` 的 `stylePreset` 与 `setCurveFromDisplay`。

## 具体实现方案

### UI：新增图标与浮层面板（scoped CSS）

- 在 `#btnSettings` 旁新增 `#btnAudio`（文本为 `🔊`），并与 Settings 按钮共享右上角布局风格（尺寸/边框/hover/active），但视觉更“轻”。
- 新增 `#audioPanel` 浮层（`position: fixed`），默认隐藏；显示时使用 `opacity/transform` 过渡；背景 `rgba(20,20,20,0.85)` + `backdrop-filter: blur(10px)`，边框 1px。
- 面板交互：
  - 点击 `#btnAudio` 切换显示；
  - 点击面板外区域 dismiss；
  - 面板内右上角 `X` 关闭；
  - `Advanced` toggle 控制资源路径输入（默认折叠）。

### 控件：武器、音量、资源路径

- 武器选择 `<select>`：`枪械/短剑/钝器`。
- 音量 `<input type="range">`：0–100；映射到 0.0–1.0，并应用到音效引擎。
- 资源路径：默认使用你给定的本地路径：
  - `assets/sound/gun_fire.mp3`
  - `assets/sound/sword_slash.mp3`
  - `assets/sound/hammer_hit.mp3`
  Advanced 下提供 3 个 `<input type="text">` 允许临时改路径（仅影响当前 session，或可选持久化到 `localStorage`）。

### Audio Engine：New Interrupts Old

- 实现一个小型 `AudioBus` 模块（同文件内 IIFE/类均可），维护：
  - `weaponType`
  - `volume`（0–1）
  - 每种武器对应一个 `HTMLAudioElement`（或按需创建并缓存）
- `play()` 行为：
  - 取当前武器 audio；设置 `volume`；执行 `pause()`（可选）→ `currentTime=0` → `play()`；失败（用户手势限制）时静默或 console warn。

### 全局联动：Attack / Auto-Hit

- 在 `performAttackOnce()`（以及 `applySlashHitOnly()` 这种“仅结算”分支，若存在）里插入对 `AudioBus.play()` 的调用，确保：
  - 单击 Attack、长按连击、Auto-Hit 都会走同一音效触发路径。
- 保证触发点在“真正结算一次攻击/命中”时调用，避免 UI 点击但无攻击结算时误响。

### Smart Presets：武器切换驱动模拟逻辑

- 设计一个 `applyWeaponPreset(type)`：
  - **枪械**：
    - 设定攻击 range=15（落点：用于 `tryHitEnemy` 的 range 或 `SKILL_RANGES`/普通攻击 range）；
    - 曲线预设切为更“陡”的观感（方案：设置 `hitFeedbackParams.floatingTextPreset` 或直接对 speed/scale/alpha 的 `stylePreset` 调 `setCurveFromDisplay`，避免侵入自定义 spline）；
    - 生成 muzzle flash（最小实现：在角色手/角色前方创建一个短寿命 `THREE.Sprite`/`Plane`，绑定到角色当前朝向与位置偏移，数十毫秒淡出）。
  - **短剑**：
    - range=2.5；
    - 攻击速度 1.5x（方案：把 `DASH_DURATION_MS/RETURN_DURATION_MS` 等 const 重构为可变基值，并以 multiplier 参与 tween duration；或新增 `attackSpeedMultiplier` 并在创建 tween 时使用 `duration / multiplier`）。
  - **钝器**：
    - range=2.5；
    - 攻击速度 0.8x；
    - 启用/加重 Hit-stop（方案：将 `HIT_STOP_MS` 从 const 改为可配置值，例如 `hitFeedbackParams.hitStopMs` 或 `attackHitStopMs`，并在 returnDelay 计算中使用）。

## 修改点清单（文件）

- 仅修改 `[e:\siqi.yi\agenticDesigner\ingameEnvironment\index.html](e:\siqi.yi\agenticDesigner\ingameEnvironment\index.html)`：
  - HTML：新增 `#btnAudio`、`#audioPanel` 结构。
  - CSS：新增 `#btnAudio` 与 `#audioPanel` scoped 样式与过渡。
  - JS：新增 `AudioSettingsPanel` 模块（state+render+events）、`AudioBus`、`applyWeaponPreset`，并在攻击结算点挂钩。

## 验证方式（实现后）

- UI：右上角 `🔊` 与 `⚙` 并排；面板出现/消失动画；外部点击关闭；Advanced 正常折叠。
- 音效：连击时每次触发都从 0 播放（不会叠加、不会继续上一次尾音）。
- 联动：切换武器立即影响 range/速度/hit-stop/曲线，并在攻击时听到对应音效；枪械有 muzzle flash。

