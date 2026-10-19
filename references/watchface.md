# Watchface 表盘开发

## 与小程序分开处理

Watchface 有专用模板、app.json 类型、工具和 API 文档。不要直接套用 Mini Program 的 Device App 生命周期和 @zos 控件 API；先查看项目模板与实际目标，再从表盘专用接口实现。

- [表盘快速开始](https://docs.zepp.com/zh-cn/docs/watchface/watchface-quick-start/)
- [表盘 app.json](https://docs.zepp.com/zh-cn/docs/watchface/app-json/)
- [表盘 API](https://docs.zepp.com/zh-cn/docs/watchface/api/)
- [表盘规范](https://docs.zepp.com/zh-cn/docs/watchface/specification/)
- [表盘资源](https://docs.zepp.com/zh-cn/docs/watchface/design-resources/)
- [表盘制作工具](https://docs.zepp.com/zh-cn/docs/guides/tools/watchface/)

表盘 API 家族包含 hmUI、hmSensor、hmSetting、hmFS、hmBle 和 timer；具体 API 应以对应目标文档为准。可视化表盘制作工具和 Zeus CLI 源代码项目是两条不同的制作流程。

## 表盘实现检查

- 优先级是时间易读，再选择少量健康/运动/天气/系统数据；数据要能区分“无数据”和真实的 0。
- 用设备表核对表盘目标设备、屏幕分辨率、预览分辨率和 API 能力。适配字体、时间制式、单位和语言。
- 检查圆屏裁切、边缘 2px 安全距离，以及顶部/底部状态点、离线语音区域；重要文字与数据不要放在这些区域。
- 表盘息屏模式应是独立检查项：黑底为主、移除秒针、限制常亮像素、尽量不让亮屏/息屏组件跳动；具体规则按官方规范和目标设备核实。
- 预览图使用规定的默认数据，格式化最大位数和负温度、12/24 小时、华氏/摄氏、星期/月缩写、地区差异。
- 控制动画、传感器更新和高亮像素，优先考虑耗电。确认健康数据权限、刷新行为和无传感器/无天气数据表现。
- 用目标机模拟器检查截图，并在真实设备检查息屏、常亮、传感器与预览资源。

设计指南中的数据最大位数、默认预览值和息屏像素规范可能随平台更新；生成提交包前重读[最新表盘规范](https://docs.zepp.com/zh-cn/docs/watchface/specification/)及[发布表盘](https://docs.zepp.com/zh-cn/docs/distribute/watchface/)。

