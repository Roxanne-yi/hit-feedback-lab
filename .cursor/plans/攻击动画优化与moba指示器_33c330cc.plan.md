---
name: 攻击动画优化与MOBA指示器
overview: 基于 plan.md 新增内容：先实现胶囊体攻击动画优化（Dash/回弹、Squash and Stretch、挥砍弧光），再实现第三阶段类王者荣耀技能指示器（S1 扇形、S2 直线、拖拽瞄准、取消区）。
todos: []
isProject: false
---

# 下一阶段：攻击动画优化 + 类王者荣耀技能指示器

## 当前状态

- Phase 1 / Phase 2 已在 [index.html](e:\siqi.yi\agenticDesigner\ingameEnvironment\index.html) 中：3C、胶囊、摇杆、相机、HUD、小地图、攻击/技能按钮、伤害飘字、敌人与血条均已存在。
- 攻击键目前仅调用 `tryHitEnemy`，无角色前冲、形变或挥砍弧光；技能键为点击即释放，无指示器与拖拽瞄准。

## 文档依据

- **[plan.md 第三阶段](e:\siqi.yi\agenticDesigner\ingameEnvironment\PRDs\plan.md)**（约 171–248 行）：右手双重状态（按下→指示器、拖动→瞄准、释放→施法/取消）、扇形/直线指示器规格、取消区、飘字。
- **[plan.md 攻击动画模拟优化方案](e:\siqi.yi\agenticDesigner\ingameEnvironment\PRDs\plan.md)**（约 257–310 行）：冲刺与回弹、Squash and Stretch、挥砍弧光（RingGeometry）、可选 Hit Stop；文末「给 Cursor 的优化指令」明确 TWEEN/GSAP、参数与视觉规格。
- **[开发计划 Phase 3/4](e:\siqi.yi\agenticDesigner\ingameEnvironment.cursor\plans\dark_dungeon_demo_0-1_开发计划_ed3db9a8.plan.md)**：Phase 3 角色状态与攻击、Phase 4 技能与指示器。

---

## 第一部分：胶囊体攻击动画优化（优先）

在现有攻击逻辑（`btnAttack` → `tryHitEnemy`）之上增加「表现层」动画，不改变命中判定与飘字逻辑。

### 1.1 依赖与时机

- 引入 **TWEEN.js**（CDN，与 Three 同层），在攻击触发时驱动位移与缩放；若无 TWEEN 则用 `requestAnimationFrame` + 简单缓动（如 Exponential.Out）实现。
- 攻击触发入口：当前 `document.getElementById('btnAttack').addEventListener('click', ...)` 内，在 `tryHitEnemy` 之前或之后启动一套「攻击表现」流程（见下），并避免在表现动画期间重复触发（可加 `isAttacking` 或冷却标志，与现有连招计时器协调）。

### 1.2 Dash 与回弹（Snappy Translation）

- 使用角色**视觉组** `character.group` 的局部位置做动画（不直接改 `character.body`，避免与物理冲突）：在**世界 XZ 平面**沿当前朝向（与 `cameraYaw` 一致）向前位移。
- **向前冲刺：** 0.1s 内向前移动约 **0.6** 单位（plan.md：0.5–0.8），使用 EaseOut 或线性。
- **回弹：** 随后 0.2–0.3s 内回到原位，使用 **Exponential.Out** 或 **Back.Out**，形成「快进慢退」。
- 实现方式：用 TWEEN 驱动 `character.group.position` 在**世界坐标**下的偏移，或先算好目标世界位再写回 `group.position`（注意每帧 `updateVisual()` 会把 `group.position` 同步为 `body.position`，因此需在循环中在 `updateVisual()` 之后叠加「攻击偏移」，或短暂改写 `group.position` 并在 TWEEN 结束后恢复，具体以不破坏物理为准）。

**建议：** 用局部变量 `attackOffset`（Vector3），在攻击时 TWEEN 该偏移（0 → 0.6 沿朝向 → 0），每帧 `updateVisual()` 之后执行 `character.group.position.add(attackOffset)`，TWEEN 只改 `attackOffset`，这样物理 body 不受影响。

### 1.3 Squash and Stretch

- 作用对象：`character.visualMesh`（胶囊）的 `scale`。
- **蓄力压缩：** 位移开始瞬间，`scale` 设为 `(1.2, 0.8, 1.2)`（plan.md：scale.y 压至 0.8，x/z 略增）。
- **爆发拉伸：** 位移最快/顶点时刻，`scale` 设为 `(0.8, 1.4, 0.8)`。
- **恢复：** 攻击结束用弹性缓动回 `(1, 1, 1)`（TWEEN 的 Elastic.Out 或自定义弹性 1–2 次震动）。
- 与 1.2 的 TWEEN 时间轴对齐：0–0.1s 为 dash，0.1s 为 peak（stretch），0.1–0.4s 为回弹 + scale 弹性恢复。

### 1.4 挥砍弧光（Weapon Swing FX）

