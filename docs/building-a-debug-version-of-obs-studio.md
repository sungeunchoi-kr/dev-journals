# Building a Debug Version of OBS Studio
2022-05-07

## Environment Setup (Windows)
1. Visual Studio Community 2019 with:
    - "Desktop development in C++ workflow".
    - Windows 11 SDK (10.0.22000.0)
    - "C++ ATL vXXX build tools with Spectre Mitigations" (Visual Studio Installer > Individual components tab)
    - NOTE: you may select these options during installation of VS 2019 or you can modify
    an existing installation to add the above packages/workflows by using the Visual Studio Installer.
2. Cmake

source: https://github.com/obsproject/obs-studio/wiki/build-instructions-for-windows

Assume the current working directory is `C:\projects`.

Get the source:
```
git clone --recursive https://github.com/obsproject/obs-studio.git
```

Run the install-dependencies script:
```ps
powershell -ExecutionPolicy Bypass -File .\obs-studio\ci\windows\01_install_dependencies.ps1
```
  - `C:\projects\obs-build-dependencies` will be created.

Copy the contents of `obs-build-dependencies\windows-deps-2022-03-16-x64\` into `obs-studio\deps\`.

Open cmake-gui and set the following:
- Where is the source code: C:\projects\obs-studio
- Where to build the binaries: C:\projects\obs-studio\build

Refer to https://github.com/obsproject/obs-studio/wiki/Building-OBS-Studio#cmake for cmake flags
- CMAKE_BUILD_TYPE=Debug
- CMAKE_PREFIX_PATH='C:\projects\obs-studio\deps' 
- ENABLE_SCRIPTING='OFF'
- ENABLE_BROWSER_SOURCE='OFF'
- ENABLE_BROWSER_SHARED_TEXTURE='OFF'
- ENABLE_VLC='OFF'

The only _required_ flag is `CMAKE_PREFIX_PATH`.

Click the "Configure" button, then select the appropriate Visual Studio version on the system.

After configuring is done, click the "Generate" button. I have added the other flags because they are not needed for my use case.

Visual Studio project solution will be generated in `obs-studio\build\obs-studio.sln`. Open the project and build the solution.

If you get the `atlbase.h: No such file or directory` error, install the "C++ ATL vXXX build tools with Spectre Mitigations" component in Visual Studio Installer > Individual components tab.

## Building the Virtual Camera (Windows)
Resources:
 - https://github.com/obsproject/obs-studio/issues/4286
 - https://github.com/obsproject/obs-studio/wiki/build-instructions-for-windows#5-install-the-virtual-camera

This command:
```
PS> cmake -G "Visual Studio 16 2019" -A x64 -S . -B build64 \
    -D CMAKE_PREFIX_PATH=C:/projects/obs/obs-studio/deps \
    -D VIRTUALCAM_GUID=e61ba6f1-ac3a-47b6-aaee-b537088061e4 \
    -D CMAKE_CONFIGURATION_TYPES=Release

PS> cmake --build .\build --config Release
```
does generate the `obs-virtualcam-module64.dll` and other installation scripts in `rundir\Release\data\obs-plugins\win-dshow`; however, running the installation script did not result in the "Start virutal camera" button being shown on the OBS UI.

To fix this, referenced https://stackoverflow.com/a/71047142 -- the `vcam_installed(bool b64)` in `dshow-plugin.cpp` takes in an argument that when set to `true`, searches for the 64-bit version of the driver. Set that to true and the virtual camera is recognized.

NOTE: the `obs-plugins\win-dshow\virtualcam-install.bat` virtual camera installation batch script had to be modified to not install the 32-bit version of the dll which does not exist.

## Building the Virtual Camera (Mac)
The process is largely the same as Windows.

The virtual camera is output as a plugin at `build/plugins/mac-virtualcam/Release` after the release build (`cmake --build .\build --config Release`).

Copying the plugin files there into `/Library/CoreMediaIO/Plug-Ins/DAL` and restarting the computer installs the virtual camera.
