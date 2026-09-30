# AdeptBLE · dya

Ploopy Adept 的 BLE 改装版（XIAO nRF52840 + PMW3610 轨迹球，6 颗按键）的 ZMK 配置。

`dya` 分支在原来的配置上接入 **DYA Studio**（cormoran 的 ZMK Studio 增强版）。
`main` 分支保持原样：badjeff 的轨迹球驱动 + 官方 ZMK Studio。

## 与 main 分支的区别

| 项目 | main | dya |
| --- | --- | --- |
| ZMK | `zmkfirmware/zmk@main` | `cormoran/zmk@main+dya`（Custom Studio Protocol） |
| Zephyr | 随 ZMK 决定 | `cormoran/zephyr@v4.1.0+zmk-fixes+nrf-half-duplex-uart` |
| 轨迹球驱动 | badjeff `pixart,pmw3610` | cormoran `cormoran,pmw3610` |
| Studio | 官方 ZMK Studio | 官方功能 + DYA Studio（轨迹球 / 连接 / 设置 / 宏 / 组合键 / 诊断） |
| 板级 target | `seeeduino_xiao_ble` | `xiao_ble//zmk`（Zephyr 4.1 / HWMv2 的名称） |

## 已启用的 DYA Studio 功能

* **Keymap**：键位 / 层编辑；布局预览里会画出轨迹球位置；Macro 页可创建、改名、删除运行时宏，在键位上用 `&rmacro <槽位>` 播放；Combo 页可编辑运行时组合键
* **Trackball**：CPI、轴方向、smart algorithm、downshift / sample 等参数在线调整；运行时可调的输入处理器（速度、旋转、轴吸附、active layers）
* **Connection**：BLE profile 管理、OS 自动识别、按连接 / OS 切换默认层
* **Settings**：idle 超时等设置、通用 custom settings、电池历史
* **Troubleshooting**：device info、watchdog 重启原因

## 使用 DYA Studio

1. 用 USB 线连接键盘
2. 打开 <https://studio.dya.cormoran.works/>，选 Connect via USB
3. 固件关闭了 Studio 锁定（`CONFIG_ZMK_STUDIO_LOCKING=n`）：锁状态一开始就是 unlocked，这个 ZMK fork 也没有 lock / unlock 的 RPC，自动锁定（idle 超时、BLE 断开）都在编译期关掉了，所以**键盘永远处于 unlocked**，不需要按任何解锁键

### 键位上的新增绑定（只在 Device 层，原有按键不受影响）

Device 层的触发键是第 2 颗键（原来的 `MB5`）。按住它之后：

| 按键 | 作用 |
| --- | --- |
| 第 1 颗（左上大键） | `&bootloader`（原有） |
| 第 4 颗（中间偏右小键） | `&rmacro 0`（播放运行时宏槽位 0，空槽位无动作） |
| 第 6 颗（右下宽键） | `&bt BT_CLR`（原有） |

## 构建 / 烧录

GitHub Actions 的 `Build ZMK firmware` 工作流会构建两个固件：

| 文件 | 用途 |
| --- | --- |
| `AdeptBLE.uf2` | 正常固件 |
| `AdeptBLE_reset.uf2` | 刷一次即可清空所有已保存的设置（键位、custom settings、电池历史、BLE 配对），之后需要再刷回正常固件 |

## 注意事项

* 轨迹球在布局预览里的位置是估算值，改 `boards/shields/AdeptBLE/AdeptBLE.overlay` 中 `trackball_layout` 的 `x` / `y` / `size` 即可
* 轨迹球的静态方向处理（X 反向、XY 互换）仍写在 overlay 里，运行时处理器排在它后面，所以 DYA Studio 里的速度 / 旋转等参数作用在修正后的坐标上；再在 DYA Studio 里打开 swap / invert 会叠加在静态处理之上
* 已开启 deep sleep（`CONFIG_ZMK_SLEEP=y`，默认 30 分钟无操作）：**只在用电池时才会睡**，插着 USB 时 ZMK 不会进入深睡，所以插线调 DYA Studio 不会被打断；睡眠时间可以在 DYA Studio 的 Settings 页改（设成 0 就是永不睡），改完会保存。唤醒方式是按任意键（kscan 是 wakeup source）
* 电池历史会定期写 flash（默认约 2 小时一次，且电量有明显变化时才写）；不想要的话删掉 `CONFIG_ZMK_BATTERY_HISTORY*` 两行

---

## English

The `dya` branch adds [DYA Studio](https://studio.dya.cormoran.works/) support
(cormoran's enhanced ZMK Studio) to this AdeptBLE config; `main` keeps the
original badjeff trackball driver and official ZMK Studio setup.

* ZMK/Zephyr come from `cormoran/zmk@main+dya` and
  `cormoran/zephyr@v4.1.0+zmk-fixes+nrf-half-duplex-uart`
* The trackball uses the DYA-compatible `cormoran,pmw3610` driver and DYA
  runtime input processors, so speed / rotation / axis snap / CPI and the
  active layers can be changed from the web UI
* Studio locking is disabled and cannot be re-enabled from a client (no
  lock/unlock RPC), so the keyboard is always unlocked — no `&studio_unlock`
  binding is needed
* Deep sleep is enabled (`CONFIG_ZMK_SLEEP=y`, 30 minute default) and only
  happens on battery; while USB is connected ZMK never sleeps, so a Studio
  session is not interrupted. The timeout can be changed from DYA Studio's
  Settings tab
* Build artifacts: `AdeptBLE.uf2` and `AdeptBLE_reset.uf2` (wipes stored
  settings once, for starting over from the firmware defaults)
