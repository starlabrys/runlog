# runlog — 设计草稿 (v0.3)

> 草稿。仓库建好后会按 mattpocock-skills 规范拆成：
> 第 3 节 → `CONTEXT.md`（术语表）；第 4 节 → `docs/adr/`（决策记录）；第 5–9 节 → `/to-spec` 生成的需求文档。
>
> **仓库**：`starlabrys/runlog`（公司 org，公开），由 OpenTofu 创建（`starlabrys-infra` 的 `github/opentofu/runlog/`）。**License**：AGPL-3.0（与 enx 一致）。决策见 `starlabrys/ops` 的 `docs/github/ADR-0002-runlog-repo.md`。

---

## 1. 背景与目标

Keep 广告多、社区功能重、无 REST API、数据不开放。**runlog** 是自用的极简跑步记录工具：只记录路线和跑步数据，数据归属自己。

**差异点：打开即运动。** 无「开始跑步」按钮、无 3-2-1 倒计时。App 自行判断跑 / 走 / 静止，静止不计入跑步时间。只有「结束」需要手动操作。

---

## 2. 分阶段计划

### Phase 1 — iPhone 单机版（免费证书，验证核心逻辑）

目标：验证「自动识别 + 分段」这套核心逻辑在真实跑步中是否可用。

- 免费个人证书（profile 7 天过期，需每周重签），仅 iPhone，iOS 最新正式版
- 数据源：CoreLocation（GPS、后台定位）、CoreMotion（`CMMotionActivity` 走/跑/静止、`CMPedometer` 步频、`CMAltimeter` 爬升）
- **无** HealthKit、**无** 手表、**无** 心率、**无** 后端同步
- 本地存储（SwiftData），可导出 GPX / JSON
- 自研分类器 + 自动暂停（速度 + 动作类型 + 阈值状态机）
- 手机端「强制开始 / 结束分段」按钮
- 忘记结束的兜底：距离或时长超过可配置上限即自动结束（默认距离 42.195 km，默认时长 = 用户按个人配速 / 马拉松关门时间设定）
- **无** 锁屏实时数据（Live Activity 推迟到 Phase 2）
- UI：实时跑步页（地图 + 数据）、历史列表、详情页（地图 + 配速图）、设置页
- 结束后仅支持删除整条记录

### Phase 2 — 完整版（Phase 1 基本可用后 → 开通 $99 Apple Developer 会员）

- 手表 App，**手表锚定** `HKWorkoutSession`（负责后台存活 + 心率）
- workout mirroring：手机负责 GPS + 主 UI，手表负责心率 + 停止
- 手机拉起手表：`HKHealthStore.startWatchApp(with:)`
- 心率实时流（手表 → 手机）+ 结束后用 HealthKit 补齐缺口
- 写入 Apple「健康」（`HKWorkout`），进 Apple Fitness 圆环
- 户外跑步自动暂停改用系统 `motionPaused` / `motionResumed` 事件（插入 `HKLiveWorkoutBuilder`），自研分类器保留作兜底
- Go 后端 + REST API 同步，跨设备与长期归档
- 崩溃 / 被杀恢复（周期性 checkpoint）
- 手机没电时手表尽力独立继续记录
- Live Activity：锁屏 + 灵动岛实时数据（距离 / 时长 / 配速 / 心率），仿 Keep

---

## 3. 术语表（→ CONTEXT.md）

- **Session（会话）**：从打开 App 到手动停止的整段时间。一次会话产出一条 Workout Record。
- **Segment（分段）**：会话内部按运动状态切分的连续片段，类型为 Running / Walking / Idle。
- **Running time（跑步时长）**：所有 Running 分段时长之和，是展示的主时长。
- **Idle（静止）**：被判定为静止的分段，不计入跑步时长也不计入走路时长。红灯、系鞋带、休息都属于。
  - _Avoid_: 「暂停」——「暂停」在本项目里只指用户主动操作，系统自动的叫 Idle / auto-pause。