- 在**攻击开始时**于角色**正前方**（沿 cameraYaw 方向、略高于脚底）创建一 **THREE.Mesh**：
  - 几何：**RingGeometry**（或 PlaneGeometry 裁成扇形），张角 **120°**（plan.md），内径可接近 0、外径约 1–1.5，形成弧状薄片。
  - 材质：`MeshBasicMaterial`，`transparent: true`，`opacity` 约 0.5–0.7，颜色可用 plan.md 推荐 `#d4af37`（古铜金）或 `#4a90e2`（灵能蓝）。
- 动画：弧光在约 **0.2s** 内绕其中心或角色前方轴旋转约 120°，同时 `opacity` 从初始值过渡到 0，然后从场景移除并 dispose。

### 1.5 可选：Hit Stop

- 若本帧或本段逻辑中 `tryHitEnemy` 判定命中，在攻击表现中插入 **0.05s** 的「停顿」：例如暂停 TWEEN 更新 0.05s，或让 dash 总时长延长 0.05s 再继续回弹，以增强打击感。

### 1.6 与现有逻辑的衔接

- 保持 `tryHitEnemy(ATTACK_RANGE, ATTACK_DAMAGE, 'Attack')`、`lastAttackTime`、`updateEnemyVisible()` 的调用时机不变；仅在同一点击中**增加**上述表现层 TWEEN 与弧光生成。
- 若引入 `isAttacking` 标志，在攻击表现动画结束（约 0.4s）后清除，避免连点导致动画叠加错乱。

---

## 第二部分：类王者荣耀技能指示器（S1/S2 + 取消区）

在现有 S1/S2/S3 按钮的「点击即释放」基础上，改为「按下显示指示器 → 拖动瞄准 → 松手释放或取消」。

### 2.1 右手双重状态

- **按下 (touchstart/mousedown)：** 在角色脚底（或略高，y=0.01）生成当前技能对应的**指示器 Mesh**（S1 扇形，S2 直线），初始方向为角色当前朝向（与 `cameraYaw` 一致）。
- **拖动 (touchmove/mousemove)：** 指示器旋转角度由**手指/鼠标相对于按钮中心的 2D 偏移**映射到 3D 的 XZ 平面 Yaw：例如将偏移向量转为角度，设到指示器 `rotation.y`（或单独 Group 的 rotation），**角色本体不转**（仅指示器转）。
- **释放 (touchend/mouseup)：** 若在**取消区**内则销毁指示器、不施法；否则执行 `castSkill(N)`（或现有 tryHitEnemy + startSkillCooldown），然后销毁指示器。

### 2.2 取消区 (Cancel Zone)

- 定义屏幕上方一矩形区域为取消区（如顶部 15% 高度或 plan.md 所述「红色 X 区域」）；可用一透明 div 或通过 `clientY < 某阈值` 判断。
- 拖入取消区时：将指示器材质色改为 plan.md 的 **#ff4d4d**（警戒红）；松手时不调用施法逻辑，只移除指示器。

### 2.3 指示器规格

- **S1（扇形）：** `THREE.Shape` + `ShapeGeometry` 或扇形平面，半径约 **5**，张角 **90°**；`MeshBasicMaterial`，`transparent: true`，`opacity: 0.4`，颜色正常为 `#4a90e2` 或 `#d4af37`，取消区为 `#ff4d4d`；贴地 y=0.01。
- **S2（直线）：** 薄矩形平面，长 **10**、宽 **2**；材质同上；贴地 y=0.01。
- S3 可暂保持点击即释放，或复用 S1/S2 之一逻辑，视实现成本决定。

### 2.4 与现有技能逻辑的衔接

- 保留 `tryHitEnemy(SKILL_RANGES[n-1], SKILL_DAMAGES[n-1], ...)` 与 `startSkillCooldown(n)`；释放时传入「当前指示器朝向」用于后续扩展（如投射物方向）；当前若仍为范围判定，可先沿用现有距离判定，仅增加指示器表现与取消区逻辑。

---

## 实现顺序建议

1. **第一部分**（攻击动画优化）：TWEEN + attackOffset + visualMesh scale 动画 + 120° 弧光 + 可选 Hit Stop；验证与现有攻击键、飘字、敌人血条无冲突。
2. **第二部分**（技能指示器）：S1 扇形、S2 直线、共用的「按下-拖动-释放」状态与取消区；再接现有 `tryHitEnemy` 与冷却。

## 涉及文件

- 仅 [index.html](e:\siqi.yi\agenticDesigner\ingameEnvironment\index.html)：引入 TWEEN（或手写缓动）、在 CharacterManager 或全局中增加攻击表现状态与 TWEEN 更新、弧光创建与销毁、技能指示器 Mesh 的创建/旋转/取消区判断与事件绑定（在现有 btnSkill1/2/3 上增加 touch 与 mouse 的 start/move/end）。

## 验收要点

- 攻击键：有前冲、缩放形变、挥砍弧光，手感明显增强；命中时可有 0.05s 停顿。
- S1：按下出扇形，拖动旋转，松手施法或取消区取消；取消区显示为红色。
- S2：同上，指示器为直线矩形。
- 不破坏现有移动、跳跃、下蹲、相机、小地图与血条逻辑。

