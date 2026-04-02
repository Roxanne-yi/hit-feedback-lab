**第一阶段的执行任务清单**

---

## 第一阶段：3D 基础环境与手机适配 3C 系统

### 1. 场景基础配置 (Scene Setup)

* **引擎选择：** 使用 Three.js + Cannon.js (物理引擎)。
* **环境：** 创建一个 50x50 的平面作为地板。背景色设为深灰色 (`#1a1a1a`)。
* **角色占位符：** * **身体：** 一个高 1.8m 的胶囊体 (Capsule)。
* **正面标识：** 在胶囊体正面添加一个小的方块或发光条，用于辨别朝向。


* **预留接口：** 创建一个 `CharacterManager` 类，其中包含 `loadModel(url)` 方法，方便后续将胶囊体替换为 `.glb` 模型。

### 2. 移动端 UI 适配 (Mobile UI)

* **左侧摇杆 (Joystick)：** 使用 HTML/CSS 绘制一个圆形的虚拟摇杆。
* **逻辑：** 监听 `touchstart` 和 `touchmove`。计算摇杆中心到手指的向量，并将其归一化。
* **同步：** 将摇杆向量实时传递给 3D 物理引擎，驱动角色在 $x-z$ 平面移动。


* **右侧旋转视角 (Camera Orbit)：** * 在屏幕右侧区域添加透明遮罩层。
* 通过监听水平滑动的 $\Delta x$ 来旋转摄像机绕角色的 $y$ 轴角度；通过 $\Delta y$ 控制摄像机的俯仰角。



### 3. 第三人称摄像机 (Third-Person Camera)

* **相机跟随：** 摄像机始终维持在角色后方的一个偏移量 $D = (0, 2.5, -4)$。
* **平滑插值：** 使用 `lerp` 算法让相机跟随更顺滑，避免生硬的吸附感。

### 4. 基础动作 (Jump & Crouch)

* **跳跃：** 在右下角放置一个“Jump”按钮。点击时给物理胶囊体施加一个垂直向上的冲量 (Impulse)。
* **下蹲：** 点击“Crouch”按钮时，将胶囊体的高度缩放为原来的 0.6 倍，松开后恢复。

---

## 给 Cursor 的 Prompt 指令（你可以直接复制）

> **Role:** You are a Senior Game Developer specialized in Three.js and mobile web optimization.
> **Goal:** Create a 3D Third-Person Demo using Three.js and a physics engine (like Cannon.js).
> **Requirements:**
> 1. **Scene:** A dark dungeon-style plane (dark grey) with a few boxes as obstacles.
> 2. **Character:** Use a white CapsuleGeometry as a placeholder. Add a `loadModel` method for future GLTF integration.
> 3. **Mobile Controls:** >    - Implement a virtual joystick on the bottom-left for movement.
> * Implement touch-swipe on the right side of the screen to rotate the camera.
> * Add two HTML buttons on the bottom-right for 'Jump' and 'Crouch'.
> 
> 
> 4. **Physics:** Use a physics engine to handle collision between the character and the ground/obstacles.
> 5. **Responsiveness:** Ensure the canvas and UI scales correctly for mobile browsers (portrait or landscape).
> 
> 
> **Output:** A single index.html with embedded script OR a clean modular structure (if using a bundler).

---

## Phase 1 验收清单（对照 Cursor Prompt）

开发完成后可用本清单自检，确认无遗漏。

### Cursor Prompt 五项

| 项 | 要求 | 验收 |
|---|------|------|
| 1 | **Scene:** 深色地牢风格平面 + 若干盒子障碍 | 50×50 地面、背景 `#1a1a1a`、若干障碍盒 + 物理 |
| 2 | **Character:** 白色 CapsuleGeometry 占位 + `loadModel` 预留 | 胶囊体 + `CharacterManager.loadModel(url)` |
| 3 | **Mobile Controls:** 左下摇杆、右侧滑动转视角、右下 Jump/Crouch | 摇杆 + 右侧触摸区 + 两个按钮（touch/click） |
| 4 | **Physics:** 角色与地面/障碍碰撞 | 物理引擎 world.step，地面 + 障碍 + 角色刚体 |
| 5 | **Responsiveness:** 移动端横竖屏适配 | viewport、canvas 全屏、resize、UI 百分比布局 |

