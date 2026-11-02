# MECHREVO GM5AR5E — macOS 15.6 (Sequoia) OpenCore EFI

**English** | [中文说明](#中文说明)

---

## English

A working OpenCore EFI for the **MECHREVO JIGUANGE GM5AR5E** laptop, booting **macOS 15.6 Sequoia** with the
[NullMoth NVIDIA driver](https://github.com/nullmoth/nvidia-macos-driver) (RTX 4050 Laptop).
Shared so that owners of the same or similar hardware do not have to repeat the debugging.

> **Generate your own SMBIOS before using this.** `SystemSerialNumber`, `MLB` and `SystemUUID` in
> `EFI/OC/config.plist` are placeholders. Use OpenCore Configurator / OCAuxiliaryTools / GenSMBIOS
> (model `MacBookPro16,2`) and replace them with values generated for you.

### Hardware

| Part | Model / ID |
|---|---|
| Laptop | MECHREVO JIGUANGE SERIES GM5AR5E, BIOS `N.1.13MRO14` |
| CPU | Intel Core i5-12450H (Alder Lake-H, 4P+4E, 8C/12T) |
| dGPU | NVIDIA GeForce RTX 4050 Laptop, `10DE-28A1` — **internal eDP panel wired to the dGPU** (MUX discrete mode) |
| Ethernet | Realtek RTL8111 (`10EC-8168`) |
| Wi-Fi | Intel AX101 (`8086-51F0`) |
| Bluetooth | Intel AX101 BT (`8087-0026`) |
| Audio | Realtek ALC256 (`10EC-0256`) |
| Storage | NVMe: MAP1602 (`1E4B-1602`) / PC300 (`1E49-1031`) |

### Software

| Item | Version |
|---|---|
| macOS | 15.6 Sequoia (tested) |
| OpenCore | 1.0.9 (RELEASE) |
| NullMoth NVIDIA driver | 2.6.6 |

### What is in this EFI

- **ACPI**: native tables + 8 SSDTs (SBUS, EC, MCHC, PLUG-ALT, RTCAWAC, USB-Reset, USBX, XOSI) + `SSDT-PNLF.aml`.
  **The DSDT is NOT replaced** — a replacement DSDT also affects Windows and broke its boot.
- **Kernel patches (5)** — the set that gets past `EXITBS:START` on this CPU:
  - `force HT enabled for …` (3 patches, `_cpu_thread_alloc`) — kept enabled
  - `ProvideCurrentCpuInfoTopologyValidation` — `_x86_validate_topology` → `RET`
  - `Force nPThreadsPerCore=1 for Alder Lake 4P+4E` — patches `cpuid_cores_per_package` to a constant
  - `ProvideCurrentCpuInfo` = **True** (required by the two patches above)
- **Kexts (26)**: Lilu, AMFIPass, AppleALC, BlueToolFixup, BrightnessKeys, ECEnabler, IntelBTPatcher,
  IntelBluetoothFirmware, NVMeFix, RealtekRTL8111, RestrictEvents, VirtualSMC (+SMC plugins), USBToolBox,
  UTBDefault, VoodooPS2 family, WhateverGreen, XHCI-unsupported, itlwm
- **boot-args**: driver parameters + `applbkl=3` + `nvrmacpibl=1` (experimental backlight switches)
- `boot-args` is present in `NVRAM → Delete` — **without this the boot-args never reach the kernel**
  (NVRAM `keep-boot-args` rule; OpenCore logs `boot-args - ignored, exists`)
- SMBIOS: `MacBookPro16,2`, `SecureBootModel = Disabled`, `csr-active-config = 0x0A43`

### How to use

1. Make a macOS 15.6 installer stick with [1401](https://github.com/nullmoth/1401) (`Build` → macOS 15).
2. Replace that stick's `EFI/OC` with the `EFI/OC` here (keep the stick's own `EFI/BOOT`).
3. Generate a fresh SMBIOS (`MacBookPro16,2`) and put it into `EFI/OC/config.plist`.
4. Boot the stick, install macOS.
5. Install the NullMoth driver **2.6.6** with the 1401 Mac app (or `pkgroot/install.sh`), then reboot.

### What works

- Boot, install, desktop, Metal acceleration on the RTX 4050 (NullMoth 2.6.6)
- Built-in keyboard / trackpad buttons (VoodooPS2), USB
- Ethernet (RTL8111), Bluetooth (Intel)
- Audio (ALC256, layout-id `13`)
- Battery (SMCBatteryManager + ECEnabler), brightness keys (BrightnessKeys)
- **Brightness control, sleep/wake and full-screen performance** — all confirmed working on this machine
- Wi-Fi **via [HeliPort](https://github.com/OpenIntelWireless/HeliPort)** only — itlwm is not an Airport interface,
  so macOS Settings has no Wi-Fi menu. During the installer there is no network over Wi-Fi; use Ethernet.

### Known limitations

- **Trackpad (MCHP1930) does not work.** The internal trackpad is an I2C HID device with a vendor `_HID`
  (`MCHP1930`) and a `_STA` that returns 0 under macOS. Adding VoodooI2C 2.8/2.9 with a patched DSDT
  (`_CID = PNP0C50`, `_STA = 0x0F`) caused a kernel panic (`va_map_delete` in `com.alexandred.VoodooI2C`),
  so the trackpad is left disabled. Use a USB mouse.

### Credits

- [NullMoth](https://github.com/nullmoth) — 1401 and the NVIDIA driver (the whole reason this is possible)
- [OpenIntelWireless](https://github.com/OpenIntelWireless) — itlwm / HeliPort
- [@naevui](https://github.com/naevui) — his OpenCore config supplied the two Alder Lake 4P+4E topology patches
- Acidanthera — OpenCore, Lilu, VirtualSMC, WhateverGreen, AppleALC, BrightnessKeys, USBToolBox, VoodooPS2

---

## 中文说明

**机械革命 极光E GM5AR5E（2023款）** 笔记本的 OpenCore EFI，使用
[NullMoth NVIDIA 驱动](https://github.com/nullmoth/nvidia-macos-driver)（RTX 4050 Laptop）启动 **macOS 15.6 Sequoia**。

> **使用前必须生成你自己的 SMBIOS。** `EFI/OC/config.plist` 里的 `SystemSerialNumber`、`MLB`、`SystemUUID`
> 都是占位符。请用 OpenCore Configurator / OCAuxiliaryTools / GenSMBIOS（机型 `MacBookPro16,2`）生成你自己的值替换。

### 硬件配置

| 部件 | 型号 / ID |
|---|---|
| 笔记本 | 机械革命 极光E GM5AR5E，BIOS `N.1.13MRO14` |
| CPU | Intel Core i5-12450H（Alder Lake-H，4P+4E，8C/12T） |
| 独显 | NVIDIA GeForce RTX 4050 Laptop，`10DE-28A1` — **内屏 eDP 直连独显**（MUX 独显直连模式） |
| 有线网卡 | Realtek RTL8111（`10EC-8168`） |
| 无线网卡 | Intel AX101（`8086-51F0`） |
| 蓝牙 | Intel AX101 BT（`8087-0026`） |
| 声卡 | Realtek ALC256（`10EC-0256`） |
| 硬盘 | NVMe：MAP1602（`1E4B-1602`）/ PC300（`1E49-1031`） |

### 软件版本

| 项目 | 版本 |
|---|---|
| macOS | 15.6 Sequoia（已测试） |
| OpenCore | 1.0.9（RELEASE） |
| NullMoth NVIDIA 驱动 | 2.6.6 |

### 这份 EFI 里有什么

- **ACPI**：原生表 + 8 个 SSDT（SBUS、EC、MCHC、PLUG-ALT、RTCAWAC、USB-Reset、USBX、XOSI）+ `SSDT-PNLF.aml`。
  **没有替换 DSDT**——替换 DSDT 会同时影响 Windows，导致 Windows 无法启动。
- **内核补丁（5 条）**——这是让这台 CPU 通过 `EXITBS:START` 的关键组合：
  - `force HT enabled for …`（3 条，`_cpu_thread_alloc`）——保持启用
  - `ProvideCurrentCpuInfoTopologyValidation`——`_x86_validate_topology` → `RET`
  - `Force nPThreadsPerCore=1 for Alder Lake 4P+4E`——把 `cpuid_cores_per_package` 补成常量
  - `ProvideCurrentCpuInfo` = **True**（上面两条补丁需要）
- **Kexts（26 个）**：Lilu、AMFIPass、AppleALC、BlueToolFixup、BrightnessKeys、ECEnabler、IntelBTPatcher、
  IntelBluetoothFirmware、NVMeFix、RealtekRTL8111、RestrictEvents、VirtualSMC（含 SMC 插件）、USBToolBox、
  UTBDefault、VoodooPS2 全家、WhateverGreen、XHCI-unsupported、itlwm
- **boot-args**：驱动参数 + `applbkl=3` + `nvrmacpibl=1`（实验性背光开关）
- `boot-args` 已放进 `NVRAM → Delete`——**不放的话启动参数永远到不了内核**
  （NVRAM 的 `keep-boot-args` 规则；OpenCore 日志会打印 `boot-args - ignored, exists`）
- SMBIOS：`MacBookPro16,2`，`SecureBootModel = Disabled`，`csr-active-config = 0x0A43`

### 使用方法

1. 用 [1401](https://github.com/nullmoth/1401)（Windows）制作 macOS 15.6 启动盘（`Build` → 选 macOS 15）。
2. 把本包的 `EFI/OC` 替换到 U 盘的 `EFI/OC`（保留 U 盘自己的 `EFI/BOOT`）。
3. 生成新的 SMBIOS（机型 `MacBookPro16,2`）并填入 `EFI/OC/config.plist`。
4. 用 U 盘启动并安装 macOS。
5. 用 1401 Mac app（或 `pkgroot/install.sh`）安装 NullMoth 驱动 **2.6.6**，然后重启。

### 已工作

- 启动、安装、桌面、RTX 4050 的 Metal 加速（NullMoth 2.6.6）
- 内置键盘 / 触摸板按键（VoodooPS2）、USB
- 有线网卡（RTL8111）、蓝牙（Intel）
- 声卡（ALC256，layout-id `13`）
- 电池（SMCBatteryManager + ECEnabler）、亮度键（BrightnessKeys）
- **亮度调节、睡眠唤醒、全屏性能**——均已在真机上确认正常
- Wi-Fi 需要 **[HeliPort](https://github.com/OpenIntelWireless/HeliPort)**——itlwm 不是 Airport 接口，
  系统设置里没有 Wi-Fi 菜单。安装器阶段无法用 Wi-Fi 联网，请用有线网络。

### 已知限制
//折腾两三次一直panic我也不想折腾了，平常不常用触摸板，如果有高人能修一下就更好了
- **触摸板（MCHP1930）不可用。** 内置触摸板是 I2C HID 设备，厂商 `_HID` 为 `MCHP1930`，且 `_STA`
  在 macOS 下返回 0。尝试用改过的 DSDT（`_CID = PNP0C50`、`_STA = 0x0F`）+ VoodooI2C 2.8/2.9 会导致内核
  panic（`va_map_delete` in `com.alexandred.VoodooI2C`），因此触摸板保持禁用。请用 USB 鼠标。

### 致谢

- [NullMoth](https://github.com/nullmoth) — 1401 与 NVIDIA 驱动（没有它就没有这一切）
- [OpenIntelWireless](https://github.com/OpenIntelWireless) — itlwm / HeliPort
- [@naevui](https://github.com/naevui) — 两条 Alder Lake 4P+4E 拓扑补丁来自他的 OpenCore 配置
- Acidanthera — OpenCore、Lilu、VirtualSMC、WhateverGreen、AppleALC、BrightnessKeys、USBToolBox、VoodooPS2

---

## Disclaimer / 免责声明

**English.** This EFI is provided **"as is"**, without warranty of any kind, express or implied, including but
not limited to the warranties of merchantability, fitness for a particular purpose and noninfringement. In no
event shall the authors or copyright holders be liable for any claim, damages or other liability, whether in an
action of contract, tort or otherwise, arising from, out of or in connection with this EFI or the use or other
dealings in this EFI.

macOS on non-Apple hardware ("Hackintosh") is not supported by Apple. Installing macOS on non-Apple hardware may
violate the macOS Software License Agreement (EULA) in some jurisdictions. Nothing here grants any right to
Apple software or trademarks; you install macOS and use it entirely at your own risk and under your own
responsibility. This EFI comes with no support, no update guarantee and no fitness for a particular machine.
Verify every setting against your own hardware before flashing or booting it.

**中文.** 本 EFI 按**"原样"**提供，不附带任何明示或暗示的保证，包括但不限于对适销性、特定用途适用性和
不侵权的保证。在任何情况下，作者或版权持有人均不对任何索赔、损害或其他责任负责，无论其是否因合同、侵权
或其他行为引起，也无论其是否与本 EFI 或对本 EFI 的使用或其他交易有关。

在非 Apple 硬件上安装 macOS（"黑苹果"）不受 Apple 支持，在某些司法管辖区可能违反 macOS 软件许可协议（EULA）。
本文件不授予任何 Apple 软件或商标的权利；安装和使用 macOS 的全部风险与责任由你自己承担。本 EFI 不提供任何
支持、不保证更新、不保证适用于任何特定机器。刷入或启动前，请逐项核对你自己的硬件配置。
