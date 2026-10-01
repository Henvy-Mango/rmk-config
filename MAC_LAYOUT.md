# Mac 键位、Vial 与休眠参数

## 设计依据

参考 [Keychron V4 官方组合键表](https://www.keychron.com/blogs/news/v4-key-combinations)的媒体键顺序，以及 [Keychron 在 QMK 上的 Mac/Windows 分层实现](https://github.com/qmk/qmk_firmware/blob/master/keyboards/keychron/c3_pro/ansi/rgb/keymaps/default/keymap.c)。按用户要求，保留 Fn + 数字行 = F1–F12，另外提供 Mac 多媒体层。

这是适配 ggbr 66 键布局的实现；板上没有 Keychron 的 Mac/Win 拨动开关，仍用原来的软件切层键。

## 层与入口

| Vial 层号 | 名称 | 入口 |
| --- | --- | --- |
| 0 | Windows 基础层 | 按住 Caps，再按 `[` |
| 1 | Mac 基础层 | 按住 Caps，再按 `]` |
| 2 | Windows Fn | Windows 模式下按住任意 Fn |
| 3 | 宏/切换层 | 按住 Caps，或右侧 tap dance |
| 4 | Mac Fn | Mac 模式下按住任意 Fn |
| 5 | Mac 多媒体 | Mac 模式下先按住 Fn，再按住空格 |

Fn 是空格两侧原有的两个 Fn 位置。多媒体层在空格松开时退出，Fn 松开时退出 Fn 层。Fn 必须先于空格按下；先按空格仍然输入空格。Mac/Windows 切换使用 TO(0)/TO(1)，不是开机自动识别操作系统，也不承诺跨重启记住选择。

左侧修饰键保持 Control、Option、Command；右侧两个修饰键调整为靠近空格的 Command、外侧的 Option，参考 Keychron 的 Mac Command 布局。外侧 Option 保留原先 tap dance 的双击/点按后长按进入宏层动作。Windows 层保持原来的修饰键和 Fn 动作。

## Mac 多媒体层

同时按住 **Fn + 空格**，再按下面的键：

| 按键 | 动作 |
| --- | --- |
| 1 / 2 | 屏幕亮度降低 / 提高 |
| 3 | Mission Control |
| 4 | Launchpad |
| 5 / 6 | 无动作：尚无 RGB/背光驱动，避免误触 F5/F6 |
| 7 / 8 / 9 | 上一曲 / 播放暂停 / 下一曲 |
| 0 / - / = | 静音 / 音量降低 / 音量提高 |

亮度、Mission Control 和 Launchpad 使用 RMK v0.9 已有的 HID 消费者键码。主机系统版本、设置及显示器对这些功能的支持需要实机验证；固件编译通过不代表所有 macOS 版本都保留同名功能。

## vial.json 更新范围

本次名称由 HID Keyboard 改为 ggbr minila。66 个键的物理尺寸与矩阵坐标、VID/PID 和 7 个蓝牙 User 键码顺序经核对后保留。

`vial.json` 描述键盘外形、矩阵坐标和自定义键码标签；每层按键动作与层数来自 `keyboard.toml`，无需在 JSON 中复制六层键位。BT0/1/2 对应 User0/1/2，Next/Prev/Clear/Switch 对应 User3/4/5/6。

若刷机后 Vial 仍显示旧的自定义键位，是设备中保存的动态布局覆盖了编译默认值。先导出原 Vial 布局备份，再使用固件支持的重置方式恢复新默认键位；没有在本次固件中启用自动清空存储。若使用全存储清除，会同时影响配对等数据，不应把它当作只刷新界面。

## split_central_sleep_timeout_seconds

当前为 **900 秒（15 分钟）**；默认 **0** 表示关闭这个软件休眠管理器。

尽管名字带 split_central，RMK v0.9 的 BLE transport 也会为单体键盘运行该管理器。每次键盘输入活动都会重置计时；超时后设置 sleeping 状态并广播 SleepStateEvent，活动恢复时清除该状态。主机休眠或广播超时也可以请求提前进入该状态。

对这块无显示屏的单体键盘，明确可见的作用之一是：休眠时不再通过 30 分钟超时路径触发 BLE 电量通知。它不等于停止所有电池 ADC 采样，也不等于断开所有 BLE 连接。带显示屏或分体功能的配置可另外使用此事件关闭显示或调整分体链路参数。

**这个参数本身不使 nRF52840 进入 System OFF，不是自动关机，也不是 P1.09 外设电源的定时开关。** P1.09 在本配置中初始化后已保持关闭。实际节电幅度需要测量，不能据此保证待机电流。

源码依据：[休眠管理器](https://github.com/rmk-rs/rmk/blob/rmk-v0.9.0/rmk/src/ble/sleep.rs)、[电量通知](https://github.com/rmk-rs/rmk/blob/rmk-v0.9.0/rmk/src/ble/battery_service.rs)。本次保留 900 秒，没有因解释参数而更改其数值。