### plan.md 第一阶段细项

- **场景：** 引擎 Three.js + Cannon、50×50 地板、背景 `#1a1a1a`、1.8m 胶囊体、正面标识（小方块/发光条）、`CharacterManager` + `loadModel(url)`。
- **移动端 UI：** 左侧圆形摇杆（touchstart/touchmove、归一化向量驱动 x-z）；右侧透明区（Δx 转 yaw、Δy 转俯仰）。
- **相机：** 偏移 D=(0, 2.5, -4)、lerp 平滑跟随。
- **基础动作：** Jump 按钮 → 垂直向上冲量；Crouch 按住 → 高度 0.6 倍，松开恢复。

### 输出

- 单文件 `index.html` 或模块化结构均可。
既然我们已经完成了基础的 3C 框架（角色、视角、控制），现在进入 **B部分：UI 布局与小地图同步系统**。

这一部分的核心是建立一套“游戏化”的界面，并实现 3D 世界坐标到 2D UI 空间的映射逻辑。

---

**第二阶段的执行任务清单**

---

## 第二阶段：UI 布局与小地图同步系统

#### 1. UI 全局布局 (UI Layout)

根据你提供的参考图，我们需要在 Canvas 之上覆盖一层响应式的 HTML/CSS UI 层。

* **左上角：** 玩家属性（角色 ID、血条/蓝条预留位、金币数显示）。
* **右上角：** 设置/菜单按钮。
* **左下角：** 虚拟摇杆（已在 A 部分实现，需优化视觉）。
* **右下角：** 攻击键（最大）、三个技能键（环绕）、跳跃与下蹲（较小）。
* **左侧中部：** 简易的任务引导或提示文字。

#### 2. 小地图系统 (Minimap System)

这是 B 部分的技术难点，我们需要实现一个实时的“上帝视角”缩略图。

* **实现方案：**
* **双摄像头方案：** 在 3D 场景上方放置一个垂直向下的正交摄像机（Orthographic Camera）。
* **渲染窗口：** 将正交摄像机的内容渲染到一个小的正方形 Canvas（或 Three.js 的 Scissor 区域）中，放在屏幕左上角。
* **实时同步：** * 小地图中心始终跟随玩家的 $x, z$ 坐标。
* 玩家在小地图上显示为一个“方向箭头”图标，其旋转角度同步 3D 玩家的朝向。




* **视觉风格：** 简化的暗灰色块代表墙壁，高亮圆点代表玩家。

#### 3. 攻击与技能按钮交互 (Action Buttons)

* **视觉反馈：** 按钮按下时要有明显的缩放（Scale down）和变色反馈。
* **冷却效果 (Cooldown)：** 预留技能转圈（SVG Mask 顺时针遮罩）的动画逻辑。
* **攻击连招逻辑预留：** 点击攻击键时，控制台先输出 `Action: Attack`，并触发一个计时器，用于后续判断二段攻击。

#### 4. 适配与响应式

* **安全区域 (Safe Area)：** UI 元素需避开手机屏幕的“刘海”和“底条”。
* **横竖屏切换：** 自动调整小地图和按钮组的大小。

---

### 给 Cursor 的 Prompt 指令

你可以将这段指令发送给 Cursor，继续在原有的代码基础上进行开发：

> **Role:** Senior Frontend Game UI Developer.
> **Task:** Add a HUD and Minimap system to the existing Three.js project.
> **Requirements:**
> 1. **Minimap Implementation:**
> * Create a secondary Orthographic Camera looking top-down at the player.
> * Display this view in a small square container at the top-left (e.g., 150px).
> * Ensure the minimap follows the player's X and Z position.
> * Add a specialized marker (an arrow) in the minimap to represent the player's heading.
> 
> 
> 2. **Game HUD Overlay:**
> * Use HTML/CSS (position: absolute) to create a gaming UI overlay.
> * **Top Left:** A placeholder health bar and a gold counter (e.g., "GAIN: 5,000").
> * **Bottom Right:** Arrange action buttons:
> * 1 Large Main Attack Button.
> * 3 Smaller Skill Buttons (S1, S2, S3) around the attack button.
> * Ensure these buttons work with `touchstart` events.
> 
> 
> 
> 
> 3. **Interactivity:**
> * When a skill button is pressed, trigger a console log: "Skill [N] Cast".
> * Implement a simple "Cooldown Overlay" (semi-transparent grey cover) that lasts for 2 seconds after a skill is clicked.
> 
> 
> 4. **Styling:** Use a dark, gothic/medieval aesthetic for buttons (dark borders, gold/white text, slightly transparent backgrounds).
> 
> 
> **Optimization:** Ensure the UI is responsive and doesn't interfere with the touch-swipe camera controls.

