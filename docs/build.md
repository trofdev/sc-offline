# Building and checks

This repository publishes no releases. You build `dinput8.dll` and `sc-offline.exe` yourself; the
step-by-step is in the [README](../README.md#build-it-yourself). This page has the details.

## Build on Windows

1. Install **Visual Studio 2026** with the **Desktop development with C++** workload. The projects use the `v145` toolset. On VS 2022, retarget them to `v143` first (Project → Retarget).
2. Open `sc-offline.slnx`, select **Release | x64**, and build. Or from the command line:

   ```powershell
   msbuild sc-offline.slnx /p:Configuration=Release /p:Platform=x64 /m
   ```

3. The build produces `dinput8.dll` (the mod, from `src/`) and `sc-offline.exe` (the launcher, from `launcher/`) under `x64\Release\`. To play, put both in a folder together with `data/` and `launcher/sc-offline.ini`.

Release builds link the C runtime statically, so they don't need the Visual C++ redistributable installed.

## Check on macOS or Linux

```bash
tools/check.sh
```

This runs in a few seconds. It parses every `src/*.cpp` and `launcher/*.cpp` file with clang against mingw-w64's Windows headers. It also screens `src/` for MSVC error C2712 (`__try` in a function that owns an object needing unwinding, such as a `std::string`). It is not a build: only MSVC's build is. Known clang-only diagnostics are listed in `tools/check-baseline.txt`, and only new ones fail the check. You need clang and mingw-w64 (`brew install llvm mingw-w64` on macOS).

## CI

[`.github/workflows/build.yml`](../.github/workflows/build.yml) runs on pushes to `main` and on pull requests. It only verifies: `check` runs `tools/check.sh` on Linux, then `build` compiles Release x64 on Windows. It uploads nothing and publishes no release. Every action is pinned to a commit SHA and the workflow has read-only permissions.

## Version

`SCO_VERSION` in `src/version.h` is the one version for the DLL and the launcher. It shows in `mod.log`, the menu title and the launcher window.
