# runlog

极简的个人跑步记录工具，iPhone + Apple Watch。用来替代 Keep（广告多、社区功能重、无 REST API），只做一件事：记录跑步路线和数据，数据归自己。

**差异点：打开即运动。** 没有「开始跑步」按钮、没有 3-2-1 倒计时。App 自行判断跑 / 走 / 静止，静止不计入跑步时间。只有「结束」需要手动。

## 状态

设计完成，尚无 App 代码。

- 术语表：[`CONTEXT.md`](CONTEXT.md)
- 架构决策：[`docs/adr/`](docs/adr/)（0001–0006）
- Phase 1 需求：[issue #1](https://github.com/starlabrys/runlog/issues/1)（真源；仓库里 [`docs/spec-phase1.md`](docs/spec-phase1.md) 只是指针）

工程流程按 [mattpocock-skills](https://github.com/mattpocock/skills)。

## 分阶段

- **Phase 1**：iPhone 单机版（免费证书）。GPS + CoreMotion 分类、走 / 跑 / 静止分段、路径、距离、配速、每公里分段、本地存储、GPX / JSON 导出。无手表、无心率、无 HealthKit、无后端。
- **Phase 2**（开通 Apple Developer 账号后）：Apple Watch App（手表锚定 `HKWorkoutSession`）、心率、写入 Apple 健康、锁屏 Live Activity、Go 后端同步。

## 技术栈

Swift + SwiftUI，HealthKit / CoreLocation / CoreMotion / WatchConnectivity / MapKit / ActivityKit。后端 Go。

## License

[AGPL-3.0](LICENSE)