---
既然决定绕过模型资源问题，直接使用几何体（Capsule）来推进核心逻辑，那我们现在的重心就是打造**极致的“类王者荣耀”操作手感**。

以下是针对 **C部分：类王者荣耀技能交互与指示器系统** 的 PRD 补充，旨在让 Cursor 能够写出具有“工业级”手感的代码。

---

## 第三阶段：类王者荣耀（MOBA）技能交互系统

### 1. 核心操作逻辑：右手双重状态

在“类王者荣耀”模式下，技能按钮不仅仅是点击，它承载了“按下-拖动-瞄准-释放”的完整状态机。

* **状态 A：按下 (Touch Start)**
* 在角色足底生成**技能指示器**（扇形、直线或圆形）。
* 此时指示器默认指向角色当前正前方。


* **状态 B：拖动瞄准 (Touch Move)**
* **核心逻辑：** 技能指示器的旋转角度由**右手手指相对于按钮中心的偏移向量**决定。
* **视觉反馈：** 角色模型不旋转（或者仅头部扭向瞄准方向），但指示器随手指平滑转动。


* **状态 C：释放 (Touch End)**
* **取消机制：** 如果手指拖动到屏幕上方的“取消投掷”区域（通常是红色 X 区域），则销毁指示器，不触发技能。
* **释放逻辑：** 如果在有效范围内松开，执行技能逻辑（如发射投射物或播放攻击特效），指示器消失。



#### 2. 指示器视觉规格 (Indicator Visuals)

* **扇形 (Sector)：** 用于近战挥砍。
* 参数：半径 (Radius)、张角 (Angle, 如 90°)。
* 材质：半透明蓝色/金色，边缘有高亮描边。


* **直线 (Linear)：** 用于远程突刺/射击。
* 参数：长度 (Length)、宽度 (Width)。
* 材质：带有流动感的箭头纹理。



#### 3. 战斗事件飘字 (Floating Combat Text)

* 当按下攻击键时，在胶囊体上方生成随机位置的飘字。
* **物理效果：** 字迹向上跳动，带有微弱的重力下坠感，随后透明度变为 0 并销毁。

---

### 🚀 给 Cursor 的“类王者荣耀”手感专项指令 (Prompt)

请复制以下指令给 Cursor，它将直接构建出这套复杂的交互逻辑：

> **Role:** Professional MOBA Gameplay Engineer.
> **Task:** Implement a "Honor of Kings" (王者荣耀) style skill aiming system.
> **Requirements:**
> 1. **Joystick & Button Decoupling:** >    - Left Joystick controls character movement (already done).
> * Right Skill Button acts as a **"Directional Joystick"** when held down.
> 
> 
> 2. **Skill 1 (Sector Aiming):**
> * **On Press:** Show a 90-degree Sector mesh (radius: 5) at the player's feet.
> * **On Drag:** The sector should rotate based on the **drag offset** from the button center. (Map the 2D touch offset to the 3D X-Z rotation).
> * **On Release:** Execute a `castSkill()` function that logs the final direction and triggers a simple "Flash" effect on the sector before hiding it.
> 
> 
> 3. **Skill 2 (Linear Aiming):**
> * Similar to Skill 1, but use a thin rectangular plane (length: 10, width: 2) as the indicator.
> 
> 
> 4. **Cancel Zone:**
> * If the user drags the finger to the top of the screen (define a `cancel_zone`), change the indicator color to **RED** and do not cast on release.
> 
> 
> 5. **Visuals:** >    - Use `THREE.MeshBasicMaterial` with `transparent: true` and `opacity: 0.4` for indicators.
> * Ensure indicators stay flat on the ground (`y = 0.01`).
> 
> 
> 
> 
> **Refinement:** Make the rotation smooth using `lerp` or `slerp` for a premium feel.

