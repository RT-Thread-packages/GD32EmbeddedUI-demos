# GD32EmbeddedUI-demos

[中文说明](README_zh.md)

An editable GD32EmbeddedUI starter project with generated LVGL sources and an
RT-Thread SCons integration script. It can be used with a target project that
already provides a compatible LVGL port, display driver and, when needed, input
driver. The UI sources are independent of a particular development board.

The bundled project was exported with **GD32EmbeddedUI 0.3.0-beta** for
**LVGL 8.3.11**. Its default canvas is **720 × 720**, with **16-bit RGB565** color.
These are the supplied design settings, not a fixed hardware requirement.

The starter contains one empty screen and an empty `custom_init()`. Add widgets
in GD32EmbeddedUI and generate code to create a visible, interactive interface.

## Open and edit

1. Clone or download this repository.
2. Open [`GD32Embeddedui.gdlvgl`](GD32Embeddedui.gdlvgl) in GD32EmbeddedUI.
3. Add screens, widgets, events and resources, then save and generate code into
   this project directory.
4. Update the source list in your target project and rebuild it.

Keep `GD32Embeddedui.gdlvgl`, `assets/`, `generated/` and `custom/` together so the
project remains editable. Put application logic in `custom/`; generated files
may be overwritten when code is regenerated. Preserve `port/` and `SConscript`.

## Change the screen size

**Edit `width` and `height` directly in the `display` object of
[`GD32Embeddedui.gdlvgl`](GD32Embeddedui.gdlvgl).** The file is JSON. For example,
to use an 800 × 480 canvas, change the existing object to:

```json
"display": {
  "width": 800,
  "height": 480,
  "colorDepth": 16
}
```

Save and close the project in GD32EmbeddedUI before editing the file externally.
After saving the file, reopen it in GD32EmbeddedUI and **generate code again**.
Review widget positions and sizes for the new canvas; changing the dimensions
does not automatically adapt an existing layout.

The target project's LVGL display driver resolution, drawing buffers and any
resolution checks must match the new size. Regeneration updates the UI sources
and exported configuration; it does not configure the physical display driver.
Keep the color depth consistent with the target's `LV_COLOR_DEPTH` and pixel
format. Do not change only `generated/config/ui_config.h`, as regeneration will
replace it.

## Integrate with an existing LVGL project

The bundled output targets LVGL 8.3.11. Other LVGL versions require compatibility
checks or regeneration with the matching designer export version.

1. Compile all `.c` files recursively under `generated/` and `custom/`.
2. Add the repository root, `port/`, and the directories under `generated/` and
   `custom/` to the compiler's include paths. Firmware must use `port/aic_ui.h`.
   Exclude `simulator/` from the firmware build.
3. Include the exported configuration from the target's `lv_conf.h`, within its
   active configuration section. Place the patch after the base options so its
   overrides apply to both LVGL itself and the UI sources:

   ```c
   #include "generated/config/ui_config.h"
   #include "generated/config/lv_conf_patch.h"
   ```

   Configure `LV_COLOR_DEPTH` to match `UI_COLOR_DEPTH` and enable the widgets,
   fonts and features used by the generated interface. Both exported headers are
   currently minimal; review them after regenerating a more complex design.
4. Include `ui_init.h` and call `ui_init()` once, after LVGL and the display/input
   drivers are initialized, in the target's LVGL thread or protected LVGL context:

   ```c
   #include "ui_init.h"

   /* In the existing LVGL startup path, after driver registration: */
   ui_init();
   ```

`ui_init()` loads the initial screen and calls `custom_init()`. Continue using
the target's existing LVGL tick and `lv_timer_handler()` loop. The package does
not initialize hardware or create an additional LVGL thread.

### RT-Thread / SCons

Place the repository in the target project's package directory and ensure the
parent build script loads its `SConscript`. For example, from a parent
`packages/SConscript` that maintains an `objs` list:

```python
objs += SConscript('GD32EmbeddedUI-demos/SConscript')
```

The supplied script builds the UI when `PKG_USING_LVGL` is enabled and discovers
source files and include directories recursively on each invocation. It uses
the target project's LVGL library and port. This repository can be included
locally; it does not require registration in the RT-Thread online package index.

After adding or removing generated source files, refresh your IDE's source list.
For RT-Thread projects supporting SCons project generation, run
`scons --target=mdk5` or `scons --target=eclipse` from the target project directory.

## Repository layout

| Path | Purpose |
| --- | --- |
| `GD32Embeddedui.gdlvgl` | Editable design, display size and resource settings |
| `assets/` | Original image, font and video resources |
| `generated/` | Generated screens, events, themes, translations and C resources |
| `custom/` | Application logic and `custom_init()` |
| `port/aic_ui.h` | Firmware header with simulator mode disabled |
| `SConscript` | RT-Thread source and include-path collection |
| `simulator/`, `CMakeLists.txt` | Windows simulator project for desktop preview |

Export firmware images and fonts as C arrays/static fonts under `generated/`.
Files in `assets/` are not automatically installed into a target filesystem.
Filesystem-backed resources, FreeType and video playback require additional
support in the target project.

The simulator requires a Windows toolchain and the designer's LVGL/dependencies.
Third-party implementations are not bundled here; see `simulator/CMakeLists.txt`
and the README files under `simulator/third_party/` for dependency locations.

## License notices

Exported files retain their original GigaDevice copyright and license notices.
The firmware compatibility header retains its RT-Thread Apache-2.0 notice.
Observe the licenses of LVGL, fonts and any other resources you add.
