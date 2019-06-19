While creating a keyboard layout with Microsoft Keyboard Layout Creator (MSKLC) is an easy task, this repository aims at automating as much as is possible in order to reduce the possibility to make mistakes and improve the experience while developing the keyboard layout.

This document gives some insights into how the build works.

### High-level overview

The build is split into roughly two stages:

1. Creation of the keyboard layout DLL.
2. Compilation of the x86 and x64 setup packages.

MSKLC is a high level graphical front-end application that allows to author a keyboard layout. It consumes and produce a `.klc` file. When instructed to produce the actual layout DLL, MSKLC delegates the work to a low-level tool called `kbdutool` which, itself is able to produce the source code for a Windows DLL which is ultimately compiled by a C compiler and linker accompanying the MSKLC distribution.

### Creation of the keyboard layout DLL

Before starting the build proper, [a private Docker image](https://github.com/springcomp/optimized-azerty-win/blob/dd8448402c373365462d5d99b0d9581d83002989/appveyor.yml#L17) is created each time the build starts. The private image is based upon a custom but [publically available](https://hub.docker.com/r/springcompdocker/msklc) image that hosts the MSKLC distribution.

For practical reasons, the main keyboard layout source file is encoded in UTF-8 for better support in Git and development tools (e.g _diff_ and _merge_ tools). `kbdutool`, however, works best with UTF-16LE encoded files. Therefore, the first step of the build is to [create a copy of the `.klc` file in Unicode](https://github.com/springcomp/optimized-azerty-win/blob/dd8448402c373365462d5d99b0d9581d83002989/appveyor.yml#L20).

The resulting copy is stored in the `src` folder which is used as [a shared filesystem folder](https://github.com/springcomp/optimized-azerty-win/blob/dd8448402c373365462d5d99b0d9581d83002989/appveyor.yml#L25) for the Docker image that builds the keyboard layout DLL.

MSKLC itself does not lend itself well to automation. However, its low-level counterpart, `kbdutool` is a console program an can be used to create the C source code files from the `.klc` layout file.