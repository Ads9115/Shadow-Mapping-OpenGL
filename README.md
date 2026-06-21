# Shadow Mapping

This project is configured for the bundled x64 GLFW library in `../Libraries/lib`.

## Layout

- `src/`: C++ implementation files
- `include/ShadowMapping/`: project headers
- `assets/shaders/`: GLSL shader assets copied beside the executable
- `vendor/glad/`: generated GLAD source used by the CMake target

## Build

```powershell
cmake --preset windows-x64
cmake --build --preset windows-x64-debug
```

Use the preset above, or configure a fresh build directory with `-A x64`.

The bundled `glfw3.lib` was built against the debug MSVC runtime, so the Debug preset is the clean local build. Release builds work, but MSVC will warn until `Libraries/lib/glfw3.lib` is replaced with a release-built GLFW library.
