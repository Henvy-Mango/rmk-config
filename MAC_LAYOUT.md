# 按 SKN Launcher 调整 Mac / Windows 分层

## 已查看的参考

用户连接的 SKN Launcher：QingLong 87 Ultra 8K ANSI，2026-10-02，逐层查看 Layer 0、1、2、3。只切换显示层查看，没有修改这台 SKN 键盘的键位。

| SKN 层 | 看到的结构 |
| --- | --- |
| 0 | 完整 Mac 基础层；左侧 Ctrl/Opt/Cmd，右侧 Cmd/Opt；Fn 是 MO(1)；独立 F 区是桌面切换、MCtl、LPad、背光与媒体键 |
| 1 | Mac Fn；独立 F 区为 F1–F12；多数其他键为透明键；有蓝牙、WIN/Mac、RGB、电量等厂商功能 |
| 2 | 完整 Windows 基础层；Ctrl/Win/Alt；Fn 是 MO(3)；独立 F 区为 F1–F12 |
| 3 | Windows Fn；独立 F 区为亮度、Task、File、背光与媒体键；多数其他键为透明键 |

参考中 Mac 顶排前两键实际标注 DESKTOP_L-m / DESKTOP_R-m，并非屏幕亮度。本次参考的是可见键位与层关系，没有声称读取了 SKN 的底层固件实现。

## 当前 ggbr 的层结构

按用户最新要求：**不新增媒体键，只保留原来已有的媒体功能。** ggbr 没有独立 F 区，所以继续保留数字行，Fn + 数字行为 F1–F12。

| Vial 层号 | 名称 | 进入方式 |
| --- | --- | --- |
| 0 | Mac 基础层 | 宏层按 `]`，即 PDF(0) |
| 1 | Mac Fn | Mac 模式按住任意 Fn，即 MO(1) |
| 2 | Windows 基础层 | 宏层按 `[`，即 PDF(2) |
| 3 | Windows Fn | Windows 模式按住任意 Fn，即 MO(3) |
| 4 | 原宏层 | 按住 Caps，或右侧 tap dance |

0、1、2、3 与用户展示的系统配对顺序一致；额外的第 4 层仅用于保留原宏层及已有媒体键，不是新的媒体层。原来的 Fn + 空格媒体入口和独立媒体层均已移除。

- Mac 与 Windows 基础层都写出完整 66 键，不再让 Mac 大量透明键依赖 Windows 基础层。
- Fn 层使用透明键继承所选系统基础层；两个 Fn 层当前都沿用原 Fn 键位，未添加 SKN 的额外媒体/背光/2.4G/电量显示/锁 Win 功能。
- Mac 左侧为 Ctrl/Option/Command，右侧两个修饰键为 Command/Option；Windows 保持 Ctrl/Win/Alt 及右 Alt/Ctrl。因键数不同，不复制 SKN 全部 87 个物理位置。
- 左右两个 Fn 均保留；Caps 点按为 Caps Lock，长按进宏层。右侧 tap dance 的原宏层入口改为 MO(4)。
- 使用 PDF(0)/PDF(2) 切换并保存系统基础层；模式切换后松开 Caps/所有层键再继续输入。首次无旧存储时从 Mac 基础层 0 启动，后续按保存的基础层启动。不是自动识别主机系统。

## 保留的原有媒体键

| 操作 | 功能 |
| --- | --- |
| Fn + 左方向键 | 静音 |
| Fn + 下方向键 | 音量降低 |
| Fn + 右方向键 | 音量提高 |
| 宏层 + 左方向键 | 上一曲 |
| 宏层 + 下方向键 | 播放/暂停 |
| 宏层 + 右方向键 | 下一曲 |

原来的两个文本宏、蓝牙前后档位、导航及复制/粘贴组合继续保留。没有把原宏层合并进 Fn 并重新安排这些按键。

## vial.json 与已有存储

`vial.json` 继续使用 ggbr minila 名称、已核对的 66 键物理布局和 7 个蓝牙自定义键码。层数及动作由 `keyboard.toml` 提供，Vial 应读取五层。

本次层号重新排列；刷机前先导出当前 Vial 配置备份。旧动态布局若仍被保留，会覆盖编译默认值，需要恢复新默认键位。不要直接把旧五/六层布局不作调整地导回去，也不要仅按层数判断版本：旧项目与当前五层可能含义不同。

本次未启用自动清空存储；全存储清除还可能删除蓝牙配对信息。完成初始化后可通过“按住 Caps + ]”选择并保存 Mac，或“按住 Caps + [”选择并保存 Windows。

## split_central_sleep_timeout_seconds

当前为 **900 秒（15 分钟）**；默认 **0** 表示关闭这个软件休眠管理器。

尽管名字带 split_central，RMK v0.9 的 BLE transport 也会为单体键盘运行该管理器。每次键盘输入活动都会重置计时；超时后设置 sleeping 状态并广播 SleepStateEvent，活动恢复时清除该状态。主机休眠或广播超时也可以请求提前进入该状态。

对这块无显示屏的单体键盘，明确可见的作用之一是：休眠时不再通过 30 分钟超时路径触发 BLE 电量通知。它不等于停止所有电池 ADC 采样，也不等于断开所有 BLE 连接。带显示屏或分体功能的配置可另外使用此事件关闭显示或调整分体链路参数。

**这个参数本身不使 nRF52840 进入 System OFF，不是自动关机，也不是 P1.09 外设电源的定时开关。** P1.09 在本配置中初始化后已保持关闭。实际节电幅度需要测量，不能据此保证待机电流。

源码依据：[休眠管理器](https://github.com/rmk-rs/rmk/blob/rmk-v0.9.0/rmk/src/ble/sleep.rs)、[电量通知](https://github.com/rmk-rs/rmk/blob/rmk-v0.9.0/rmk/src/ble/battery_service.rs)。本次保留 900 秒，没有因解释参数而更改其数值。