---

### 🎨 推荐的“暗黑地牢”视觉色号（供 UI 调节参考）

为了让你的几何体 Demo 看起来更有地牢味，可以要求 Cursor 使用这些色号：

* **背景地面：** `#0a0a0a` (深渊黑)
* **角色占位符：** `#d4d4d4` (冷钢色)
* **技能指示器（正常）：** `#4a90e2` (灵能蓝) 或 `#d4af37` (古铜金)
* **技能指示器（取消）：** `#ff4d4d` (警戒红)
* **飘字颜色（暴击）：** `#ffcc00` (亮金)

## ⚔️ 胶囊体攻击动画优化方案

### 1. 瞬间位移与回弹 (Snappy Translation)

攻击时不应只是原地不动，通过逻辑模拟“踏步”或“出招冲力”。

* **冲刺偏移 (Forward Dash)：** 按下攻击键时，玩家模型在 $100ms$ 内沿朝向坐标系向前平移 $0.5 - 0.8$ 单位。
* **弹性回缩 (Elastic Recovery)：** 到达顶点后，使用 `Exponential.Out` 或 `Back.Out` 缓动函数在 $200ms$ 内回到原位。这种“快进慢退”的节奏感是打击感的关键。

### 2. 攻击形变 (Squash and Stretch)

利用 3D 缩放模拟蓄力和爆发的物理反馈。

* **蓄力压缩 (Squash)：** 在位移开始的一瞬间，将胶囊体的 `scale.y` 压缩至 $0.8$，`scale.x` 和 `scale.z` 略微扩大（保持体积守恒）。
* **爆发拉伸 (Stretch)：** 在位移最快的那一帧，将 `scale.y` 激增至 $1.4$，使胶囊体看起来像一根射出的利箭。
* **震动恢复：** 攻击结束后，让缩放比例像弹簧一样震动一两次再归位。

### 3. 视觉残留与“残影” (Visual Trails)

由于几何体很单薄，需要增加视觉占据空间。

* **位移残影 (Ghosting)：** 攻击瞬间，在玩家路径上克隆 2-3 个半透明的胶囊体占位符，分别以不同的 `opacity` 极速淡出。
* **攻击切片 (Swing Arc)：** 在玩家正前方瞬间生成一个半透明的**薄片扇形网格**（类似你之前的技能指示器，但更薄、颜色更亮），模拟武器挥砍的轨迹。

---

## 🚀 给 Cursor 的优化指令 (Prompt)

你可以直接将这段专项优化指令发给 Cursor：

> **Role:** Game Juice & Animation Specialist.
> **Task:** Enhance the "Attack" feel for the current Capsule player using code-based animations (Tweens).
> **Requirements:**
> 1. **Dash & Bounce:** When `attack()` is triggered, use `TWEEN.js` (or `GSAP`) to move the player mesh forward by 0.6 units in 0.1s, then bounce back to the original local position in 0.3s. Use an `EaseOut` curve.
> 2. **Squash and Stretch:** >    - At the start of the dash, set `scale` to `(1.2, 0.8, 1.2)`.
> * At the peak of the dash, set `scale` to `(0.8, 1.4, 0.8)`.
> * Return to `(1, 1, 1)` with an elastic effect.
> 
> 
> 3. **Weapon Swing FX:**
> * Create a simple `THREE.Mesh` with a `RingGeometry` (arc: 120 degrees) to simulate a sword swing.
> * This mesh should appear in front of the capsule, rotate 120 degrees rapidly, and fade out its opacity to 0 within 0.2s.
> 
> 
> 4. **Impact Freeze (Optional):** Add a tiny "Hit Stop" (pause the animation for 0.05s) if the attack logic detects a collision to simulate impact.
> 
> 
> **Goal:** Make the capsule look like it's lunging and swinging with weight and speed.

---


这份文档整合了你提供的**受击反馈功能架构图** 以及**界面布局参考**，旨在为 Cursor 提供一个结构清晰、逻辑严密的开发指南。

---

# 🎮 开发者文档：受击反馈 (Hit Feedback) Debug 模块

## 1. 界面与布局规范

