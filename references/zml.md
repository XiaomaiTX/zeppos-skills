# ZML（@zeppos/zml）

本页是 ZML 的按需参考。先确认项目是 v3 Mini Program，且确实需要 Device App 与 App Side 协作；简单单向数据先比较原生 Messaging 与 ZML 的维护成本。ZML 是 Zepp OS 官方推荐的 npm 封装库，封装设备与 Side Service 的消息通道、原生 TransferFile 以及手机侧 Fetch/Downloader 等能力；它不会新增 BLE、网络或文件系统等系统权限，也不会把手机侧 API 变成手表端 API。

版本状态核验日期：2026-10-19。

## 版本与兼容性

| 检查项 | 本次源码审阅结论 | 使用规则 |
| --- | --- | --- |
| npm | 当前检索到的最新包为 **@zeppos/zml@0.0.43** | 安装时查 npm 与 lockfile；不要把 npm 最新版等同于已审阅源码 |
| 可审阅源码 | main 的 package.json 为 **0.0.41**；最新可读取 tag 为 **v0.0.42** | 本页实现细节和代码以 v0.0.42 源码、类型和 examples 为准；示例固定安装 0.0.42，升级到 0.0.43+ 时先比较发行包和类型再构建、真机验证 |
| API_LEVEL | npm 0.0.43 README 说 API_LEVEL ≥3.0（并特别写 0.0.28 需要 3.6）；v0.0.42 仓库 README 也如此，但仓库 known-issues 又写“最新版本”要求 3.6；example app.json 仍用 3.0 | 官方材料互相矛盾；不得把 3.0 或 3.6 当作无条件结论。此示例以 3.6 满足较严格说明；若需覆盖 3.0 设备，先核对安装包实际说明、API 文档与目标真机 |
| 导入入口 | @zeppos/zml/base-app、@zeppos/zml/base-page、@zeppos/zml/base-side | 使用 package.json exports 暴露的子路径；不要从未导出的源码路径导入 |
| 类型定义 | 根目录 zml.d.ts 没有覆盖部分运行时能力/可选参数 | TypeScript 项目以实际安装包和编译器检查为准；不能因为类型没写就假设运行时不支持，也不能因源码存在就称其为稳定公开 API |

