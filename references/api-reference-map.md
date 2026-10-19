# 官方 API 文档地图

手表端的 API 不是浏览器 API。对任何新调用，打开对应官方页面确认 import、签名、参数、返回值、事件、API_LEVEL、权限和设备限制；不要仅凭模块名猜接口。完整设备 API 源码索引在[官方文档仓库](https://github.com/zepp-health/zeppos-docs/tree/main/docs/reference/device-app-api/newAPI)。

## Device App：模块化 @zos API

| 模块 | 典型用途 |
| --- | --- |
| global、app、page、router | App/Page 构造器、当前实例、生命周期和页面导航 |
| ui、interaction、display | Widget 创建、事件、绘制、按键/触控和屏幕显示 |
| sensor、user、device、settings | 健康/运动/时间传感器、用户数据、设备与系统设置 |
| app-service、alarm、notification | 无 UI 设备服务、定时/事件触发与系统通知 |
| ble、transfer-file | 手表作为 BLE 中央设备连接外设，以及设备文件传输 |
| fs、storage、share-storage | 资源文件、应用本地持久数据和共享存储 |
| media、crypto、timer、i18n、utils | 音频、多媒体、安全随机数、平台计时、国际化和工具函数 |
| app-access | 查询/访问其他应用能力；严格按专页和权限判断 |

仓库的 newAPI 目前将 API 按 alarm、app-access、app-service、app、ble、crypto、device、display、fs、global、i18n、interaction、media、notification、page、router、sensor、settings、share-storage、storage、timer、transfer-file、ui、user、utils 分组。

查找方式：到文档仓库相同 module 子目录定位页面，然后打开网站对应的 reference/device-app-api/newAPI/<module>/<API>/ 页面。例如：
- [TEXT Widget](https://docs.zepp.com/zh-cn/docs/reference/device-app-api/newAPI/ui/widget/TEXT/)
- [LIST_VIEW（API_LEVEL 5.0，使用前核对设备支持）](https://docs.zepp.com/zh-cn/docs/reference/device-app-api/newAPI/ui/widget/LIST_VIEW/)
- [VIRTUAL_CONTAINER](https://docs.zepp.com/zh-cn/docs/reference/device-app-api/newAPI/ui/widget/VIRTUAL_CONTAINER/)
- [App](https://docs.zepp.com/zh-cn/docs/reference/device-app-api/newAPI/global/App/)
- [API_LEVEL 查询设备](https://docs.zepp.com/zh-cn/docs/guides/tools/zepp-app/)

## 手机端组件 API

- **Settings App**：AppSettingsPage、render 函数组件、SettingsStorage。参考[设置应用框架](https://docs.zepp.com/zh-cn/docs/guides/framework/app-settings/)和[设置应用 API](https://docs.zepp.com/zh-cn/docs/reference/app-settings-api/)。
- **Side Service**：AppSideService、Messaging、Fetch、SettingsStorage、文件传输/下载与图像转换。参考[伴生服务 API](https://docs.zepp.com/zh-cn/docs/reference/side-service-api/)。
- 设备端和手机端有不同运行时及 API；在错误运行时调用模块不会变成兼容代码。

## Watchface API

表盘使用专用 API 家族：hmUI 控件、hmSensor 传感器、hmSetting 系统设置、hmFS 文件系统、hmBle 和 timer。完整页面按 API 方法/控件拆分，查[表盘 API 源码目录](https://github.com/zepp-health/zeppos-docs/tree/main/docs/watchface/api)和[Watchface 快速开始](https://docs.zepp.com/zh-cn/docs/watchface/watchface-quick-start/)。不要用 Mini Program 的 @zos/ui 页面代替表盘接口定义。

## 快速查询策略

1. 从任务中的能力词找到模块，例如 BLE、路由、健康传感器、文件或 Widget。
2. 打开具体 API 页面，检查顶部 API_LEVEL、参数级别、权限与示例；阅读相关设计或生命周期说明。
3. 对跨模块特性再读 architecture、framework、best practice 页面。
4. 查目标机型的 deviceSource、屏幕特征和功能支持；不以模拟器可见即为真机支持。
5. API 页签名和示例冲突时，以目标版本的当前官方参考为准并尽可能作最小真机验证。

