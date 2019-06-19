While creating a keyboard layout with Microsoft Keyboard Layout Creator (MSKLC) is an easy task, this repository aims at automating as much as is possible in order to reduce the possibility to make mistakes and improve the experience while developing the keyboard layout.

This document gives some insights into how the build works.

### High-level overview

The build is split into roughly two stages:

1. Creation of the keyboard layout DLL.
2. Compilation of the x86 and x64 setup packages.

MSKLC is a high level graphical front-end application that allows to author a keyboard layout. It consumes and produce a `.klc` file. When instructed to produce the actual layout DLL, MSKLC delegates the work to a low-level tool called `kbdutool` which, itself is able to produce the source code for a Windows DLL which is ultimately compiled by a C compiler and linker accompanying the MSKLC distribution.

### Creation of the keyboard layout DLL

For practical reasons, the main keyboard layout source file is encoded in UTF-8 for better support in Git and development tools (e.g _diff_ and _merge_ tools). `kbdutool`, however, works best with UTF-16LE encoded files. Therefore, the first step of the build is to [create a copy of the `.klc` file in Unicode](https://github.com/springcomp/optimized-azerty-win/blob/dd8448402c373365462d5d99b0d9581d83002989/appveyor.yml#L20).