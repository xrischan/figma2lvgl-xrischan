# figma2lvgl

**Convert Figma UI designs into native LVGL 9 C code.**

`figma2lvgl` converts Figma XML exports into structured C source and header files that can be integrated directly into embedded LVGL projects.

The aim is to make Figma a practical UI design environment for embedded development: design your interface visually, export the XML, generate the C code, and integrate it into your firmware.

- Converts Figma XML into LVGL 9 C code.
- Generates individual screens from Figma frames.
- Preserves the original Figma frame and element names.
- Retains the hierarchy of UI elements for structured access from firmware.
- Converts supported Figma styling into LVGL style definitions.
- Generates native C code, not a runtime XML interpreter.
- Supports ESP-IDF, Zephyr and other LVGL-based environments.
- Runs on Windows, Linux and macOS.

---

## Installation

### Install the latest version from this repository

```bash
pip install --upgrade git+https://github.com/xrischan/figma2lvgl-xrischan.git
```

### Requirements

- Python 3.9 or later.
- Figma.
- The FigML — Figma XML Exporter plugin.
- LVGL 9 in your target firmware project.

The LVGL image conversion utility, `LVGLImage.py`, is downloaded and cached automatically when required. Alternatively, you can supply your own copy.

---

## Workflow

The basic workflow is:

1. Design your interface in Figma.
2. Organise your interface into frames representing individual screens.
3. Export the design using the FigML XML exporter.
4. Run `figma2lvgl` against the exported XML.
5. Integrate the generated C files into your LVGL project.
6. Implement your application logic using the generated UI objects and accessors.

### Exporting from Figma

Use the FigML — Figma XML Exporter plugin to export your design.

The exported XML is the input to `figma2lvgl`.

Frames are used to organise the interface into individual screens. Their original names are retained, allowing the generated screen files and structures to remain associated with the corresponding Figma frames.

You do not need to rename your frames to fit a special naming convention.

---

## Usage

The basic command is:

```bash
figma2lvgl -x layout.xml
```

### Command-line options

| Argument | Description | Default |
|---|---|---|
| `-x`, `--xml` | Input Figma XML file | Required |
| `-i`, `--images` | Directory containing exported PNG images | XML directory |
| `-d`, `--dest` | Output directory | XML directory |
| `-y`, `--yes` | Skip interactive prompts | Off |
| `--lvgl-tool PATH` | Specify the LVGL image converter | Automatic |
| `-f`, `--color-format` | Image pixel format: `RGB565`, `RGB888`, `ARGB8888`, `L8` | `RGB565` |
| `-v`, `--verbose` | Enable detailed logging | Off |
| `--verify-compile` | Check generated C syntax using GCC | Off |

`--patch-esp-includes` is retained for backwards compatibility but is deprecated. ESP-IDF image include handling is now applied automatically.

### Examples

Basic conversion:

```bash
figma2lvgl -x home.xml
```

Specify image and output directories:

```bash
figma2lvgl -x home.xml -i images -d generated
```

Non-interactive conversion:

```bash
figma2lvgl -x home.xml --yes
```

Use a specific image converter:

```bash
figma2lvgl -x home.xml --lvgl-tool ./tools/LVGLImage.py
```

Generate images in ARGB8888 format:

```bash
figma2lvgl -x home.xml -f ARGB8888
```

---

## Generated output

The generator produces a self-contained `ui_src` directory.

```text
ui_src/
│
├── src/
│   └── Generated screen C files
│
├── include/
│   └── Generated screen header files
│
├── priv_src/
│   ├── Converted image C files
│   └── Shared style implementation
│
└── priv_include/
    ├── assets.h
    ├── ui_config.h
    ├── ui_defs.h
    └── ui_style.h
```

Each Figma screen is represented by generated C and header files.

The generated UI is native LVGL code. XML parsing and layout interpretation take place during generation rather than at runtime.

Copy `ui_src` into your firmware project and include its source files in your build.

Ensure that `ui_src/priv_include` is available on the include path for all generated source files.

For a bare-metal Makefile, for example:

```makefile
CFLAGS += -Iui_src/priv_include
```

ESP-IDF and Zephyr projects can handle the include configuration through their respective build systems.

---

## Using the generated UI

Include the generated header for the screen you want to use:

```c
#include "ui_home.h"
```

Initialise and load the screen:

```c
ui_home_init();
ui_home_load();
```

The generated interface provides access to UI elements through the generated screen structure and associated accessor functions.

For example, update a label or bar at runtime:

```c
ui_home_time_set_text("16:30");

ui_home_battery_bar_set_value(85, 300);
```

The values and content initially defined in Figma provide the design-time defaults. Your firmware can update these objects as the application runs.

---

## UI elements and styling

The generator converts supported Figma elements into their corresponding LVGL objects.

| UI element | LVGL object |
|---|---|
| Text | `lv_label` |
| Button | `lv_button` |
| Image | `lv_image` |
| Bar | `lv_bar` |
| Slider | `lv_slider` |
| Panel / Container | `lv_obj` |

Figma styling is converted into LVGL style calls and applied during screen initialisation.

Supported styling includes:

| Figma property | LVGL styling |
|---|---|
| Fill colour | Background colour |
| Fill opacity | Background opacity |
| Text colour | Text colour |
| Font size | Text font |
| Corner radius | Object radius |
| Stroke colour | Border colour |
| Stroke weight | Border width |
| Opacity | Object opacity |

The generated code uses LVGL's native styling system.

### Fonts

Figma font sizes are mapped to LVGL Montserrat fonts where supported.

Currently supported sizes:

`10, 12, 14, 16, 18, 20, 22, 24`

Enable the font sizes used by your design in `lv_conf.h`.

For example:

```c
#define LV_FONT_MONTSERRAT_12 1
#define LV_FONT_MONTSERRAT_14 1
```

---

## Event handling

Generated interactive elements can expose named event callbacks in their corresponding header files.

Implement the callbacks in your application code to connect the UI to your firmware logic.

For example:

```c
void ui_home_on_btn_ok_clicked(lv_event_t *e)
{
    // Application-specific button handling
}
```

The generated initialisation code registers the associated callbacks with LVGL.

---

## Images

Image assets are supplied separately from the XML export.

The `-i` option specifies the directory containing the exported PNG files.

The generator converts supported images into C source files using LVGL's image conversion utility.

Available output formats:

- RGB565
- RGB888
- ARGB8888
- L8

The generated image source files are included in `ui_src/priv_src`.

---

## Development

The project is written in Python and uses a modular structure.

```text
figma2lvgl/
├── core/
│   ├── figma_parser.py
│   └── generator.py
└── ...
```

The parser handles the exported Figma XML and extracts the UI structure and properties. The generator converts the parsed representation into LVGL C source and header files.

Tests are located in the `tests` directory.

---

## Project status

This is an actively developed project.

The intention is to make it easier to design, generate and maintain embedded LVGL interfaces using Figma, while keeping the final firmware implementation in native C.

Contributions, bug reports and suggestions are welcome.

---

## Licence

MIT License. See [LICENSE](LICENSE).
