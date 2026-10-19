# 平台版本与兼容性

## 先选对文档版本

Zepp OS 官方文档的 v3+ 对应 API_LEVEL 3.0 及以上；v2 和 1.0 文档保留历史 API 以及迁移说明。Zepp OS 1.0 项目使用旧式 API；API 2.0 起采用模块化能力和 API_LEVEL 兼容承诺。新项目默认使用当前文档，旧项目按其配置和代码风格维护，不要直接混写两套 API。

- [Zepp OS 版本说明](https://docs.zepp.com/zh-cn/docs/intro/version/)
- [API_LEVEL 兼容性](https://docs.zepp.com/zh-cn/docs/guides/framework/device/compatibility/)
- [API 版本选择](https://docs.zepp.com/zh-cn/docs/guides/version-info/version-choose/)
- [1.0 迁移指南](https://docs.zepp.com/zh-cn/docs/guides/version-info/migration-guide/)

## 用 API_LEVEL 做兼容判断

API_LEVEL 表示某个固件承诺提供的 JS API 能力，不等于 Zepp OS 固件版本。按主版本和次版本作数字比较，例如 4.10 高于 4.4；不要用字符串字典序比较。

1. 查[设备基本信息](https://docs.zepp.com/zh-cn/docs/reference/related-resources/device-list/)，核实目标型号的 API_LEVEL、固件、屏幕形态、分辨率、deviceSource、物理按键和特殊能力支持。
2. 查目标 API 的具体文档。API 页面会标注 API_LEVEL，有些参数或字段还有单独的最低版本。
3. 根据实际最低需求设置 app.json 的 apiVersion.minVersion。更高 API_LEVEL 的设备承诺兼容较低 API_LEVEL 应用；低于 minVersion 的设备不能安装。
4. 对可选 API 使用文档认可的能力检测或提供降级路径。对必须能力则明确提高最低版本，并让用户知道设备范围。
5. 重新检查应用商店设备分发范围；设备列表、固件和市场可分发范围可能变化。

## 注意文档版本差异

官方中文首页、设备表和仓库中的新特性页面可能不同步。2026-10-19 检视的文档快照中，首页仍提示 API_LEVEL 4.2；设备表已经出现 API_LEVEL 4.4 机型（Zepp OS 5.0）；仓库还有 API_LEVEL 5.0 新特性源文件。不能据此推断 5.0 功能已在目标设备发布。对每项新能力，以在线具体 API 页、设备表和目标真机固件交叉核对；若不一致，按目标设备明确列出的 API_LEVEL 和 API 页标注实现，并将不确定性写入验证结果。

## app.json 的不同版本字段

- configVersion 控制配置文件格式；当前文档的主要格式是 v3，v1 已弃用。使用模板中的完整结构，不要凭记忆重建配置。
- apiVersion.minVersion 表示最低运行 API_LEVEL；它不是 configVersion，也不是固件版本。
- app.appType 区分 Mini Program 与 Watchface 等应用类型；选择对应项目模板和 API。
- permissions 列出应用调用的用户数据、系统服务和事件权限。每个 API 的权限需求以 API 页面为准。
- targets 配置构建/运行设备。v3 屏幕抽象使用 round / square / band 等特性；旧格式可能使用 deviceSource。不要把不同配置格式拼在一起。
- module 登记可选组件，例如 Side Service、App Service、App Event、快捷卡片等；只有使用时才注册。
- i18n 和 defaultLanguage 控制语言资源与回退语言。保持目录键名和语言代码与官方映射一致。

官方配置参考：[Mini Program app.json](https://docs.zepp.com/zh-cn/docs/reference/app-json/)。

## JavaScript Runtime 边界

按设备端当前文档的支持情况编写代码：ES6 可用；动态执行代码被禁用，eval 与 new Function 不支持（new Function('return this') 例外）；文档标注 Promise、Generator 和 JavaScript timers 不支持。不要默认浏览器 DOM、Fetch、Node.js 文件系统或 Node 原生模块可用。网络请求优先放在 Side Service，并按 Side Service Fetch API 实现。若项目模板、CLI 或目标固件表现与文档不一致，构造最小复现并在模拟器和真机验证。

参考：[JavaScript 支持情况](https://docs.zepp.com/zh-cn/docs/guides/framework/device/js-support/)。

