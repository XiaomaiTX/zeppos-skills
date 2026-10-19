# 可选动画与响应式状态库

本文件记录作者自己的两个 ZeppOS 项目，作为可选实现案例，不是 Zepp Health 官方 API 或兼容性承诺。以 ZeppOS 官方文档、目标设备 API_LEVEL 和真实运行验证为准；每次采用前检查仓库当前版本、源码和依赖。

| 项目 | 适用场景 | 当前源码关注点 |
| --- | --- | --- |
| [zeppos-fx](https://github.com/XiaomaiTX/zeppos-fx) | Device App 中对现有控件做数值补间、缓动或颜色插值 | `Fx` 提供 `start`、`pause`、`stop`、`restart`、`onStop`，通过回调更新控件属性；源码定时器使用 `@zos/sensor` 的 `Time` 与 `setTimeout` / `clearTimeout`。 |
| [zeppos-reactive](https://github.com/XiaomaiTX/zeppos-reactive) | 状态依赖较多，需要 `reactive`、`effect`、`computed`、`memo`、`watch`、`merge` 或 `mergeProps` | 当前核心源码使用 `Proxy`、`Reflect`、`WeakMap`、`Map`、`Set` 和 `Object.is`，不直接导入 `@zos` 模块；它不提供完整 UI 框架，需在 effect 中调用官方控件 API 更新界面。 |

## 采用流程

1. 先用官方 API 实现最小可行的交互。只有动效确有信息价值或状态联动开始变复杂时，才引入相应库；避免为简单页面增加依赖。
2. 固定依赖版本并检查包的构建产物、打包体积、转译目标、运行时内置 API 与目标 API_LEVEL。npm 安装成功或 Node.js 测试通过，不代表手表运行时兼容。
3. 确认应用形态。两个项目示例主要面向 Mini Program Device App；不要把 `@zos` Device App API 直接带入 Watchface、Settings App 或 Side Service。按对应应用类型重新查 API。
4. 在目标模拟器和真机验证运行、页面返回、电量、内存及重复进入页面的行为。不要仅凭 README 中的性能描述或桌面环境测试通过就推断设备表现。

## zeppos-fx

- 适合在有限控件上做短时、低频的数值动画。优先复用已创建的控件并更新其属性；控制动画数量与刷新频率，避免常驻动画、无意义动效和不必要重绘。
- 源码当前使用 `Time.getTime()` 计算时间，并以递归 `setTimeout` 调度。官方 `@zos/sensor` `Time` 和 Device App 全局 `setTimeout` / `clearTimeout` 页面均标注 API_LEVEL 2.0 起支持；仍需根据目标 API 文档、应用形态和固件验证。旧版 v2 JavaScript 总览对 timers 的描述与当前具体 API 页面不同，按目标项目应使用的文档版本判断，不能仅凭旧版总览判定包不可用。
- 页面或应用离开时，在相应生命周期调用 `pause()` 或 `stop()` 停止调度；检查当前源码的状态语义。不要假设存在 `destroy()`。若目标形态或 API_LEVEL 不提供其调度依赖，改用该形态文档支持的方案，不要未经验证自行补全定时器 polyfill。
- 将 `fps` 视为上限而非视觉质量保证；从较低刷新频率开始，在模拟器与真机测量动画连贯性和功耗。不要为了追求 60 FPS 持续更新手表屏幕。

源码入口：[README](https://github.com/XiaomaiTX/zeppos-fx/blob/master/README.md)、[动画与定时器实现](https://github.com/XiaomaiTX/zeppos-fx/blob/master/src/fx.ts)。相关官方接口：[Time](https://docs.zepp.com/zh-cn/docs/reference/device-app-api/newAPI/sensor/Time/)、[setTimeout](https://docs.zepp.com/zh-cn/docs/reference/device-app-api/newAPI/global/setTimeout/)、[clearTimeout](https://docs.zepp.com/zh-cn/docs/reference/device-app-api/newAPI/global/clearTimeout/)。

## zeppos-reactive

- 适合多个 UI 字段依赖共享状态的页面；小页面优先使用普通对象和直接事件处理，避免为了单个计数器、传感器读数引入代理与依赖追踪。
- `effect` 中只处理必要的 UI 同步；不要把高频传感器采样或大型、频繁变化的对象图无差别地做深层响应式处理。测量代理、依赖集合、effect 回调和重复更新带来的 CPU 与内存开销。
- 当前仓库实现未暴露 `stop` / `dispose` 类 effect 清理 API：`effect()` 返回的是可再次执行的 effect 函数，不是停止函数；当前 `watch()` 也未返回清理函数。页面短生命周期代码不要直接创建无法按需注销的 effect/watch。若要使用，先查看采用版本是否已增加清理机制，或为该项目补齐并测试生命周期清理。
- `Proxy`、`Reflect`、`WeakMap` 等具体运行时能力没有在该库 README 中建立目标设备兼容保证。检查实际打包代码、目标设备运行时和 API_LEVEL，并在模拟器、真机上验证；不要用改变语言目标的转译来假定可补齐缺失的运行时原语。

源码入口：[README](https://github.com/XiaomaiTX/zeppos-reactive/blob/main/README.md)、[响应式核心实现](https://github.com/XiaomaiTX/zeppos-reactive/blob/main/src/index.ts)。
