While creating a keyboard layout with Microsoft Keyboard Layout Creator (MSKLC) is an easy task, this repository aims at automating as much as is possible in order to reduce the possibility to make mistakes and improve the experience while developing the keyboard layout.

This document gives some insights into how the build works.

## High-level overview

The build is split into roughly two stages:

1. Creation of the keyboard layout DLL.
2. Compilation of the x86 and x64 setup packages.

MSKLC is a high level graphical front-end application that allows to author a keyboard layout. It consumes and produce a `.klc` file. When instructed to produce the actual layout DLL, MSKLC delegates the work to a low-level tool called `kbdutool` which, itself is able to produce the source code for a Windows DLL which is ultimately compiled by a C compiler and linker accompanying the MSKLC distribution.

## Creation of the keyboard layout DLL

### Using a private Docker Image

Before starting the build proper, [a private Docker image](https://github.com/springcomp/optimized-azerty-win/blob/dd8448402c373365462d5d99b0d9581d83002989/appveyor.yml#L17) is created each time the build starts. The private image is based upon a custom but [publically available](https://hub.docker.com/r/springcompdocker/msklc) image that hosts the MSKLC distribution.

The private image adds a [custom `Make-KeyboardLayout.ps1` PowerShell script](https://github.com/springcomp/optimized-azerty-win/blob/master/context/Make-KeyboardLayout.ps1) that automates most of the work needed to actually produce a resulting keyboard layout DLL.

### Creating the C Source Code for the keyboard layout

For practical reasons, the main keyboard layout source file is encoded in UTF-8 for better support in Git and development tools (e.g _diff_ and _merge_ tools). `kbdutool`, however, works best with UTF-16LE encoded files. Therefore, the first step of the build is to [create a copy of the `.klc` file in Unicode](https://github.com/springcomp/optimized-azerty-win/blob/dd8448402c373365462d5d99b0d9581d83002989/appveyor.yml#L20).

The resulting copy is stored in the `src` folder which is used as [a shared filesystem folder](https://github.com/springcomp/optimized-azerty-win/blob/dd8448402c373365462d5d99b0d9581d83002989/appveyor.yml#L25) for the Docker image that builds the keyboard layout DLL.

MSKLC itself does not lend itself well to automation. However, its low-level counterpart, `kbdutool` is a console program that can be used to create the C source code files from the `.klc` layout file with [its `-s` command-line option](https://github.com/springcomp/optimized-azerty-win/blob/dd8448402c373365462d5d99b0d9581d83002989/context/Make-KeyboardLayout.ps1#L172).

Unfortunately, the resulting C source code files are not properly encoded and loose most Unicode characters. Those characters are important to correctly specify the localized names of the keys, such as __??chap.__ for <kbd>Esc</kbd>. Fortunately, localized key names in French only use the `??` and `??` accented characters. When creating the C source code files, `kbdutool` replaced Unicode characters that it could not deal with by the well-known Unicode characters `???` (REPLACEMENT CHARACTER U+FFFD). So it is a simple matter to [replace this character](https://github.com/springcomp/optimized-azerty-win/blob/master/context/Make-KeyboardLayout.ps1#L86-L104) and restore the more appropriate accented characters in the C file.

Left to its own devices, `kbdutool` only ever produces a resulting DLL with the fixed version `1.0.3.40`. For a better experience, the build [injects the current build version number](https://github.com/springcomp/optimized-azerty-win/blob/master/context/Make-KeyboardLayout.ps1#L106-L138) into the .RC source file. 

### Compiling the keyboard layout DLL

To compile the actual keyboard layout DLL, `kbdutool` is invoked multiple times, each of which corresponds to a target CPU architecture. While, by default, MSKLC builds four target DLLs, only the __x86__ and __x64__ variants are in common use today on Windows??.

However, once invoked, `kbdutool` produces the desired layout DLL and then _deletes_ the C source code files! To prevent that, the build [first protect the files in read-only mode](https://github.com/springcomp/optimized-azerty-win/blob/master/context/Make-KeyboardLayout.ps1#L74-L82), then invokes multiple compilations of the keyboard layout DLL ??? for different CPU architectures ??? and then restore the read-write mode on the files (mostly, for symmetry reasons).

The resulting keyboard layout DLLs are compiled and [copied to their corresponding folder](https://github.com/springcomp/optimized-azerty-win/blob/master/context/Make-KeyboardLayout.ps1#L44), in the source code for the __x86__ and __x64__ variants of the [Windows Installer XML (WiX) Toolset](https://wixtoolset.org/releases/) setup package projects.

## Compilation of the setup packages

While MSKLC is able to produce setup packages to install the resulting keyboard layout, `kbdutool` is not. Likewise, MSKLC generates setup packages with a fixed version number of `1.0.3.40`. Finally, MSKLC supports a single keyboard layout DLL per setup package.

For these reasons, this repository comes with its own set of projects to create setup packages that support installing multiple keyboard layout DLLs. This is useful to support two variants of the AZERTY-NF layout ??? one that strictly complies with the standard, and another one that implements the popular SGCaps feature.

As an added benefit, it is very easy to inject the current build version number in while compiling the setup packages.

### Reusing MSKLC Custom Actions

One challenge, however, is to be able to register the keyboard and add to the _Language Bar_ the layout(s) upon installation, and conversely, to remove the layout(s) from the _Language Bar_ and unregister them from Windows?? upon uninstallation.

This, in Windows Installer parlance is performed with _Custom Actions_, which are custom code that ship with the setup packages for the sole purposes of supporting custom behavior during installation and uninstallation. To perform these tasks, this repository reuses the Custom Actions `KbdMsi.dll` file that ships with MSKLC.

The `KbdMsi.dll` file is only available in __x86__ but can be used in both setup packages. The file is therefore [copied as the last step](https://github.com/springcomp/optimized-azerty-win/blob/master/context/Make-KeyboardLayout.ps1#L198) of the Docker-based build, into the corresponding folder of the __x86__ setup package project folder.

### Building the setup Packages

The remaining part of the build ??? that consists in [compiling the setup packages](https://github.com/springcomp/optimized-azerty-win/blob/master/appveyor.yml#L39-L40) and a simple `setup.exe` bootstrapper app ??? is done effortlessly using `msbuild` and `Wix`, both of which are already installed in the Windows?? Appveyor image.

### Gathering build artifacts

Finally, [build artifacts are gathered using `7Zip`](https://github.com/springcomp/optimized-azerty-win/blob/master/appveyor.yml#L42-L45) to produce an archive that can be [made available on each build](https://github.com/springcomp/optimized-azerty-win/blob/master/appveyor.yml#L47-L50) or [pushed as a GitHub Release](https://github.com/springcomp/optimized-azerty-win/blob/master/appveyor.yml#L52-L62) when building a tagged version.
