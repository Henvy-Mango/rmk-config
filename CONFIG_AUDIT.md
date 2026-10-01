# ggbr：RMK v0.9 配置核对

核对对象：原仓库 `config/ggbr.keymap`、`config/ggbr.conf`、`boards/arm/ggbr`，用户提供的主控原理图，以及 RMK 官方 `rmk-v0.9.0` 源码和云端模板。旧 ZMK 的 `west.yml` 使用 `v0.3`，因此按该版本的默认值核对时序。硬件接线以用户实际打样所依据的 `ggbr___ggbr_PCB__20250506234707` 制造文件为准；已与 2026-10-02 从当前 EDA 工程导出的 Gerber 和 IPC 网络文件对比。

## 当前策略：通用行为参数使用 RMK 默认值

按用户最新偏好，撤销此前为贴近 ZMK 添加的时序和 Morse 判定覆盖；保留硬件配置、键位定义和 Mac 层组合键可用性。默认值按 RMK `rmk-v0.9.0` 源码核对。

| 项目 | 此前迁移配置 | 现在 |
| --- | --- | --- |
| 消抖 | 显式 5 ms | 删除覆盖，使用 RMK 默认 20 ms |
| Caps Lock 点按/长按 | 显式 200 ms、normal 模式 | 删除覆盖，使用 RMK 默认 Normal 模式，hold/gap 各 250 ms |
| 右 Ctrl/GUI tap dance | 200 ms、独立 `hold_on_other_press` profile | 删除专用 profile，使用 RMK 默认判定和 hold/gap 各 250 ms；保留双击/点按后长按的 MO(3) 动作 |
| 组合键窗口 | 显式 50 ms（更早的配置曾为 150 ms） | 删除覆盖，RMK v0.9 默认也是 50 ms |
| 组合键层限制 | 仅 layer 0 | 删除限制，使 Mac 层也可触发 |
| 主机配置解锁 | 默认 Vial 锁定 | `host.insecure = true`，沿用旧 `CONFIG_ZMK_STUDIO_LOCKING=n`；连接的主机无需实体解锁组合即可使用受保护的 Vial 操作 |
| 蓝牙档位数量 | 隐式默认 3 | 显式固定 `ble_profiles_num = 3`，使 User3～User6 的含义稳定 |

## 已有配置与模板默认项

| 项目 | 核对结果 |
| --- | --- |
| 主控 | nRF52840 / E73-2G4M08S1C |
| 矩阵 | 5×14，66 个实际键位；col2row；引脚与旧配置及打样 Gerber 一致，新旧 PCB 的 COL9 差异见下方 |
| 四层键位、宏、多媒体键 | 已迁移；Grave/Escape 通过 fork 保留 ZMK 的 Shift/GUI 条件及修饰键抑制方式 |
| USB + BLE | 已启用；nRF52840 的默认芯片配置启用 USB |
| 低功耗扫描 | 官方 nRF52840 Cargo 模板已启用 `async_matrix`，不必另加 TOML 开关 |
| NFC 引脚作为 GPIO | 官方模板已启用 `embassy-nrf/nfc-pins-as-gpio`，适用于旧配置中的 P0.09 |
| REG0 / REG1 | `dcdc_reg0 = false`、`dcdc_reg1 = true`；保持外部 3.3 V LDO 供电方案 |
| 外设供电 | P1.09 active-low，上电初始化后输出高电平，关闭不用的 RGB 供电 |
| Caps Lock 指示灯 | P0.05，active-high；自定义 `leds.c` 按要求忽略 |
| 电池 | P0.04，分压 2000 / 2820，与 2 MΩ + 820 kΩ 电路一致；电量百分比和高阻分压的采样准确度仍需实测 |
| 蓝牙功率 | +8 dBm，沿用旧配置；没有为了省电擅自降低 |
| 蓝牙 PHY | 保留 RMK 的 2M PHY 默认值；旧适配器连接问题再有针对性调整 |
| 空闲休眠 | 900 秒；官方 idle manager 支持单体 BLE 键盘，尽管参数名带 split |
| 存储 | Cargo 默认启用 storage；nRF52840 默认地址 0xA0000、32 个 4 KiB 扇区；使用 RMK 格式，不能沿用 ZMK NVS 数据 |
| Vial | 默认 Cargo 功能已启用；`vial.json` 保留实际物理布局 |
| 看门狗 | RMK v0.9 默认 Cargo 功能已启用，无需新增开关 |
| 低频时钟 | 官方 BLE 初始化使用内部 RC，与原 ZMK 的 K32SRC_RC 一致；旧板有外部 32.768 kHz 晶振焊盘，但没有新版 C5/C6 外接负载电容焊盘；v0.9 芯片 TOML 未提供 LFCLK 来源选项 |
| 云端编译 | 官方 `user_build.yml@rmk-v0.9.0`，传入 `rmk_version: "0.9"` |

## 仍然存在的迁移差异

