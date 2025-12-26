# Floating Play Window: Dynamic Resizing Guide

## Problem
When resizing the floating play window via code using `window.set_size()`, the window bounds do not change. Only the viewport inside scales, creating a "screen-in-screen" effect. Physical dragging works fine, but code-based resizing is ineffective.

## Solution
Use the `force_resize()` method on the `WindowWrapper` instead of `set_size()` on the Window directly.

## Implementation

### Get Reference to the Floating Window
In your game code running in the floating play window:

```gdscript
# The floating window is the root window
var floating_window = get_window()
```

### Resize Properly
```gdscript
# DO NOT USE THIS (doesn't work for floating windows):
# get_window().set_size(Vector2i(1280, 720))

# USE THIS INSTEAD:
# The floating window wrapper has force_resize() method
if floating_window and floating_window.has_method("force_resize"):
    floating_window.force_resize(Vector2i(1280, 720))
```

### Full Example
```gdscript
extends Node

func _ready():
    # Resize floating window on startup
    var floating_window = get_window()
    if floating_window.has_method("force_resize"):
        floating_window.force_resize(Vector2i(1920, 1080))

func _input(event):
    if event is InputEventKey and event.pressed:
        match event.keycode:
            KEY_P:  # Press P to resize to 800x600
                get_window().force_resize(Vector2i(800, 600))
            KEY_F:  # Press F to resize to fullscreen-like
                get_window().force_resize(Vector2i(1920, 1080))
```

## Why This Works

The `force_resize()` method:
1. Clears size constraints (min_size, max_size)
2. Disables automatic sizing from child controls
3. Directly calls DisplayServer to update the native window size
4. Bypasses Godot's Window class constraints that were preventing resizing

## Technical Details

- **Location:** `editor/gui/window_wrapper.h` and `editor/gui/window_wrapper.cpp`
- **Method signature:** `void force_resize(const Size2i &p_size)`
- **Requirements:** Window must be available and enabled (floating)
- **Safety:** Method safely checks if window exists before operating

## Notes
- Only works on the floating play window (sub-windows created during play)
- The main editor window cannot be resized this way
- Physical window dragging still works as expected
- This is a temporary workaround; a permanent solution would involve rearchitecting how floating windows handle size constraints
