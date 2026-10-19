# Workout Extension、Keyboard 与快捷组件

## Workout Extension（运动扩展）

运动扩展以插件形式进入系统“运动”应用，用于运动过程中显示实时指标或提供配套能力；它不是普通小程序页面的替代品。文档将该能力标记为 Zepp OS 3.5 新特性，但支持设备、运动数据、生命周期和 API 限制都要按最新设备表和扩展文档核对。

- [介绍](https://docs.zepp.com/zh-cn/docs/guides/workout-extension/intro/)
- [完整快速开始](https://docs.zepp.com/zh-cn/docs/guides/workout-extension/quick-start/)
- [发布流程](https://docs.zepp.com/zh-cn/docs/guides/workout-extension/distribute/)
- 运动指标、子运动类型、扩展注册和 app.json 约束：以快速开始里的“相关 API / 生命周期 / API 限制”为准。
- 先用示例工程跑通创建、模拟器/真机预览，再改布局；实时数据缺失、暂停/恢复和结束运动都要验证。
- 不要默认普通 Device App 可调用 Workout Extension 的 API，也不要把扩展的 API 限制和生命周期套用到普通页面。

## 自定义键盘

自定义 Keyboard Widget 从 API_LEVEL 4.2 开始开放，并依附一个 Mini Program 主体；主体负责入口、启用说明和设置，键盘负责实际输入界面。用户在系统设置中启用并在输入时切换键盘，同一时间只能启用一个自定义键盘。

- [自定义键盘介绍](https://docs.zepp.com/zh-cn/docs/guides/keyboard/intro/)
- [快速开始与完整示例](https://docs.zepp.com/zh-cn/docs/guides/keyboard/quick-start/)
- 核对 app.json 注册、键盘生命周期、允许使用的 API、系统输入方式及设备固件限制；键盘生命周期包含 onResume 和 onPause。
- 文档记录过部分机型固件 API 缺失的情况。即使目标 API_LEVEL 满足，也要检查设备表、API 存在性与目标固件，并提供引导或降级。

## SecondaryWidget 与 Shortcut Card

副屏应用和快捷卡片都是 Mini Program 的附属呈现形态，不等价于独立设备应用。它们的 UI 面积、入口和生命周期不同；部分设备不支持副屏应用。

- 先读[注册副屏应用和快捷卡片](https://docs.zepp.com/zh-cn/docs/guides/framework/device/secondary-widget/)。
- 通过设备表确认支持情况；依规范配置 module 和路径，验证 onResume / onPause 行为。
- 按其使用场景限制信息量：Widget 侧重信息展示，Shortcut Card 侧重快速访问。