1. **NKRO**：旧 ZMK 显式开启 NKRO。RMK v0.9 的 HID 报告使用 `[u8; 6]` 普通键数组，修饰键另计；没有可补的 NKRO TOML 开关。
2. **RGB**：官方配置式模板没有这块板的 66 颗 WS2812 驱动。Fn 的 RGB 开关/效果键现在为透明键，外设电源保持关闭；要恢复需要额外驱动代码。
3. **深度关机**：原 Fn+Delete 进入 soft-off、H 唤醒，没有迁移为等效功能。RMK 的 900 秒 idle manager 发布休眠状态、暂停电池报告等，不能据此声称它进入了 nRF System OFF。原 30 秒 ZMK idle 阶段也无直接对应项。
4. **清空全部蓝牙档位**：原四键 `BT_CLR_ALL` 没有等价单个动作。可在 Vial 给按键分配 `User5`，逐档清除当前配对；没有替换成会抹掉键位设置的整库重置。ZMK 的配对信息不会迁移，需要在主机端忘记旧设备后重新配对。
5. **组合键语义**：ZMK 按物理位置匹配，RMK 按当前解析的动作匹配。删除层限制修好了 Mac 层，但如果后续重映射 B/U/Shift 等键，组合键不会自动跟随原物理位置。
6. **Tap dance 精确时序**：ZMK 两动作 tap dance 在第二次按下时立即激活 MO(3)；RMK 使用 Morse 判定，解析时机取决于 Morse 模式、按键释放和超时。当前按用户偏好使用默认 Normal 模式，不再用独立的 hold-on-other-press profile 对齐 ZMK；默认点按/长按窗口为 250 ms，两种状态机并非完全相同。
7. **复位期间的 RGB 供电**：原理图 R6 为 2 MΩ 下拉，MCU 尚未初始化时 Q3 默认导通。固件只负责初始化后关闭，无法替代硬件上拉改版。

## 已确认的打样板接线及版本差异

打样 Gerber 中，COL9 铜箔连接 **E73 41 脚 / P0.09**，并连到 K10、K24、K38、K51、K62；当前 EDA PCB 则改接 **42 脚 / P1.06**。因此本分支保留旧 ZMK 的 P0.09，已经有制造文件依据。该列对应基础层的 `9`、`O`、`L`、`.`、右 Alt。

其余 13 列和 5 行的 MCU 引脚，以及全部 66 个按键的逻辑矩阵位置保持一致；H 键热插拔座/灯位方向有变化，但仍为 ROW2 / COL6。外设供电 P1.09、电池 ADC P0.04、指示灯 P0.05、RGB 数据 P0.26 均保持一致。

新版还增加了 LDO 输入/输出电容和晶振外接负载电容，调整了电池/USB 电容，并更换电源开关封装、移动多个器件。不能把新版原理图的所有外围器件视为旧板已经具备。这些硬件差异不需要改变本分支现有引脚值，也不能通过 REG0/REG1 软件设置补偿。

制造文件能确认设计接线，不能确认打样后的飞线、补焊或实装阻容值；编译成功也不能证明待机电流、ADC 精度或 bootloader 与实物一致。

## 不需要为本板添加的可选功能

分体、dongle、显示屏、编码器、鼠标传感器、三层联动、单次修饰键、自动鼠标层均没有对应的原硬件或键位需求。Rynk、配对密码输入和 embassy-boot DFU 也不是迁移必需项，部分还需要额外 Cargo 功能，不能只填 TOML 开关。保持官方 Vial / Adafruit UF2 模板。

## 核对依据

- [RMK 配置目录](https://rmk.rs/docs/configuration/)
- [行为与时序](https://rmk.rs/docs/configuration/behavior)
- [低功耗与 External VCC](https://rmk.rs/docs/features/low_power)
- [芯片设置](https://rmk.rs/docs/configuration/chip_config)
- [无线配置](https://rmk.rs/docs/configuration/wireless)
- [官方云端编译](https://rmk.rs/docs/user_guide/create_firmware/cloud_compilation.html)
- [v0.9.0 HID 报告实现](https://github.com/rmk-rs/rmk/blob/rmk-v0.9.0/rmk/src/hid.rs)
- [v0.9.0 默认 Cargo 功能](https://github.com/rmk-rs/rmk/blob/rmk-v0.9.0/rmk/Cargo.toml)
- [ZMK v0.3 layer-tap 默认值](https://github.com/zmkfirmware/zmk/blob/v0.3/app/dts/behaviors/layer_tap.dtsi)
- [ZMK v0.3 tap-dance 默认值](https://github.com/zmkfirmware/zmk/blob/v0.3/app/dts/bindings/behaviors/zmk%2Cbehavior-tap-dance.yaml)
- [ZMK v0.3 combo 默认值](https://github.com/zmkfirmware/zmk/blob/v0.3/app/dts/bindings/zmk%2Ccombos.yaml)
- [ZMK v0.3 消抖默认值](https://github.com/zmkfirmware/zmk/blob/v0.3/app/module/drivers/kscan/Kconfig)

- [v0.9.0 Morse 与 Combo 默认值](https://github.com/rmk-rs/rmk/blob/rmk-v0.9.0/rmk/src/config/behavior.rs)
- [v0.9.0 消抖默认值](https://github.com/rmk-rs/rmk/blob/rmk-v0.9.0/rmk-config/src/lib.rs)
