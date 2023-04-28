---
layout: post
title:  "Installing windows-build-tools to allow Python builds."
author: magnus
categories: [ SPFx, Python ]
image: assets/images/3.jpg
beforetoc: "windows-build-tools is used for some python tools"
toc: true
---
If you are a developer working on a Windows machine, you may have encountered issues when building native Node.js modules. This is because some Node.js modules require compilation of native C/C++ code, which typically requires a compiler toolchain to be installed on the machine. The process of setting up a toolchain on Windows can be complex and time-consuming, especially for beginners or those without a background in C/C++ development.

To simplify the process of setting up a toolchain on Windows, Microsoft has released an npm package called windows-build-tools. This package includes the necessary build tools, such as the Visual C++ Build Tools and Python, required to build native Node.js modules on a Windows machine.

Install it with the following command:

```c
npm install --global windows-build-tools
```
With windows-build-tools, developers can easily install and configure the required toolchain with just a few simple commands, saving time and effort compared to setting up the toolchain manually. Additionally, the package automatically sets the necessary environment variables to ensure the tools are available to Node.js when building native modules.

The windows-build-tools package is especially useful for developers working with popular Node.js packages like node-gyp or bcrypt, which require compilation of native code. By simplifying the toolchain setup process, windows-build-tools allows developers to focus on building their applications rather than troubleshooting build toolchain issues.

Overall, the windows-build-tools package is a valuable tool for Windows developers who need to build native Node.js modules. Its ease of use and simplicity make it an attractive option for developers of all levels of experience.