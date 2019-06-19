While creating a keyboard layout with Microsoft Keyboard Layout Creator (MSKLC) is an easy task, this repository aims at automating as much as is possible in order to reduce the possibility to make mistakes and improve the experience while developing the keyboard layout.

This document gives some insights into how the build works.

### High-level overview

The build is split into roughly two stages:

1. Creation of the keyboard layout DLL.
2. Compilation of the x86 and x64 setup packages.

MSKLC is a high level graphical front-end application that allows to author a keyboard layout. It consumes and produce a `.klc` file. When instructed to produce the actual layout DLL, MSKLC delegates the work to a low-level tool called `kbdutool` which, itself is able to produce the source code for a Windows DLL which is ultimately compiled by a C compiler and linker accompanying the MSKLC distribution.

### Creation of the keyboard layout DLL

#### Docker Image

Before starting the build proper, [a private Docker image](https://github.com/springcomp/optimized-azerty-win/blob/dd8448402c373365462d5d99b0d9581d83002989/appveyor.yml#L17) is created each time the build starts. The private image is based upon a custom but [publically available](https://hub.docker.com/r/springcompdocker/msklc) image that hosts the MSKLC distribution.

The private image adds a [custom `Make-KeyboardLayout.ps1` PowerShell script](https://github.com/springcomp/optimized-azerty-win/blob/master/context/Make-KeyboardLayout.ps1) that automates most of the work needed to actually produce a resulting keyboard layout DLL.

#### C Source Code

For practical reasons, the main keyboard layout source file is encoded in UTF-8 for better support in Git and development tools (e.g _diff_ and _merge_ tools). `kbdutool`, however, works best with UTF-16LE encoded files. Therefore, the first step of the build is to [create a copy of the `.klc` file in Unicode](https://github.com/springcomp/optimized-azerty-win/blob/dd8448402c373365462d5d99b0d9581d83002989/appveyor.yml#L20).

The resulting copy is stored in the `src` folder which is used as [a shared filesystem folder](https://github.com/springcomp/optimized-azerty-win/blob/dd8448402c373365462d5d99b0d9581d83002989/appveyor.yml#L25) for the Docker image that builds the keyboard layout DLL.

MSKLC itself does not lend itself well to automation. However, its low-level counterpart, `kbdutool` is a console program that can be used to create the C source code files from the `.klc` layout file with [its `-s` command-line option](https://github.com/springcomp/optimized-azerty-win/blob/dd8448402c373365462d5d99b0d9581d83002989/context/Make-KeyboardLayout.ps1#L172).

Unfortunately, the resulting C source code files are not properly encoded and loose most Unicode characters. Those characters are important to correctly specify the localized names of the keys, such as __Échap.__ for <kbd>Esc</kbd>. Fortunately, localized key names in French only use the `é` and `É` accented characters. When creating the C source code files, `kbdutool` replaced Unicode characters that it could not deal with by the well-known Unicode characters `�` (REPLACEMENT CHARACTER U+FFFD). So it is a simple matter to [replace this character](https://github.com/springcomp/optimized-azerty-win/blob/master/context/Make-KeyboardLayout.ps1#L93-L101) and restore the more appropriate accented characters in the C file. 