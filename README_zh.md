# GD32EmbeddedUI-demos

[English](README.md)

本仓库提供可在 GD32EmbeddedUI 中继续编辑的基础 UI 工程、生成的 LVGL 源码，以及
RT-Thread SCons 接入脚本。适用于已移植兼容版本 LVGL、显示驱动及所需输入驱动的工程，
UI 源码不依赖特定开发板。

当前工程由 **GD32EmbeddedUI 0.3.0-beta** 导出，使用 **LVGL 8.3.11**，默认画布为
**720 × 720、16 位 RGB565**。这些是附带设计的默认设置，可以按目标屏幕调整。

仓库附带一个空白页面，`custom_init()` 也为空。请在设计器中添加控件并生成代码，
即可在完成接入后显示自己设计的界面。

## 打开和编辑工程

1. 克隆或下载本仓库。
2. 在 GD32EmbeddedUI 中打开 [`GD32Embeddedui.gdlvgl`](GD32Embeddedui.gdlvgl)。
3. 添加页面、控件、事件和资源，保存并生成代码到当前工程目录。
4. 更新目标工程的源文件列表，重新构建目标工程。

请将 `GD32Embeddedui.gdlvgl`、`assets/`、`generated/` 和 `custom/` 一起保留，
以便后续继续编辑。业务逻辑放在 `custom/` 中；重新生成代码时，`generated/` 中的文件
可能被覆盖。请保留接入层的 `port/` 和 `SConscript`。

## 修改屏幕尺寸

**直接修改 [`GD32Embeddedui.gdlvgl`](GD32Embeddedui.gdlvgl) 中 `display` 对象的
`width` 和 `height`。** 这是一个 JSON 文件。例如，改为 800 × 480 时，将原有对象改为：

```json
"display": {
  "width": 800,
  "height": 480,
  "colorDepth": 16
}
```

使用文本编辑器修改前，先在 GD32EmbeddedUI 中保存并关闭该工程。修改文件并保存后，
重新在 GD32EmbeddedUI 中打开工程，**再次生成代码**。已有页面的控件位置和大小需要
按新画布检查、调整，修改尺寸不会自动完成所有布局适配。

目标工程中的 LVGL 显示驱动分辨率、绘图缓冲区和分辨率检查也应与新尺寸匹配。
重新生成代码会更新 UI 源码和导出配置，不会自动配置物理屏幕驱动。
色深应与目标工程的 `LV_COLOR_DEPTH` 和像素格式保持一致。
不要仅修改 `generated/config/ui_config.h`，否则下次生成代码时会被覆盖。

## 接入已移植 LVGL 的工程

当前导出代码面向 LVGL 8.3.11。使用其他版本时，需要检查兼容性，或使用设计器中匹配的
LVGL 导出版本重新生成代码。

1. 将 `generated/` 和 `custom/` 下的全部 `.c` 文件递归加入构建。
2. 将仓库根目录、`port/`、`generated/` 和 `custom/` 下各级目录加入头文件搜索路径。
   固件使用 `port/aic_ui.h`，不将 `simulator/` 加入固件构建。
3. 在目标工程 `lv_conf.h` 的有效配置区域中引入导出配置，补丁放在基础选项之后，
   使其同时作用于 LVGL 库和 UI 源码：

   ```c
   #include "generated/config/ui_config.h"
   #include "generated/config/lv_conf_patch.h"
   ```

   将 `LV_COLOR_DEPTH` 配置为与 `UI_COLOR_DEPTH` 一致，并启用生成界面需要的控件、
   字体和功能。当前两个导出头文件的配置较少，生成复杂界面后应重新检查。
4. 引入 `ui_init.h`。在 LVGL 和显示、输入驱动初始化完成后，在已有 LVGL 线程或
   受保护的 LVGL 调用上下文中调用一次 `ui_init()`：

   ```c
   #include "ui_init.h"

   /* 在已有 LVGL 启动流程中，完成驱动注册后调用： */
   ui_init();
   ```

`ui_init()` 会加载初始页面，并调用 `custom_init()`。继续使用目标工程已有的 LVGL
节拍和 `lv_timer_handler()` 循环。本软件包不初始化硬件，也不额外创建 LVGL 线程。

### RT-Thread / SCons

将仓库放入目标工程的软件包目录，并确保上层构建脚本加载本仓库的 `SConscript`。
例如，上层 `packages/SConscript` 使用 `objs` 汇总构建结果时可加入：

```python
objs += SConscript('GD32EmbeddedUI-demos/SConscript')
```

附带脚本在启用 `PKG_USING_LVGL` 时参与构建，每次运行都会递归收集源码和头文件路径，
复用目标工程的 LVGL 库与移植层。本仓库可直接作为本地软件包接入，不要求先注册到
RT-Thread 在线软件包索引。

新增或删除生成的源文件后，需刷新 IDE 的源文件列表。对于支持 SCons 工程生成的
RT-Thread 工程，在目标工程目录执行 `scons --target=mdk5` 或
`scons --target=eclipse`，更新对应 IDE 工程。

## 目录结构

| 路径 | 用途 |
| --- | --- |
| `GD32Embeddedui.gdlvgl` | 可编辑设计、屏幕尺寸和资源配置 |
| `assets/` | 原始图片、字体和视频资源 |
| `generated/` | 生成的页面、事件、主题、多语言和 C 资源 |
| `custom/` | 业务逻辑与 `custom_init()` |
| `port/aic_ui.h` | 固件平台头文件，关闭模拟器模式 |
| `SConscript` | RT-Thread 源码与头文件路径收集 |
| `simulator/`、`CMakeLists.txt` | 用于桌面预览的 Windows 模拟器工程 |

固件使用的图片和字体请导出为 `generated/` 下的 C 数组或静态字体。
`assets/` 中的原始文件不会自动部署到目标文件系统；如需文件资源、FreeType 或视频播放，
应在目标工程中另行完成对应支持。

模拟器需要 Windows 工具链及设计器提供的 LVGL 等依赖。本仓库没有附带第三方库实现，
依赖目录配置见 `simulator/CMakeLists.txt` 和 `simulator/third_party/` 下的 README。

## 许可说明

导出文件保留原有的 GigaDevice 版权和许可声明；固件兼容头文件保留 RT-Thread 的
Apache-2.0 声明。新增 LVGL、字体及其他资源时，请遵守其各自的许可证。