* **屏幕切分**：页面左侧 **1/3** 为 Debug 参数控制面板（使用 `lil-gui`）；右侧 **2/3** 为 3D 游戏预览区域。
* **模块化切换**：在 Debug 面板顶部设置页签，当前默认锁定在 **“受击反馈” (Hit Feedback)** 模块。
* **自动测试循环**：进入该模块后，代表玩家的胶囊体每隔 **2 秒** 自动触发一次 `onHit` 逻辑，以便实时预览参数调整。

---

## 2. 功能架构与调试参数 (Debug Settings)

根据受击反馈架构图，Debug 面板需包含以下可调项：

### A. 血条反馈 (Health Bar Feedback)

| 调试项 | 控件类型 | 功能描述 |
| --- | --- | --- |
| **数值位置** | 下拉菜单 | 可选：**Top, Bottom, Left, Right**。根据选择调整伤害数值与血条的相对方位。 |
| **下降速度** | 滑动条 | 调节血条被扣除后的平滑缩减速度（Lerp Speed）。 |
| **下降颜色变化** | 色域选择 | 设置血条扣除瞬间的底色闪烁（如由绿变红或变白）。 |
| **反馈动效强度** | 滑动条 | 调节受击时血条本身的**抖动 (Shake)** 或**高亮 (Flash)** 频率。 |

### B. HUD 伤害跳字 (Floating Text)

| 调试项 | 控件类型 | 功能描述 |
| --- | --- | --- |
| **普通/暴击区分** | 逻辑切换 | **Normal**: 基础颜色与缩放；**Critical**: 1.5倍缩放、爆发式弹出动效。 |
| **颜色配置** | 色域选择 | 分别设置 `Normal Color` (默认白色) 与 `Critical Color` (默认亮金)。 |
| **跳字动效** | 下拉菜单 | 可选：**向上漂浮 (Float Up)**、**中心弹出 (Zoom Pop)**、**随机抖动 (Random Shake)**。 |
| **持续时间** | 滑动条 | 调节跳字从生成到完全消失的时间 (0.5s - 3s)。 |

### C. 状态与方位指示

| 调试项 | 控件类型 | 功能描述 |
| --- | --- | --- |
| **受击图标弹出** | 开关/菜单 | 模拟显示“眩晕”、“僵直”等状态图标。 |
| **被攻击方向 UI** | 开关 | 开启后，在胶囊体周围显示指向“攻击者”位置的红色 UI 箭头。 |

---

## 🚀 给 Cursor 的专项执行 Prompt

**请直接复制以下内容至 Cursor 聊天框：**

```markdown
### Task: Implement Hit Feedback Debug Module
Based on the provided architecture and UI layout, please build a professional Hit Feedback debugger.

1. **Environment Setup:**
   - Split screen: Left 30% for the Debug GUI (lil-gui), Right 70% for the Three.js Canvas.
   - Setup a 2-second auto-trigger loop: `setInterval(() => player.onHit(), 2000);`.

2. **Core Logic - Damage Types:**
   - On each hit, randomly choose between **Normal** (75%) and **Critical** (25%).
   - **Normal:** Use `normalColor`. Simple float-up animation.
   - **Critical:** Use `criticalColor`. Implement a "Pop" scale-up effect (0.1 to 1.5 back to 1.0) and screen shake.

3. **GUI Implementation (Health Bar & HUD):**
   - **Value Positioning:** Implement a dropdown to move the damage number to **Top, Bottom, Left, or Right** of the health bar.
   - **Visuals:** Add sliders for `dropSpeed`, `shakeIntensity`, `fontSize`, and `duration`.
   - **Color:** Use color pickers for health flash and text colors.

4. **Feedback Actions:**
   - When `onHit` triggers: 
     - Shake the health bar based on `shakeIntensity`.
     - Spawn floating text at the position defined by `valuePosition`.
     - Make the player Capsule flash its material color briefly.

Please provide a clean, modular implementation using Three.js and lil-gui.

```

---

### 💡 后续交互建议

* **验证方位映射**：当你在左侧切换 `Left` 或 `Top` 时，观察右侧的 HTML 元素是否正确应用了 CSS 的 `transform` 或 `flex` 布局偏移。
* **连击测试**：如果自动循环太慢，可以建议 Cursor 增加一个 **"Manual Hit"** 按钮，连续点击以测试多重跳字重叠时的排版效果。

