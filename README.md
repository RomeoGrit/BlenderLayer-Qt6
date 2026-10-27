# Blender Layer (Krita 6 / Qt6)

Live 3D view from Blender streamed into Krita as a normal layer: trace over it,
change blending modes, apply layer styles.

See the [Krita Artists thread](https://krita-artists.org/t/plugin-blender-layer-live-3d-view-in-krita/63394) for more info.

This is the original **Blender Layer** plugin, updated to run on **Krita 6 (Qt 6 / PyQt6 / Python 3.13)**.
Krita 6 refuses to load plugins that import the wrong Qt binding (`PyQt5`), which made the
original plugin fail with:

```
ModuleNotFoundError: This version of Krita is not compatible with PyQt5!
Could not import blender_layer
```

## Compatibility

| Krita | Status |
| --- | --- |
| Krita 6.x (Qt 6 / PyQt6) | ✅ supported by this repository |
| Krita 5.x (Qt 5 / PyQt5) | ❌ this version targets PyQt6 – use [Yuntokon/BlenderLayer](https://github.com/Yuntokon/BlenderLayer) |

Tested against Krita 6.0.4 (PyQt 6.8.1, Qt 6.8.0, Python 3.13).

## What was changed for Qt 6

Based on the Qt6 port by [shion64/BlenderLayer-Qt6-](https://github.com/shion64/BlenderLayer-Qt6-),
plus the following completions:

* **`blenderLayerServer.py` migrated to PyQt6** – it still imported `PyQt5`, so the plugin
  could not even be imported (the import error above came from this file).
* **Explicit PyQt6 widget imports** – Krita 6's `from krita import *` no longer re-exports
  Qt widget classes (`QComboBox`, `QProgressBar`, `QDialog`, `QFileDialog`, …), only the
  `QtCore` / `QtGui` / `QtWidgets` module objects. Every class the plugin uses is now imported explicitly.
* **Scoped enumerations** – `Qt.ItemDataRole.ToolTipRole`, `QFrame.Shape.NoFrame`,
  `QDialog.DialogCode.Accepted`, `QImage.Format.Format_RGBA8888`, `QIcon.Mode.Normal`,
  `QSizePolicy.Policy.Expanding`, … (PyQt6 removed the flat `Qt.ToolTipRole` style).
* **Removed APIs** – `dialog.exec_()` → `dialog.exec()`, `QMenu.exec_` → `QMenu.exec`,
  `QByteArray.fromRawData(...)` → `QByteArray(...)` (the pixel data path used when a frame
  is pushed from Blender into a Krita layer), `QImage(data, w, h, format)` now passes `bytesPerLine`.
* **Misc fixes** – `except e:` → `except Exception as e:`, `os.path.os.path.isfile` →
  `os.path.isfile`, raw strings for Windows paths (removes `SyntaxWarning: invalid escape sequence`).
* **Blender client** – NumPy 2.0 compatibility (`np.array(..., copy=False)` removed in NumPy 2)
  and a guard for `bpy.ops.object.mode_set` with no active object.
* **Blender 5.x pixel layout fix** – `GPUTexture.read()` now yields an `(h, w, 4)` array instead of a
  flat buffer, so the old `.ravel(order='F').reshape(h, w, 4)` scrambled the frame: the image arrived
  as gray, 4×4 repeated tiles. The buffer is now reshaped directly (works for both flat and shaped
  layouts). Verified against Blender 5.2.3 (NumPy 2.3.4): output is byte-identical to the reference
  at scale 1/2/4.

### Verification

* All 209 Qt attribute chains used by the plugin resolve against PyQt6 6.8.1 / Qt 6.8.0.
* 26 runtime smoke checks (docker construction, settings dialog, animation dialog,
  context menu, navigation widget painting/input, frame pixel conversion) executed with
  Krita's own PyQt6 build and a stubbed Krita API – all passing.

## Installation

1. Download this repository (`Code → Download ZIP`) and extract it.
2. Copy **`blender_layer/`** and **`blender_layer.desktop`** into the Krita plugin folder:
   * **Windows:** `%APPDATA%\krita\pykrita`
   * **Linux:** `~/.local/share/krita/pykrita`
   * **macOS:** `~/Library/Application Support/krita/pykrita`
3. In Krita: *Settings → Configure Krita → Python Plugin Manager* → enable **Blender Layer**.
4. Restart Krita.

## Getting Started

* Install Blender.
* Make sure the 'Blender Layer' add-on is installed and enabled in Krita.
* Open a document.
* Connect to Blender:
  * **Drag and drop** a .blend file into Krita. This will start Blender, connect and directly
    open the .blend file. (You might be asked to select the path to the Blender executable.
    Under Windows and Blender 3.4 for example: `C:\Program Files\Blender Foundation\Blender 3.4\blender.exe`)
  * Or use the **'Start Blender'** button in the 'Blender Layer' docker.
  * (Advanced) Or manually install the **companion add-on** for Blender: *Preferences → Add-Ons → Install…*
    and select **`blenderLayerClient.py`** from the `blender_layer` folder of the add-on
    (under Windows installed Krita add-ons are in `%APPDATA%\krita\pykrita`). In Krita, start
    the server from the docker, then in Blender choose *View → Connect to Krita* from the header
    of a 3D View, make sure host and port match the Krita settings and press OK.

## Usage

### Navigation

Using the navigation widget in the docker:

* **Drag** to rotate
* **Shift + Drag** to pan
* **Ctrl + Drag** vertically to zoom

You can also change the view from outside the docker with **Alt + Middle Mouse Button**.

#### Assistants

In order to create painting assistants matching Blender's perspective use the **'Create Assistant Set'**
button in the docker. You will be asked to save an xml file. Make sure the assistant tool is selected,
then open the Tool Settings and press the **'Load Assistant Set'** button (folder icon) and select the
file you have just created. The colored axis can be disabled in settings.

#### Library

The docker allows you to append a selection of objects to the current scene. By default this includes
the posable mannequin models 'Body-chan' and 'Body-kun' (CC-0 license, created by
[vinchau](https://blendswap.com/blend/23521)). You can configure additional objects from the settings
menu by providing a .blend file and the path to objects within the file, e.g. `Object/Cube`
(see Blender's [append](https://docs.blender.org/manual/en/latest/files/linked_libraries/link_append.html) function for reference).

Blender Layer also supports Blender's asset based
[Pose Library](https://docs.blender.org/manual/en/latest/animation/armatures/posing/editing/pose_library.html).
If your Blender file contains pose assets, these show up in the 'Library' section of the docker and can
be applied directly from within Krita.

## Credits

* Original plugin: [Yuntokon/BlenderLayer](https://github.com/Yuntokon/BlenderLayer)
* Qt6 port base: [shion64/BlenderLayer-Qt6-](https://github.com/shion64/BlenderLayer-Qt6-)
* Body-chan / Body-kun models: CC-0 by [vinchau](https://blendswap.com/blend/23521)

## License

GPL-3.0 (see [LICENSE](LICENSE)).