- **Auto-pause / auto-resume**：会话自动从 Running/Walking 切入 Idle、再自动切回。
- **Force start / Force end（强制开始 / 结束）**：用户手动覆盖分类器，立即开始或结束当前 Running 分段。
- **Split（每公里分段）**：按累计距离每 1 km 切一段，用于配速分析。与 Segment 正交 —— **Split 按距离切，Segment 按状态切**。
- **Route（路径）**：会话期间的 GPS 轨迹点序列（时间、坐标、精度、海拔、速度）。
- **Workout Record（运动记录）**：一次会话结束后整理保存的完整数据条目。
- **Warm-up（热身）**：跑步前的慢跑 / 快走，归为 Walking 分段，正常记录路径，不算跑步时长。
  - _Avoid_: 「锻炼 / 训练」混用 Session。

---

## 4. 关键决策（→ ADR）

1. **手表锚定运动会话（Phase 2）** —— 手表跑 `HKWorkoutSession` 负责后台存活与心率，手机负责 GPS 与主 UI，两端 mirroring。备选「手机锚定」被否，因为 iOS 后台持续采集受系统限制更多。
2. **判断跑 / 走 / 停：设备端推理，主判据用 Apple `CMMotionActivity`（HAR）**
   - `CMMotionActivity` 是 Apple 设备端训练的人体活动识别模型，跑在动作协处理器上，**免费、近零耗电、无需打包模型**。作为跑 / 走 / 静止的主判据。
   - 叠加 GPS 速度 + 阈值 + 迟滞的状态机，做 Running / Walking / Idle 分段与自动暂停 / 恢复。
   - **所有推理在设备端完成。** 服务器只用于：原始信号归档、模型迭代后离线重算历史记录、（未来）训练。实时链路不依赖网络。
   - 理由：延迟（亚秒 vs 秒级 + 依赖网络）、离线可用（隧道 / 山道 / 飞行模式）、耗电（持续上传 50Hz IMU 是最大耗电项之一）、隐私、成本。
   - Phase 2 拿到 HealthKit 后，户外跑步的自动暂停可改用系统 `motionPaused` / `motionResumed` 事件，`CMMotionActivity` + 规则保留作兜底。
   - **未来可选增强（非承诺）**：用「信号录制」攒的带标注数据，用 Create ML Activity Classification 训一个设备端 `.mlmodel`，离线对比明显优于规则版再作为主判据。
3. **本地优先存储** —— SwiftData 本地为唯一真源，离线完全可用。HealthKit 写入与后端同步都是叠加的导出层，失败不影响本地记录。
4. **写入 Apple「健康」（Phase 2）** —— 以 `HKWorkout` 形式写入，运动进入 Apple Fitness / 健康 App 并计入圆环，同时天然多一份备份。
5. **Go 后端做归档同步（Phase 2 / 2.1）** —— REST API，本地推送到自建服务（`docker-hosted.wiloon.com` 已有基础设施），用于跨设备与长期归档，非实时依赖。
6. **打开即记录，停止需手动** —— 会话在 App 启动即开始（GPS + 动作监控），无独立「开始」操作；仅「结束」需人工。接受 10–30 秒识别延迟，热身段覆盖。
7. **仅支持最新 OS** —— iOS / watchOS 最新正式版，不做旧版本兼容。
8. **地图：Phase 1 用 Apple MapKit，路径坐标统一 WGS-84** —— 路径的唯一权威坐标是 WGS-84（GPS 原始），GPX 导出 / 后端 / iCloud 均存 WGS-84。坐标系转换是显示层的事。MapKit 在中国区会自动把 WGS-84 overlay 对齐到 GCJ-02 底图（把 `CLLocation.coordinate` 原样传给 `MKPolyline`，不要预转换），国内外一套代码。把地图封装成可替换的 `MapProvider` 抽象；若日后 Apple 地图步道细节不足，升级路径是换高德 SDK（存储不变，仅显示层转 GCJ-02）。
   - 被否备选：**Google Maps**（中国区坐标偏移需自己转、无 watchOS SDK、中国数据自 2016 年停更；即便手机可访问 Google 亦不选）；**百度**（BD-09 双层偏移）。

---

## 5. 用户故事（→ spec）

### Phase 1

