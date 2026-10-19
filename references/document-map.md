# Zepp OS 官方文档目录

## 官方来源

- 当前中文开发文档：[docs.zepp.com/zh-cn/docs](https://docs.zepp.com/zh-cn/docs/)
- 官方源仓库：[zepp-health/zeppos-docs](https://github.com/zepp-health/zeppos-docs)
- 当前文档版本：[版本说明](https://docs.zepp.com/zh-cn/docs/intro/version/)
- 设备数据：[设备基本信息](https://docs.zepp.com/zh-cn/docs/reference/related-resources/device-list/)

2026-10-19 对官方仓库 main 的 Markdown / MDX 内容树做了完整目录盘点：共 534 个文档源文件，包含现行文档和技术参考。仓库内容可能先于在线站点发布，中文翻译或在线页面也可能滞后；文件存在不代表目标固件已支持。

## 主题入口

| 任务 | 首读文档 |
| --- | --- |
| 新建开发环境 | [Quick Start](https://docs.zepp.com/zh-cn/docs/guides/quick-start/)：环境、创建、模拟器、代码与真机预览 |
| 理解小程序组成 | [整体架构](https://docs.zepp.com/zh-cn/docs/guides/architecture/arc/)；[目录结构](https://docs.zepp.com/zh-cn/docs/guides/architecture/folder-structure/) |
| 设备端开发 | [Device App 简介](https://docs.zepp.com/zh-cn/docs/guides/framework/device/intro/)：App/Page、生命周期、路由、兼容、权限、文件系统、适配、服务、BLE |
| 手机端设置和连接 | [Settings App 简介](https://docs.zepp.com/zh-cn/docs/guides/framework/app-settings/intro/)；[Side Service 简介](https://docs.zepp.com/zh-cn/docs/guides/framework/side-service/intro/) |
| 开发工具 | [Zeus CLI](https://docs.zepp.com/zh-cn/docs/guides/tools/cli/)；[模拟器](https://docs.zepp.com/zh-cn/docs/guides/tools/simulator/)；[表盘工具](https://docs.zepp.com/zh-cn/docs/guides/tools/watchface/) |
| 项目规范 | [调试实践](https://docs.zepp.com/zh-cn/docs/guides/best-practice/debug/)；[多屏适配](https://docs.zepp.com/zh-cn/docs/guides/best-practice/multi-screen-adaption/)；[最佳实践源目录](https://github.com/zepp-health/zeppos-docs/tree/main/docs/guides/best-practice) |
| 版本差异 | [API_LEVEL 4.2 新特性](https://docs.zepp.com/zh-cn/docs/guides/version-info/new-features-42/)；[1.0 迁移指南](https://docs.zepp.com/zh-cn/docs/guides/version-info/migration-guide/)；[版本源目录](https://github.com/zepp-health/zeppos-docs/tree/main/docs/guides/version-info) |
| 特殊应用形态 | [Workout Extension](https://docs.zepp.com/zh-cn/docs/guides/workout-extension/intro/)；[Keyboard](https://docs.zepp.com/zh-cn/docs/guides/keyboard/intro/) |
| Watchface | [表盘快速开始](https://docs.zepp.com/zh-cn/docs/watchface/watchface-quick-start/)；[表盘工具](https://docs.zepp.com/zh-cn/docs/guides/tools/watchface/) |
| API | [Mini Program app.json](https://docs.zepp.com/zh-cn/docs/reference/app-json/)；[Device App API 源目录](https://github.com/zepp-health/zeppos-docs/tree/main/docs/reference/device-app-api/newAPI)；[SettingsStorage API](https://docs.zepp.com/zh-cn/docs/reference/side-service-api/settings-storage/) |
| 设计 | [设计规范](https://docs.zepp.com/zh-cn/docs/designs/)：交互、视觉、模板、组件、国际化和无障碍 |
| 上架 | [发布应用](https://docs.zepp.com/zh-cn/docs/distribute/)；[发布表盘](https://docs.zepp.com/zh-cn/docs/distribute/watchface/) |
| 问题排查 | [FAQ](https://docs.zepp.com/zh-cn/docs/guides/faq/)；[Samples](https://docs.zepp.com/zh-cn/docs/samples/) |

## 文档规模（仓库快照）

该次清点的源文件分布为：Guides 98、Reference 276、Watchface 93、Designs 54、Distribute 2、Samples 8、Intro 页面 3。API 参考包含 25 个 @zos 模块，表盘 API 另外按 hmUI、hmSensor、hmSetting、hmFS、hmBle、timer 分组。

这只是仓库快照，不是在线发布完成度或设备支持表。查询具体问题时直接读相关在线页面和仓库对应源文件，不需要把所有 API 方法复制进项目提示上下文。

