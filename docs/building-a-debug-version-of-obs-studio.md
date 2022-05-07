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