1. 作为跑者，我打开 App 就进入记录状态，不需要点「开始」，这样我不会忘记开始记录。
2. 作为跑者，我热身慢走 / 慢跑时 App 记录路径但不计入跑步时长，这样我的平均配速真实。
3. 作为跑者，当我真正跑起来，App 自动开始一个 Running 分段。
4. 作为跑者，我等红灯站住超过阈值时间，App 自动进入 Idle，不计入跑步时长。
5. 作为跑者，我重新起步，App 自动恢复 Running 分段。
6. 作为跑者，我从跑切到走，App 自动记为 Walking 分段。
7. 作为跑者，分类器判断错时，我能在手机上点「强制开始 / 结束」手动纠正当前分段。
8. 作为跑者，跑步时我锁屏把手机放兜里，App 在后台持续记录 GPS 和分段。
9. 作为跑者，跑步时点亮屏幕能看到地图 + 距离 + 当前配速 + 跑步时长 + 每公里分段。
10. 作为跑者，跑完我在手机上点「结束」，App 把整段会话整理成一条运动记录。
11. 作为跑者，我忘记点「结束」时，距离超过上限（默认 42.195 km）或时长超过上限（默认按我的配速 / 马拉松关门时间）后 App 自动结束会话。
12. 作为跑者，我能在设置页调整上面两个上限值。
13. 作为跑者，我能在历史列表看到所有跑步记录（日期、距离、跑步时长、平均配速）。
14. 作为跑者，我能打开一条记录看路径地图、每公里配速、爬升、步频、走 / 跑 / 静止时间构成。
15. 作为跑者，我能删除一条错误的记录。
16. 作为跑者，我能把一条记录导出成 GPX / JSON 自己保管。
17. 作为跑者，App 崩溃或被系统杀掉后重启，能恢复正在进行的会话，不丢已跑数据。
18. 作为跑者，GPS 丢失（隧道 / 高楼）期间 App 不画点、不画轨迹、不累计距离；信号恢复后从缺口处接着记录。

### Phase 2（追加）

19. 作为跑者，我在手机上打开 App，手表上的 App 被自动拉起并进入运动。
20. 作为跑者，手表持续采集心率并实时显示，同时传给手机。
21. 作为跑者，我能在手表上翻页看实时数据（距离、时长、配速、心率）。
22. 作为跑者，我能在手表上点「结束」或「强制开始 / 结束分段」。
23. 作为跑者，跑步时我锁屏，锁屏和灵动岛上有一小块区域显示实时距离 / 时长 / 配速 / 心率（Live Activity）。
24. 作为跑者，跑完记录自动写入 Apple「健康」，计入 Fitness 圆环。
25. 作为跑者，记录详情里能看心率曲线、心率区间分布。
26. 作为跑者，我的记录自动同步到自建后端，换手机也能看到历史。
27. 作为跑者，手机没电时手表能尽力独立继续记录。

---

## 6. 数据模型（→ spec / 实现决策）

**WorkoutRecord**

- `id`, `startedAt`, `endedAt`, `timezone`
- `totalDuration`, `runningDuration`, `walkingDuration`, `idleDuration`
- `distance`（米）
- `avgPace`（秒 / 公里）
- `elevationGain`（米）
- `avgCadence`（步 / 分）
- `avgHeartRate`, `maxHeartRate`（Phase 2）
- `calories`（Phase 2，由心率估算）
- `sourceDevice`, `appVersion`
- **Segments**: `[{ type: running|walking|idle, startedAt, endedAt, distance, avgPace }]`
- **Splits**: `[{ index, distance (=1000m，末段除外), duration, avgPace, elevationDelta, avgHeartRate? }]`
- **RoutePoints**: `[{ timestamp, lat, lon, altitude, horizontalAccuracy, speed, source }]`
- **HeartRateSamples**: `[{ timestamp, bpm }]`（Phase 2）

存储：SwiftData。RoutePoints 单次可能上千点，单独表 + 批量写。
导出：GPX（路径 + 心率扩展）、JSON（完整结构）。

**可配置项（设置页）**

