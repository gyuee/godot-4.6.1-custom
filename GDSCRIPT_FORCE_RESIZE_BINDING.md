# GDScript Binding for force_resize() Method

## Problem
The `force_resize()` method was added to the Godot C++ `Window` class to enable dynamic window resizing in floating play windows, but it's not accessible from GDScript. When attempting to call it from GDScript, we get:

```
Invalid call. Nonexistent function 'force_resize' in base 'Window'.
```

## Root Cause
The C++ method exists but wasn't properly bound for GDScript access using `ClassDB::bind_method()`. This binding is necessary to expose C++ methods to the GDScript runtime.

## Solution

### Step 1: Locate the Binding File
Find the `window.cpp` file in your Godot source directory:
```
godot/scene/main/window.cpp
```

### Step 2: Add the ClassDB Binding
In `window.cpp`, find the section where other Window methods are bound (typically in a function like `Window::_bind_methods()`). Add this line:

```cpp
ClassDB::bind_method(D_METHOD("force_resize", "size"), &Window::force_resize);
```

### Step 3: Verify the C++ Implementation
Make sure your `force_resize()` method exists in `window.h` and is declared as:

```cpp
void force_resize(const Size2i &p_size);
```

And implemented in `window.cpp` with the actual resizing logic.

### Step 4: Clean Rebuild
After adding the binding, perform a **clean rebuild** of Godot:

```bash
# From your Godot source directory
scons -j4 platform=windows target=editor dev_build=yes
```

The `dev_build=yes` flag is important for development/testing.

### Step 5: Test from GDScript
Once recompiled, you should be able to call it from GDScript:

```gdscript
var window = get_window()
if window.has_method("force_resize"):
    window.force_resize(Vector2i(1280, 720))
```

## Common Issues

### Issue: Still shows "Nonexistent function" after rebuild
**Solution:** You may need to:
1. Delete the `.scons_cache` folder to force a clean rebuild
2. Delete all `.o` and `.a` files in the build output directory
3. Rebuild with `-j1` instead of `-j4` to ensure no parallel build issues

### Issue: `has_method()` still returns false but `call()` works
**Solution:** This can happen if the binding was added but not in the right scope. Make sure the binding is inside the `_bind_methods()` function block.

### Issue: Compilation errors about "force_resize"
**Solution:** Verify that:
1. The method signature in the `.h` file matches the binding signature
2. The method is not marked as `private`
3. No typos in the method name

## Verification Steps

### 1. Check if binding was successful
Add this temporary debug code to your GDScript:

```gdscript
var window = get_window()
print("force_resize available: ", window.has_method("force_resize"))

# List all available window resize-related methods
for method in window.get_method_list():
    if "resize" in method.name.to_lower():
        print("Found method: ", method.name)
```

### 2. Test the resize function
```gdscript
# This should resize the window without error
window.force_resize(Vector2i(828, 1792))

# Wait a frame and verify the size changed
await get_tree().process_frame
print("New window size: ", window.size)
```

## Expected Behavior After Fix

When running the game in a floating play window and selecting a resolution from the dropdown:

1. `has_method("force_resize")` returns `true`
2. `window.force_resize(Vector2i(width, height))` executes without error
3. The floating play window actually resizes to the specified dimensions
4. The viewport inside the window resizes proportionally
5. UI elements adjust to the new window size

## Related Files (for reference)
- **Source:** `godot/scene/main/window.cpp` and `window.h`
- **Binding pattern:** Look at other method bindings in `Window::_bind_methods()` for examples
- **GDScript consumer:** `the-golden-wreath/Scripts/UI/Debug/DebugUI.gd` - `_apply_resolution()` function

## Notes for Build
- Make sure you're using the same Godot version (4.x)
- If using a custom platform (Windows), verify the Windows platform files are being compiled correctly
- The build process might take 10-30 minutes depending on your machine

## Questions?
If the binding still doesn't work after following these steps, check:
1. The exact location and syntax of other bindings in the same file for comparison
2. Whether the method visibility allows it (should be `public`)
3. Whether there are any conditional compilation flags that might exclude your code
