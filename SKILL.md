---
name: zeppos-development
description: "面向 Zepp OS（ZeppOS）的小程序、表盘、设备应用、设置应用、伴生服务、Workout Extension 和自定义键盘开发。用户要求创建或修改 Zepp OS 项目、使用 Zeus CLI 或 @zos API、处理 API_LEVEL 和权限、适配屏幕、使用 ZML（@zeppos/zml）、处理 Device App 与 App Side 通信/HTTP/二进制/文件传输、调试、测试或发布时使用。"
---

# ZeppOS 开发

根据官方 Zepp OS 文档协助规划、实现、检查和发布穿戴设备应用。先识别目标形态和设备，再读取对应参考文件；按需打开官方 API 页面核对签名、API_LEVEL、权限、参数和设备支持，不凭记忆补造接口。

## 工作流程

1. **识别产物。** 判断任务是 Mini Program（设备应用及可选伴生模块）、Watchface、Workout Extension、Keyboard Widget，还是 Shortcut Card / SecondaryWidget。查看现有项目的 app.json、目录、package.json 和 Zeus CLI 版本；保留项目既有结构。
2. **确定目标设备。** 从用户要求或项目配置取目标型号。查官方设备列表，记录 API_LEVEL、Zepp OS 版本、屏幕形态与分辨率、物理按键、特殊功能支持。型号未指定时，先做屏幕形态通用方案，并说明设备相关限制。
3. **判定兼容性。** 选当前文档版本与正确 API 家族；对每个要用的 API 查看其 API_LEVEL 和权限标记。把应用最低 API_LEVEL 设为真正使用的最低能力；低版本设备需要支持时，提供替代实现或能力检测。
4. **选架构与文档。** 按任务读取下方参考文件。小程序跨设备/手机/服务器数据时，先决定 Device App、Settings App、Side Service、App Service 各自职责，不把任意逻辑塞入单一页面。Device App ↔ App Side 的简单消息可用原生 Messaging 或 ZML；复杂双向请求、HTTP 代理、二进制消息可优先评估 [ZML](references/zml.md)，大型文件用 TransferFile 路线。需要补间动画或复杂响应式状态时，再看 [project-libraries.md](references/project-libraries.md)；这些库是可选案例。
5. **实现并适配。** 从 Zeus 模板和官方 API 开始；遵守生命周期、权限、资源、屏幕、输入、隐私和电量要求。避免引入未经设备运行验证的 Web / Node API 或第三方包。
6. **验证。** 运行项目已有脚本和 Zeus CLI；在模拟器测试布局、输入、传感器和错误路径，再用目标真机核实固件差异。检查构建包、权限、目标设备和发布资料。
7. **汇报。** 列出实现内容、构建/测试结果、实际验证的设备与 API_LEVEL，以及尚未覆盖的设备或真实硬件行为。

## 硬性检查

- 区分 Zepp OS 版本、API_LEVEL、app.json 的 configVersion 和应用 apiVersion.minVersion。它们表达不同概念。
- 以具体 API 页面和设备列表为准；不要仅凭产品名称、Zepp OS 主版本、新闻标题或另一个 API 的能力推断兼容性。
- 使用模块化 API 时按文档从 @zos/... 导入；不要混用 Zepp OS 1.0 全局 API 与新 API，除非用户明确要求维护旧项目。
- 遵守 Zepp OS JavaScript Runtime 限制。当前文档明确禁止 eval 和 new Function（new Function('return this') 例外）；具体全局 API（含定时器）按应用形态、文档版本和最低 API_LEVEL 核实，不把旧版文档的限制直接套用到新 API；不要默认浏览器或 Node.js 标准库可用。
- 将需要用户授权的权限代码加入 app.json；不得为方便而申请未使用的健康、用户或设备权限。
- 把手表看作低功耗、小屏、单任务交互设备；控制屏幕重绘、后台运行、传感器监听和网络/蓝牙连接范围。

## 参考文件

- 兼容性与配置：[platform-compatibility.md](references/platform-compatibility.md)
- 小程序架构、生命周期、权限、存储、屏幕适配：[mini-program.md](references/mini-program.md)
- ZML 基类、双向通信、HTTP 代理、二进制消息、TransferFile 与选型：[zml.md](references/zml.md)
- 表盘专用架构、接口和设计：[watchface.md](references/watchface.md)
- Workout Extension、键盘、副屏和快捷卡片：[special-app-types.md](references/special-app-types.md)
- Zeus CLI、模拟器、真机预览与发布：[tooling-release.md](references/tooling-release.md)
- 穿戴设备设计、国际化和无障碍：[design-accessibility.md](references/design-accessibility.md)
- API 模块查找地图：[api-reference-map.md](references/api-reference-map.md)
- 官方文档全局目录与版本处理：[document-map.md](references/document-map.md)
- 可选动画与响应式状态库案例：[project-libraries.md](references/project-libraries.md)