- `maxDistanceKm`：会话最长距离，超过自动结束。默认 42.195（全马）。
- `maxDuration`：会话最长时长，超过自动结束。默认按用户配速 / 马拉松关门时间设定（典型 6 小时）。
- 分类状态机阈值（速度 / 步频 / Idle 判定时长 T 等），验证阶段用。
- 距离单位：公制（km、min/km），Phase 1 固定，不做英制。

---

## 7. 分类与自动暂停（实现决策）

**分层设计**

1. **HAR 层（Apple 设备端 ML）**：`CMMotionActivityManager` 输出 stationary / walking / running / cycling / automotive + confidence。这是主判据。
2. **融合层（规则状态机）**：把 HAR 输出 + GPS 速度 + 步频融合成 Running / Walking / Idle 分段，处理边界抖动与自动暂停。
3. **人工覆盖层**：Force start / end 按钮直接置状态并锁定 M 秒不被下层覆盖。

**输入信号**

- `CMMotionActivityManager`：活动类型 + confidence（主判据）
- `CLLocation`：速度（m/s）、`horizontalAccuracy` —— 过滤 accuracy > 20–30m 的点
- `CMPedometer`：步频（辅助区分慢跑 / 快走）

**状态机**：Idle ↔ Walking ↔ Running

- → Running：`CMMotionActivity = running`（中高置信），或 activity 不确定但速度持续 > ~2.2 m/s（约 7:35/km）+ 步频 > ~150 spm 达 N 秒
- → Walking：`activity = walking`，或速度 ~0.5–2.2 m/s 持续
- → Idle：`activity = stationary` 且速度 < ~0.5 m/s 持续 T 秒（T ≈ 15–20s，可调）
- Idle → 上一状态：检测到移动即恢复
- 迟滞（hysteresis）避免边界抖动
- Phase 2：户外跑步优先采信系统 `motionPaused` / `motionResumed`

**信号录制调试模式**：真实跑步时把 raw `CMDeviceMotion`（加速度 / 陀螺仪）+ `CMMotionActivity` + GPS + 状态机输出 + 人工纠正一起存档，作为未来训练 Create ML 模型的带标注语料，并支持离线回放测试。

阈值参数放设置页可调，方便验证阶段调。

---

## 8. 测试决策（→ spec）

- **分类状态机是核心**，做成纯函数模块（输入信号序列 → 分段序列），用录制的真实跑步信号回放测试（record-replay）。这是最高价值的测试 seam。
- GPS 过滤 / 距离计算：纯函数，单元测试。
- Split 计算：纯函数（route → splits），单元测试。
- 只测外部行为（信号进、分段 / 记录出），不测实现细节。
- 需要一个「信号录制」调试模式：真实跑步时把 raw `CLLocation` / `CMMotionActivity` 存下来作为测试语料。

---

## 9. 范围外（v1）

- Android / 其它平台
- 社交、动态、好友、排行榜
- 训练计划、课程、教练
- 记录后逐点编辑（裁剪首尾也推迟到未来）
- 骑行、游泳等其它运动类型
- 语音播报
- 奖牌、成就
- 多语言（先中文）
- Apple 之外的健康平台
- **Phase 1**：锁屏 / 灵动岛实时数据（Live Activity）—— 推迟到 Phase 2（开通 Apple Developer 账号后）
- **Phase 1**：手表、心率、写 Apple 健康、后端同步 —— Phase 2

---

## 10. 仍待确认

- 后台「始终定位」权限的首启引导方案（文案 + 时机）
- `maxDuration` 的具体默认值（用户按个人配速定，待用户给数字）
- 自动结束时是否发本地通知提醒用户
- 详情页图表用什么画（Swift Charts）

## 11. 已确认（本轮）

- Live Activity → Phase 2
- 爬升（`CMAltimeter`）→ Phase 1 加
- GPS 丢失 → 不画点 / 不画轨迹 / 不累计距离，恢复后从缺口继续
- 忘记结束 → `maxDistanceKm`（默认 42.195）+ `maxDuration`（默认按配速 / 关门时间）双上限，可配置
- 单位 → 公制固定，Phase 1 不做英制
- 仓库 `starlabrys/runlog`，公开，OpenTofu 创建，License AGPL-3.0