参考：[官方推荐 npm 包](https://docs.zepp.com/zh-cn/docs/guides/tools/npm/officially-recommended/)、[npm 当前 README](https://www.npmjs.com/package/@zeppos/zml)、[main package.json](https://github.com/zepp-health/zml/blob/main/package.json)、[ZML README](https://github.com/zepp-health/zml/blob/v0.0.42/README.md)、[v0.0.42 package.json](https://github.com/zepp-health/zml/blob/v0.0.42/package.json)、[类型定义](https://github.com/zepp-health/zml/blob/v0.0.42/zml.d.ts)、[已知问题](https://github.com/zepp-health/zml/blob/v0.0.42/docs/zh/known-issues.md)。

## 目录

- [版本与兼容性](#版本与兼容性)
- [安装与配置](#安装与配置)
- [三个基类与生命周期](#三个基类与生命周期)
- [消息 API](#消息-api)
- [HTTP 代理](#http-代理)
- [二进制消息与分片](#二进制消息)
- [文件传输与可读边界](#文件传输)
- [断线与可靠性](#断线超时与可靠性)
- [其他 Side 模块](#其他-side-模块)
- [选型规则](#选型规则)

## 安装与配置

在现有 Zeus 项目根目录安装并锁定审阅过的版本：

~~~sh
npm install --save-exact @zeppos/zml@0.0.42
~~~

沿用项目自己的 app.json，只确认它启用了 Device App 页面和 App Side 模块。以下是按 ZML helloworld2 示例缩减的 v3 配置形状；项目 appId、图标文件、页面路径、屏幕目标、apiVersion、i18n 和所需权限必须按项目与官方设备资料填写，不能整份覆盖原配置：

~~~json
{
  "configVersion": "v3",
  "app": {
    "appId": 12345,
    "appName": "zml-demo",
    "appType": "app",
    "version": { "code": 1, "name": "1.0.0" },
    "icon": "icon.png",
    "vender": "your-vendor",
    "description": "ZML communication demo"
  },
  "permissions": [],
  "runtime": {
    "apiVersion": {
      "compatible": "3.0",
      "target": "3.0",
      "minVersion": "3.6"
    }
  },
  "targets": {
    "common": {
      "module": {
        "page": { "pages": ["page/index/index.page"] },
        "app-side": { "path": "app-side/index" }
      },
      "platforms": [{ "st": "r" }, { "st": "s" }, { "st": "b" }],
      "designWidth": 480
    }
  },
  "i18n": {
    "en-US": { "appName": "ZML Demo" }
  },
  "defaultLanguage": "en-US"
}
~~~

minVersion 3.6 是保守起点，不是 ZML 对每款设备的无条件要求；若支持 API_LEVEL 3.0 设备，先验证准确的 ZML 发行包、Zepp OS 文档要求和目标真机。手表端继续按项目要求从 @zos/* 导入系统 API；不要为了使用 ZML 将 Mini Program 改写为旧 hmApp 全局 API。

## 三个基类与生命周期

- BaseApp(options) 传给原生 App(...)；保存 globalData，由 ZML 在 App 生命周期挂接消息及其他 app 插件。用户 onCreate 在插件初始化后运行，onDestroy 清理用户逻辑和插件。
- BasePage(options) 传给 Page(...)；页面实例有 state，并可访问 App 的 globalData。插件在页面初始化时装配通信回调；onDestroy 时解绑。页面 build() 仍按项目 UI 结构绘制，ZML 不代替 @zos/ui。
- BaseSideService(options) 传给原生 AppSideService(...)；在 Side Service 生命周期挂接消息、文件接收、下载、图像转换及 Settings。Side Service 只有在 app.json 中声明后才会随 Mini Program 运行。
- 最小示例把 onRequest/onCall 只放在页面或 App 其中一层，避免 Device App 的 BaseApp 与 BasePage 对同一路由重复登记处理器。BasePage 的 onInit/onResume/build/onPause/onDestroy 仍应按页面生命周期工作。
- App Side 的网络、Settings、Downloader 等 API 只能在 Side Service runtime 使用。不要在 Device App 页面直接调用 this.fetch() 或 this.download()。

### 最小完整通信骨架

app.js：

~~~js
import { BaseApp } from '@zeppos/zml/base-app'

App(
  BaseApp({
    globalData: {},
    onCreate() {
      this.log('app created')
    },
    onDestroy() {
      this.log('app destroyed')
    },
    onRequest(req, res) {
      // Side Service 发来的请求在 App 层统一处理，页面切换时仍可应答。
      if (req.method === 'demo.device-info') {
        res(null, { ready: true })
      } else if (req.method === 'file.received') {
        res(null, { accepted: true, fileId: req.params.fileId })
      } else {
        res('unknown method: ' + req.method)
      }
    },
  }),
)
~~~

page/index/index.page.js：

~~~js
import { BasePage } from '@zeppos/zml/base-page'

Page(
  BasePage({
    state: { reply: '' },
    build() {
      // 按项目 UI 实现绘制；此处仅保留通信骨架
    },
    async askSide() {
      try {
        const result = await this.request({
          method: 'demo.echo',
          params: { text: 'hello from watch' },
        })
        this.state.reply = result.text
      } catch (error) {
        this.error('demo.echo failed', error)
      }
    },
    onCall(message) {
      if (message.method === 'demo.status') {
        this.state.reply = message.params.text
      }
    },
  }),
)
~~~

app-side/index.js：

~~~js
import { BaseSideService } from '@zeppos/zml/base-side'

AppSideService(
  BaseSideService({
    onInit() {
      this.log('side service initialized')
    },
    onDestroy() {
      this.log('side service destroyed')
    },
    onRequest(req, res) {
      if (req.method === 'demo.echo') {
        res(null, { text: req.params.text })
      } else {
        res('unknown method: ' + req.method)
      }
    },
    onCall(message) {
      if (message.method === 'demo.telemetry') {
        this.log('telemetry', message.params)
      }
    },
    sendStatusToWatch(text) {
      // Side Service 也可以主动呼叫 Device App；call 是单向通知。
      this.call({ method: 'demo.status', params: { text } })
    },
  }),
)
~~~

上面是 ZML v0.0.42 examples 的工厂式注册形态。页面的 build、UI import、项目路径与 app.json 页面注册仍须按实际项目补全。call() 是单向消息；其调用成功不代表远端业务逻辑执行成功。想要业务确认时使用 request() / onRequest()。

## 消息 API

| API | 用途 | 对端处理 |
| --- | --- | --- |
| request({ method, params }) | 双向异步请求，返回 Promise | 对端 onRequest(req, res) 必须且只能调用一次 res(error, data) |
| call({ method, params }) | 单向通知；不等待业务返回 | 对端 onCall(message)；需确认时另发 request |
| onRequest(req, res) | 注册本端请求处理器 | 成功 res(null, result)，失败传非空 error |
| onCall(message) | 注册本端单向消息处理器 | 不返回结果；避免长任务阻塞，必要时入队 |

Device App Page 可以调用 Side Service 的 request/call；Side Service 可以向 Device App 反向调用。Settings App 与 Device App 不因此直接通信；仍通过 Side Service、SettingsStorage 或其他官方机制协作。

- request 实现有默认约 60 秒请求超时和约 5 秒握手超时（源码内部默认值）；不要让页面在超时期间无限等待。处理 rejected Promise，并给 UI 反馈。
- 捕获错误并记录 message，可以在诊断时检查 code。当前源码中的错误码不是类型文件公开的稳定枚举；不要把数值硬编码成跨版本协议。
- BLE 断开可能导致请求失败、等待到超时或连接状态变化。源码包含 onBleChanged(status) 事件钩子，但状态语义、断线期间请求是否及时拒绝，以及系统重连时序都应在所用版本和真机验证。业务重试须有幂等 request ID 和去重逻辑。
- v0.0.42 的 zml.d.ts 把 request 声明为 request({method, params})，但运行时实现还有可选 options 参数；超时默认与配置参数可能存在类型缺口。不要在 TypeScript 项目无核对地照搬未声明的第二参数。
- onRequest/onCall 处理器可挂在 BasePage 或 BaseApp；选一个处理层负责每条 method 路由，并在页面销毁/重建后测试解绑及重复注册。

## HTTP 代理

Device App 页面调用 httpRequest() 时，ZML 把 http.request 作为消息发送给 Side Service，再由 Side Service 的 fetch() 调用原生 Fetch API。网络请求因此由手机侧执行，不是在手表上新增网络栈。

~~~js
async loadSummary() {
  try {
    const response = await this.httpRequest({
      method: 'get',
      url: 'https://api.example.com/v1/summary',
      timeout: 10000
    })
    if (response.status < 200 || response.status >= 300) {
      throw new Error('HTTP status ' + response.status)
    }
    // body 的具体形态按 Side Service Fetch 和服务端响应验证。
    return response.body
  } catch (error) {
    this.error('httpRequest failed', error)
    throw error
  }
}
~~~

仅 App Side 可直接调用 this.fetch({ url, method, headers, body, timeout })。ZML Side 插件会把 HTTP 超时默认值设为 10 秒，并支持其 wrapper 中的 baseURL 处理；Zepp OS 原生 Side Service Fetch 才是实际网络执行者。官方 Fetch API 支持字段、响应体形状、域名/证书策略以及超时行为要看目标版本专页和手机实机。ZML 类型把部分 body 声明成 Record<string,string>，而底层 Fetch 的 body 可能支持字符串；示例优先使用 GET，需 POST 时以当前官方 Fetch 类型验证后再写。

Side Service 也可在自己的逻辑中直接调用 fetch()，不经过手表消息代理：

~~~js
async fetchSummary() {
  const response = await this.fetch({
    method: 'get',
    url: 'https://api.example.com/v1/summary'
  })
  if (response.status < 200 || response.status >= 300) {
    throw new Error('HTTP status ' + response.status)
  }
  return response.body
}
~~~

参考：[Side Service Fetch API](https://docs.zepp.com/zh-cn/docs/reference/side-service-api/fetch/)、[ZML Side 消息实现](https://github.com/zepp-health/zml/blob/v0.0.42/src/core/side/messaging/messaging-plugin.js)、[HTTP 代理示例](https://github.com/zepp-health/zml/blob/v0.0.42/examples/helloworld3/page/common/home/ble-http-proxy/index.page.js)。

## 二进制消息

request()/call() 的运行时 message 层可承载 ArrayBuffer、TypedArray / Uint8Array 和 Buffer 兼容对象。传入二进制时使用原始字节载荷；不要期望 ZML 替应用编码 {method, params} 的 JSON 信封。应用要自行定义带版本号、操作码、长度、字节序和校验字段的二进制协议。zml.d.ts 只声明对象形态，TypeScript 项目需要为实际安装版本补准确的局部类型或在边界作受控类型转换。

~~~js
// Device App：只发送/接收原始 bytes；生产协议应自行定义 header/version。
async sendBinary() {
  const payload = Uint8Array.from([0x5a, 0x4d, 0x4c, 0x01]).buffer
  try {
    const reply = await this.request(payload)
    if (!(reply instanceof ArrayBuffer)) {
      throw new Error('unexpected binary reply')
    }
    return new Uint8Array(reply)
  } catch (error) {
    this.error('binary request failed', error)
    throw error
  }
}
~~~

接收端 onRequest(bytes, res) / onCall(bytes) 要按二进制分支解析；对象消息与二进制消息不可假设有相同 envelope。示意：

~~~js
onRequest(req, res) {
  if (req instanceof ArrayBuffer) {
    const bytes = new Uint8Array(req)
    // 检查协议版本、操作码、长度；以下只示意回显载荷。
    res(null, bytes.slice().buffer)
    return
  }
  // JSON 请求继续按 method/params 分发。
}
~~~

### 自动分片、重组与限制

- 审阅到的 v0.0.42 src/shared/message.js 将 ZML 协议帧设为 3600 字节，其中自定义头 16 字节、ZML 片段 payload 3584 字节；再扣除底层 HM Message 头 66 字节后，每片可用业务数据约 3518 字节。ZML 自动排序和重组连续片段，长度齐全后才向上交付完整消息。
- 这些是源码实现细节，不是公开承诺的最大消息长度。文档、类型和 API 页面没有定义可靠的总消息上限；不要据此推导任意大 Buffer 都能发送。
- 这不是流式传输：收端先在内存保留片段，再拼接完整载荷。发送端/接收端和重组期间会有额外复制，峰值内存可能明显高于 payload 长度；大数据或可落盘文件优先使用 TransferFile。
- 审阅源码未见分片级重传、校验和、断点续传或持久化。也未见不完整重组 session 的超时清理；request Promise 超时不能代表消息缓存释放。请限制二进制消息大小，并在目标设备、BLE 状态和当前实际 npm 版本上压力测试；若依赖大二进制消息，请先验证 .43+ 发布包的相同行为。

## 文件传输

文件发送和二进制消息是两条不同路线：

| | sendFile() / onReceivedFile() | request() / call() 的二进制消息 |
| --- | --- | --- |
| 底层 | ZML 封装 Zepp OS 原生 TransferFile（设备 @zos/ble/TransferFile；Side Service TransferFile Inbox） | ZML 消息管理和 BLE Messaging 协议 |
| 数据形态 | 文件路径和文件任务；由传输队列传输 | 一条消息中的内存字节载荷 |
| 反馈 | 任务 progress / change 和文件接收回调 | request 成功/失败或 call 无业务应答 |
| 适用 | 体积较大的日志、图片、导出文件 | 小型/中型二进制命令、传感器包、明确有界 payload |
| 关键限制 | 传输成功只表示原生传输状态完成，不代表接收业务验证或入库成功 | 不是文件系统、无公开总大小保证、完整消息常驻内存且无源码可见的分片重试 |

设备页面发文件并观察任务：

底层 TransferFile 从 API_LEVEL 3.0 起提供；任务状态包括 pending、transferring、transferred、error、canceled。progress 事件中 event.data.fileSize 和 loadedSize 是字节数，change 事件以 event.data.readyState 报告状态。ZML 返回并透传原生任务，不改变这些系统状态语义。

~~~js
sendReport() {
  const fileId = 'report-2026-01'
  const task = this.sendFile('data://download/report.bin', {
    fileId,
    kind: 'report'
  })

  task.on('progress', (event) => {
    const { loadedSize, fileSize } = event.data
    this.log('file progress', loadedSize, fileSize)
  })
  task.on('change', (event) => {
    const readyState = event.data.readyState
    this.log('transfer state', readyState)
    if (readyState === 'error') {
      this.error('TransferFile failed', event)
    } else if (readyState === 'canceled') {
      this.log('TransferFile canceled')
    }
  })
  return task
}
~~~

ZML 的 Side Service file inbox 监听完成事件，onReceivedFile(file) 在文件接收完成后取得原生 FileObject。接收进度应优先在发送方的 TransferFile task 上监听；不要因为 Side Service 收到了完成回调就把它当成可观察全程进度。FileObject 的 fields 和 path 由目标 runtime 的官方 TransferFile 文档定义；ZML v0.0.42 examples 在设备端读取 file.params 和 file.filePath。App Side 能否用该路径读取内容需单独验证。接收后由 Side Service 通过 request() 做业务确认：

~~~js
onReceivedFile(file) {
  const fileId = file.params && file.params.fileId
  this.log('file arrived', fileId)

  // 只确认“接收回调已到达”。不要声称已读取/校验/持久化文件内容。
  this.request({
    method: 'file.received',
    params: { fileId }
  }).then((result) => {
    this.log('device receipt result', result)
  }).catch((error) => {
    this.error('receipt acknowledgement failed', error)
  })
}
~~~

发送方在 Device App 的 BaseApp.onRequest(req, res) 处理 file.received 并返回结果；把回调放在 App 层可以避免页面切换时处理器消失。生产实现应检查 fileId 是否属于待确认任务、重复确认是否幂等。若需确认内容正确，应在接收端能用受支持 API 读取并校验文件后再返回业务结果。只依赖 transferred 状态时，结论仅限原生传输任务完成。

BaseApp 处理文件业务确认的局部示例：

~~~js
import { BaseApp } from '@zeppos/zml/base-app'

App(
  BaseApp({
    globalData: {},
    onRequest(req, res) {
      if (req.method === 'file.received' && req.params.fileId === 'report-2026-01') {
        // 这只确认 Side Service 已报告文件到达；内容校验需另行实现。
        res(null, { accepted: true, fileId: req.params.fileId })
        return
      }
      res('unknown method: ' + req.method)
    },
  }),
)
~~~

### App Side 可读文件的边界

不要直接假设 App Side 的 fs.readFile() 可公开使用。ZML v0.0.42 源码存在 src/core/side/file-system 实现，但 src/core/side/zml-side-service.js 默认组合没有注册 file-system plugin，zml.d.ts 也未公开其 Side 实例接口；Zepp OS 官方 Side Service API 页面未提供通用 fs.readFile() 契约。onReceivedFile() 收到 FileObject 不等于 ZML 保证 App Side 能取得字节内容。需要处理文件字节时，必须先查当前 Zepp OS/Zepp App 的正式 API 和所用包导出，再做实机验证；不得用 Device App 的 @zos/fs 假设手机端文件可读。

## 断线、超时与可靠性

- request() 用 try/catch 处理 reject；按任务需要设置 UI 超时/取消状态。源码有内部超时，但超时只表示本地请求等待结束，不代表远端未执行，也不代表消息 session 已清除。
- call() 没有远端业务 ack；关键状态不要 fire-and-forget。为需重试的操作设计幂等 requestId、状态查询、去重和应用层超时。
- BLE disconnect 后不能把任务自动视作安全重试；检查 onBleChanged、设备侧 request/call 错误，并在恢复后重新同步状态。ZML 源码未提供可依赖的断线续传、自动重试、消息持久化或 exactly-once。
- Side Service 的接收确认是反向 request；若 Device App 的消息监听未运行或页面/应用状态不合适，请求可能失败。把可用的应答路由放在 BaseApp，并用 request ID 在恢复后重试或对账；具体后台运行时序需要实机验证。
- TransferFile cancel() 可以取消任务；断开/应用销毁后的恢复、任务重试、重复文件清理、内容 hash/checksum 和持久化队列须由业务按原生 TransferFile 能力与产品要求设计并实机验证。
- 文件 progress 不是业务成功确认。发送端应收到接收方的 receipt request response；接收方应只在相应阶段确认：到达、读取、校验、持久化需要分开表达。

## 其他 Side 模块

BaseSideService 另含实用包装，调用前核对当前安装版本的类型与官方原生 API：

- this.settings：SettingsStorage 的 getItem、setItem、removeItem、clear、getAll；可配合 onSettingsChange({ key, newValue, oldValue }) 响应设置变化。用户偏好适合设置存储，不适合存放大文件或高频队列。
- this.download(url, opts)：封装原生 network.downloader.downloadFile 并返回任务，可监听进度、成功、失败、完成并调用取消；它是在手机侧下载，不是在手表侧下载。目标路径和可读范围以原生 Downloader 文档为准。
- this.convert(...)：封装图像转换能力；输入/输出范围和参数查当前 ZML 类型与上游示例。
- Side Service 默认模块含 messaging、fileTransfer、download、convert；BaseSideService 另附 settings。不要把 ZML 包内源码目录的每个插件都当成默认注册或公开导出。

参考：[Downloader API](https://docs.zepp.com/zh-cn/docs/reference/side-service-api/download-file/)、[Side Service API 总览](https://docs.zepp.com/zh-cn/docs/reference/side-service-api/)、[ZML Side Service 入口](https://github.com/zepp-health/zml/blob/v0.0.42/src/core/side/zml-side-service.js)。

## 选型规则

| 开发需求 | 推荐方案 |
| --- | --- |
| 简单的设备与 App Side 小消息 | 先评估原生 Messaging；若希望统一 request/call、生命周期和回调管理，可选 ZML |
| 复杂请求/响应、双向 RPC | 优先 ZML request/call；建立错误、超时、幂等和业务确认规则 |
| 手表经手机发 HTTP 请求 | 优先评估 ZML httpRequest()；网络调用实际由 Side Service 原生 Fetch 执行 |
| 大型文件传输 | 原生 TransferFile 或 ZML sendFile() 封装；不要用普通 binary request 搬文件 |
| 有界二进制消息 | ZML Binary Messaging；应用自己定义协议并测内存/帧数/断线行为 |
| BLE 外部设备通信 | 使用原生 BLE / 官方推荐 Easy BLE 包；ZML 是 Device App 与手机 Side Service 通信，不是外设 BLE GATT 库 |
| 录音、媒体播放和 VAD | 使用对应原生媒体 API 或独立处理模块 |
| 无手机伴生能力的独立设备功能 | 使用 Device App 原生 API，不为引入 ZML 而增加依赖 |

## 上游源码与官方 API 索引

- [ZML 仓库和 README](https://github.com/zepp-health/zml)
- [v0.0.42 package.json exports](https://github.com/zepp-health/zml/blob/v0.0.42/package.json)
- [TypeScript declarations](https://github.com/zepp-health/zml/blob/v0.0.42/zml.d.ts)
- [BaseApp / BasePage / BaseSideService](https://github.com/zepp-health/zml/tree/v0.0.42/src/core)
- [消息协议分片和重组实现](https://github.com/zepp-health/zml/blob/v0.0.42/src/shared/message.js)
- [Side 文件接收实现](https://github.com/zepp-health/zml/tree/v0.0.42/src/core/side/file-transfer)
- [ZML examples](https://github.com/zepp-health/zml/tree/v0.0.42/examples)
- [Zepp OS Device TransferFile](https://docs.zepp.com/zh-cn/docs/reference/device-app-api/newAPI/transfer-file/TransferFile/)
- [Zepp OS Side Service TransferFile](https://docs.zepp.com/zh-cn/docs/reference/side-service-api/transfer-file/)
- [Zepp OS Side Service Fetch](https://docs.zepp.com/zh-cn/docs/reference/side-service-api/fetch/)
