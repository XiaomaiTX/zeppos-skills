# 工具、调试、验证与发布

## Zeus CLI

从官方 Quick Start 安装 Node.js LTS 和 Zeus CLI；使用 zeus -v 确认环境。版本及操作系统要求会变化，执行前查看当前[安装说明](https://docs.zepp.com/zh-cn/docs/guides/quick-start/environment/)和[CLI 文档](https://docs.zepp.com/zh-cn/docs/guides/tools/cli/)。

| 命令 | 用途 |
| --- | --- |
| zeus create <name> | 从官方模板创建 Mini Program 或 Watchface |
| zeus dev | 连接模拟器进行监听、构建和快速预览 |
| zeus login | 登录开发者账户以使用账户相关功能 |
| zeus preview | 生成真机安装/预览二维码 |
| zeus build | 构建安装包；文档示例输出到项目 dist/ |
| zeus config | 管理 CLI 配置 |
| zeus bridge | 进入 Developer Bridge 模式 |
| zeus status | 查看登录、设备模拟器等状态 |

从现有仓库开始时保留其 CLI 依赖和 lockfile；不要未经核对全局升级工具或重建项目模板。

## 测试路径

1. **本地静态检查：** 检查 JSON、资源路径、import 包名、权限声明、设备目标和编译错误。
2. **模拟器：** 运行对应设备模拟器；测试圆/方/条带屏幕、系统状态、输入方式、传感器数据和页面返回。用日志定位问题。模拟器能快速检查 UI，但不能代替真实传感器、固件或耗电验证。
3. **真机：** 登录 CLI，在 Zepp App 开启 Developer Mode，使用 zeus preview 安装。核实真实固件、表盘常亮/息屏、BLE 外设和传感器行为。
4. **构建：** 使用 zeus build，检查 dist/ 内安装包，测试最终包而不只测开发预览包。
5. **发布前：** 再核对设备支持范围、API_LEVEL、权限、隐私声明、语言、应用分类、截图和图标。

官方调试指南说明当前开发工具并不提供传统断点调试；优先加有目的的日志，区分 Device App、Settings App 和 Side Service 日志，复现后删掉噪声日志。参考[调试 Mini Program](https://docs.zepp.com/zh-cn/docs/guides/best-practice/debug/)及[模拟器指南](https://docs.zepp.com/zh-cn/docs/guides/tools/simulator/)。

## 发布

- [发布应用](https://docs.zepp.com/zh-cn/docs/distribute/)：开发者登记、App ID、国家/地区、分类、版本、语言、支持设备、截图、icon、隐私声明、权限与 SDK 信息。
- [发布表盘](https://docs.zepp.com/zh-cn/docs/distribute/watchface/)：另有表盘类目、付费情况、支持设备与安装包要求。
- [发布运动扩展](https://docs.zepp.com/zh-cn/docs/guides/workout-extension/distribute/)：按扩展专属流程提交。

上架政策、图片尺寸、付费能力和审批要求可能改变；提交时以开发者平台与当前官方发布页为准。除非用户明确要求且授权，不代替用户提交、发布或更改线上项目。

## 依赖

先检查 Zepp OS 官方推荐的 npm 包，再核实包支持的 API_LEVEL、运行时和授权。第三方包可能依赖不支持的 Promise、Node API 或动态代码；不要因其可在 Node.js 测试中运行就假定手表运行时兼容。

官方推荐列表包含 ZML（@zeppos/zml）。引入前先阅读 [ZML 参考](zml.md)，锁定版本并核对实际包的 exports、API_LEVEL、Side Service 声明及 lockfile；ZML 内部实现细节不等于跨版本保证。