收到，明白你的核心诉求：**我们要把“打击感”从简单的数值变化，变成一套可高度自定义的“动画物理引擎”**。

根据你确认的 3 点细节（预设+滑杆双轨、2D 屏幕平面位移、重叠/排队开关），我为你整理了这份 **Cursor 迭代开发计划**。这份计划将粉色部分的复杂动效拆解为四个逻辑模块，确保 Cursor 能有条不紊地完成。

---

### 🎨 HUD 飘字动效迭代开发计划

#### **第一阶段：UI 配置协议扩展 (The Config)**
**目标**：在 `hitFeedbackParams` 中建立完整的动画参数树，并同步到 Debug 面板。
* **位移配置 (`motion`)**：
    * `directionMode`: "Fixed" (8向预设) / "Manual" (滑杆控制 X/Y 偏移量)。
    * `stackingMode`: "Overlap" (重叠) / "Queue" (自动偏移排队，每个新字 Y 轴向上堆叠)。
    * `duration`: 动画总时长（单位：秒）。
* **缩放配置 (`scale`)**：
    * `preset`: "PopUp" (先大后小) / "Dive" (先小后大) / "Static" (无缩放) 等。
    * `intensity`: 缩放的最大/最小倍率滑杆。
* **消失配置 (`fade`)**：
    * `curve`: "Linear" / "BackOut" (回弹) / "EaseIn" / "Steps" (定格后闪现消失)。

#### **第二阶段：二维位移与排队逻辑 (The Movement)**
**目标**：将 3D 世界坐标投影转为 2D 动画矢量。
* **8 向位移算法**：
    * 建立方向向量表（例如：`UR: {x: 1, y: -1}` 代表右斜上）。
    * 计算最终屏幕坐标：$ScreenPos = ProjectedPos + (Vector \times Intensity \times Progress)$。
* **自动排队逻辑**：
    * 维护一个 `activeFloatingTexts` 队列。
    * 如果开启 `Queue`，新生成的飘字 $Y$ 偏移量需累加：$Offset = LastText.Y + TextHeight + Gap$。

#### **第三阶段：多维曲线引擎 (The Animation Engine)**
**目标**：重构 `TWEEN` 逻辑，支持独立的时间曲线。
* **非线性缩放**：
    * 使用 `TWEEN.Easing.Back.Out` 实现“弹出回弹”效果。
    * 使用 `TWEEN.Easing.Elastic` 实现“肉感震动”效果。
* **时间分段**：
    * 前 20% 时间执行 `Scale` 爆发。
    * 中间 60% 执行 `Motion` 平移。
    * 后 20% 执行 `Fade` 消失。

#### **第四阶段：手感专项调试 (The Juice)**
* **打击定格联动**：当飘字执行“爆发缩放”时，联动 `HIT_STOP_MS`。
* **颜色分级**：对齐架构图中的“普通/暴击/治疗”颜色预设。

---

### 🛠️ 给 Cursor 的核心指令 (可以直接粘贴)

> **Role:** Senior Game Animation & VFX Engineer.
> **Task:** Refactor the Floating Damage Text system to support advanced "Juice" and "Game Feel" settings based on the new architecture.
>
> **Core Logic to Implement:**
> 1. **2D Screen-Space Motion**: Floating texts should calculate their trajectory in 2D Screen Space after the world-to-screen projection. Support 8-directional presets (Up, Down, Left, Right, 4 Diagonals) and manual X/Y offset sliders.
> 2. **Stacking & Queuing**: Add a toggle for "Stacking Mode". 
>    - `Overlap`: All numbers spawn at the same anchor.
>    - `Queue`: New numbers offset upwards based on the height of existing active texts to avoid overlapping.
> 3. **Independent Animation Curves**: Use `window.TWEEN` to create multi-stage animations:
>    - **Scale**: Implement "Pop-up" (Start 0.1 -> Max 1.5 -> End 1.0) with `Back.Out` easing.
>    - **Opacity**: Support a "Stay & Flash" fade-out where opacity remains 1.0 for 80% of the duration and drops to 0 rapidly at the end.
> 4. **Parametric Control**: All values (duration, scale intensity, direction, easing type) must be linked to the `hitFeedbackParams` object for real-time debugging.
>
> **Refactor the `updateFloatingText` and `showFloatingText` functions to handle this new logic.**

---
