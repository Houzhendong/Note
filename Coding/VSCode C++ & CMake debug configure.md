1. launch文件配置
```
{
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "(Windows) 启动",
            "type": "cppvsdbg",
            "request": "launch",
            "program": "${command:cmake.launchTargetPath}", 
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole":true ,
            "preLaunchTask": "MSVC Build" //如果只调试不生成，注释这行
        },
        {
            "name": "(gdb) 启动",
            "type": "cppdbg",
            "request": "launch",
            "program": "${command:cmake.launchTargetPath}", 
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "preLaunchTask": "g++ Build", //如果只调试不生成，注释这行
            "setupCommands": [
                {
                    "description": "为 gdb 启用整齐打印",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ]
        }
    ]
}
```

2. Task文件配置
```
{
    // See https://go.microsoft.com/fwlink/?LinkId=733558
    // for the documentation about the tasks.json format
    "version": "2.0.0",
    "options": {
        "cwd": "${workspaceFolder}/build"
    },
    "tasks": [
        {
            "label": "g++ cmake",
            "type":"shell",
            "command": "cmake",
            "args": [
                "-G",
                "'MinGW Makefiles'",
                ".."
            ],
            "group": "build",
        },
        {
            "label": "msvc cmake",
            "type": "shell",
            "command":"cmake",
            "args": [
                ".."
            ]
        },
        {
            "label": "make",
            "command":"cmake",
            "args": [                    
                "--build",
                "."
            ],
            "group": "build",
        },
        {
            "label": "g++ Build",
            "group": "build",
            "dependsOn":[
                "g++ cmake",
                "make"
            ],
        },
        {
            "label": "MSVC Build",
            "group": "build",
            "dependsOn":[
                "msvc cmake",
                "make"
            ],
        },
    ]
}
```