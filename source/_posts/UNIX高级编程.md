---
title: UNIX高级编程
date: 2023-06-01 16:42:18
tags: [C/C++, UNIX]
excerpt:  UNIX高级编程的学习
categories: UNIX
index_img: /img/index_img/17.png
banner_img: /img/banner_img/background18.jpg
---

## 环境配置

### 下载相关文件

[参考](https://blog.csdn.net/liqun_li/article/details/38053087)
下载src.3e.tar.gz源码
下载gcc、libbsd-dev、libbsd


### 配置vscode远程调试

![](https://raw.githubusercontent.com/univwang/img/master/202306011835352.png)

配置.vscode文件

```json
// c_cpp_properties 注意添加includePath
{
    "configurations": [
        {
            "name": "Linux",
            "includePath": [
                "${workspaceFolder}/**"
            ],
            "defines": [],
            "compilerPath": "/usr/bin/gcc",
            "cStandard": "c11",
            "cppStandard": "c++98",
            "intelliSenseMode": "linux-gcc-x64"
        }
    ],
    "version": 4
}
```

```json
// launch.json
{
  "version": "0.2.0",
  "configurations": [
      {
          "name": "run",
          "type": "cppdbg",
          "request": "launch",
          "program": "${fileDirname}/${fileBasenameNoExtension}",
          "args": [
            // ".."
          ],
          "stopAtEntry": false,
          "cwd": "${fileDirname}",
          "environment": [],
          "externalConsole": false,
          "MIMode": "gdb",
          "preLaunchTask": "compile",
          "setupCommands": [
              {
                  "description": "Enable pretty-printing for gdb",
                  "text": "-enable-pretty-printing",
                  "ignoreFailures": true
              }
          ]
      },
      
  ]
}
```


```json
// setting
{
    "files.associations": {
        "apue.h": "c"
    }
}
```

```json
// tasks
{
    "tasks": [
        {
            "type": "cppbuild",
            "label": "compile",
            "command": "/usr/bin/gcc",
            "args": [
                "-g",
                "-I",
                "${workspaceFolder}/include",
                "${file}",
                "-o",
                "${fileDirname}/${fileBasenameNoExtension}",
                "-l",
                "apue",
                "-L",
                "${workspaceFolder}/lib"
            ],
            "options": {
                "cwd": "${fileDirname}"
            },
            "problemMatcher": [
                "$gcc"
            ],
            "group": "build",
            "detail": "调试器生成的任务。"
        },
    ],
    "version": "2.0.0"
}
```