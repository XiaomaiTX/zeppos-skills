# Mini Program 开发

## 组件边界

完整应用可由运行在手表上的 Device App、运行在 Zepp App 的 Settings App（可选）、运行在 Zepp App 且无 UI 的 Side Service（可选）组成。Device App 不能直接和 Settings App 通信：

| 场景 | 推荐连接方式 |
| --- | --- |
| 手表设备应用 ↔ 手机伴生服务 | 简单消息可用原生 Messaging；复杂双向请求、统一消息管理或 HTTP 代理优先评估官方推荐的 ZML |
| 设置应用 ↔ 伴生服务 | 共享 SettingsStorage；写入会触发对端更新/重绘 |
| 伴生服务 ↔ 外部服务器 | Side Service Fetch API |
| 设备应用 ↔ 自身多页面 | 路由参数、App globalData、本地存储等；按数据寿命选择 |

先读[整体架构](https://docs.zepp.com/zh-cn/docs/guides/architecture/arc/)和[目录结构](https://docs.zepp.com/zh-cn/docs/guides/architecture/folder-structure/)；Side Service 参考[简介](https://docs.zepp.com/zh-cn/docs/guides/framework/side-service/intro/)及[API](https://docs.zepp.com/zh-cn/docs/reference/side-service-api/)。不要把重网络请求或手机 UI 逻辑放到手表 Device App。

ZML（@zeppos/zml）是官方推荐 npm 封装库，适合双向 RPC、HTTP 代理、消息分片和 TransferFile 接入；简单消息仍可直接使用原生 Messaging。它封装 Zepp OS 能力，不新增网络、BLE 或文件 API。按需读取 [ZML 参考](zml.md)，核对具体版本、API_LEVEL 与设备兼容性。

## Device App 生命周期

- 在 app.js 中用 App() 注册实例；onCreate 初始化全局状态，onDestroy 清理。
- 每个页面用 Page() 注册；页面路径须列在 app.json 的目标配置中。
- 常用顺序是页面 onInit → build → onDestroy；build 负责创建页面控件，路由参数从 push / replace 传入。
- App Service 使用单独的 AppService() 注册和生命周期。设置应用用 AppSettingsPage()，返回手机端 RenderFunc；伴生服务用 AppSideService()。
- 对称地清理传感器、BLE、UI 和系统事件监听，避免退出页面后仍留有工作。

参考：[Device App](https://docs.zepp.com/zh-cn/docs/guides/framework/device/app/)、[页面注册与绘制](https://docs.zepp.com/zh-cn/docs/guides/framework/device/page/)、[生命周期](https://docs.zepp.com/zh-cn/docs/guides/framework/device/life-cycle/)、[路由](https://docs.zepp.com/zh-cn/docs/guides/framework/device/router/)。

## app.json、权限和数据

- 先依据 Zeus 模板配置 app、runtime、permissions、targets、module、i18n、defaultLanguage；核对 schema 和已使用组件，不要只复制半份示例。
- API 文档中声明的权限代码应加入 permissions；移除未用权限。手表应用申请权限会影响用户安装同意和商店隐私说明。
- /assets 是打包资源，只读；/data 是应用隔离的可读写数据目录。Zepp OS 3.0 起文档支持 assets:// 和 data:// 路径。
- 资源和数据路径应使用官方文件 API。不要将服务器数据误写入只读资源，也不要假设不同应用共享 /data。
- 根据数据是否跨页面、跨重启、跨组件选择 sessionStorage、localStorage、share-storage 或 SettingsStorage；查看它们各自的生命周期、容量和可见范围后再存。
- 健康与个人资料只读到实现目标所需字段；向用户说明采集、保存、上传用途。

参考：[API 权限控制](https://docs.zepp.com/zh-cn/docs/guides/framework/device/permission/)、[文件系统](https://docs.zepp.com/zh-cn/docs/guides/framework/device/fs/)、[持久化存储实践](https://docs.zepp.com/zh-cn/docs/guides/best-practice/persistence-storage/)。

## UI 与屏幕适配

- 默认 UI 控件使用 @zos/ui 的 widget API。常规控件坐标为屏幕绝对坐标；先核对控件对坐标、对齐、字体、点击区域和动画的定义。
- v3 app.json 的屏幕类型抽象为圆屏 r、方屏 s、条带屏 b，可按需要加入具体分辨率。官方建议的设计基准为圆屏 480×480、方屏 390×450、条带屏 194×368；这些是设计基准，不是每款设备的实际分辨率。
- 用 px() 和配置限定符组织布局/资源；同屏幕形态也要核对目标机型和设备表。按屏幕边缘、安全区、系统状态栏、圆屏裁切处理内容。
- API_LEVEL 4.0 引入的 VIRTUAL_CONTAINER / Flex layout 可简化可变布局；使用前检查目标 API_LEVEL 和控件属性支持情况。
- SecondaryWidget / Shortcut Card 是否可用按设备表核验；它们扩展 onResume / onPause 等生命周期，按对应页面限制实现。

参考：[屏幕适配规范](https://docs.zepp.com/zh-cn/docs/guides/framework/device/screen-adaption/)、[Layout 属性与 Flex 布局](https://docs.zepp.com/zh-cn/docs/guides/framework/device/layout/)、[副屏应用和快捷卡片](https://docs.zepp.com/zh-cn/docs/guides/framework/device/secondary-widget/)。

## App Service 与 BLE

App Service 在手表端无 UI 运行，可由 alarm、notification、系统事件或 Device App 启动。持续运行会受系统资源和 API 限制；优先选择单次执行，只有需求必要时才保持运行，声明所需权限并验证退出/重启行为。

BLE 主机模式可以让手表连接外设；它不同于手表与手机的 BLE 通道。按外设 GATT 协议处理扫描、连接、Service、Characteristic 和断线重连。不要猜测设备协议或 UUID。

参考：[App Service](https://docs.zepp.com/zh-cn/docs/guides/framework/device/app-service/)、[系统事件](https://docs.zepp.com/zh-cn/docs/guides/framework/device/system-event/)、[BLE 主机模式](https://docs.zepp.com/zh-cn/docs/guides/framework/device/ble-central/)。

