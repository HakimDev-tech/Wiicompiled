# Experimental desktop OpenGL backend

This is an opt-in compatibility experiment for older Windows GPUs. The default renderer and
backend order are unchanged unless `AURORA_EXPERIMENTAL_DESKTOP_GL_BACKEND=ON` is supplied.

## Why this exists

Aurora already routes GX rendering through WebGPU/Dawn. The Intel HD Graphics 4400 exposes
OpenGL through its legacy driver, while Vulkan is not a reliable native option on this GPU.
This change does not replace GX or implement a second renderer: it asks Dawn to compile its
desktop OpenGL backend and adds that backend to Aurora's automatic fallback order.

## Configure

Use the repository's supported 64-bit Clang/LLVM-MinGW Windows toolchain and a source build
of Dawn. From the repository root, configure the normal runtime build with:

```powershell
cmake -S runtime -B build-gl -G Ninja -DCMAKE_BUILD_TYPE=Release -DAURORA_EXPERIMENTAL_DESKTOP_GL_BACKEND=ON -DAURORA_DAWN_PROVIDER=vendor
```

Keep the project's normal required build options/toolchain file if your local setup needs them.
Do not set `MKW_NATIVE_PREBUILT_DIR`: that path skips compiling Aurora from source, so it cannot
pick up this experiment. Dawn source compilation may take substantial time and disk space.

## What this changes

- The option defaults to `OFF`; normal builds retain their existing backend list.
- When enabled, the provider resolves to `vendor` because a prebuilt Dawn binary cannot gain
  OpenGL support from CMake cache variables.
- Dawn's desktop OpenGL backend is enabled; OpenGL ES remains disabled.
- If Dawn exposes `DAWN_ENABLE_BACKEND_DESKTOP_GL`, Aurora tries desktop OpenGL before its
  existing D3D12/Vulkan fallback candidates.

## Validation status and test checklist

The change is committed to the feature branch, but has **not** been compiled or run in this
environment. Enabling a backend at configure time does not guarantee Dawn's pinned revision,
SDL surface integration, shaders, or the game's GX workload work on this driver.

1. Configure and compile with the option enabled; confirm CMake reports the vendor Dawn build.
2. Check the startup log for the selected adapter/backend and verify it reports OpenGL.
3. Launch a title screen and a race; inspect textures, shaders, depth, EFB copies, and presentation.
4. Compare stability and frame pacing against the current backend.
5. If Dawn's pinned revision does not compile desktop GL, revert this experiment rather than
   silently treating the backend as supported.

This is an isolated experiment, not a claim of confirmed HD 4400 compatibility.
