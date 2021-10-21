# 安卓源码编译环境(VS Code)

## Tasks编写

首先给出Task代码(完整的 `tasks.json`不止于此)：

```json
"tasks": [
    {
        "label": "虚拟机启动",
        "type": "process",
        "command": "C:\\Android-SDK\\emulator\\emulator.exe",
        "args": [
            "-avd",
            "MyTestEnv"
        ]
    },
    {
        "label": "输出Log(专属)",
        "type": "shell",
        "command": "adb",
        "args": [
            "logcat",
            "-v",
            "time",
            "|",
            "find",
            "I",
            "|",
            "tee",
            "AOSP.log"
        ]
    },
    {
        "label": "输出Log(全部)",
        "type": "shell",
        "command": "adb",
        "args": [
            "logcat",
            "-v",
            "time",
            "|",
            "tee",
            "AOSP.log"
        ]
    }
]
```

首先是运行安卓虚拟机(AVD)[TODO:创建方法]，其相应的控制台命令如下：`%ANDROID_HOME%\\emulator\\emulator.exe -avd <Your AVD name>`，其中 `%ANDROID_HOME%`是你的安卓SDK安装目录[TODO:安装方法]。

![AVD启动过程](https://github.com/easternDay/ReadMe.IMGS/raw/master/AndroidEnv_Windows/%E5%90%AF%E5%8A%A8AVD.gif)

打开AVD模拟器之后，我们便可以利用如下命令进行一个log的输出到文件和控制台：

| 内容                   | 命令                                          |
| ---------------------- | --------------------------------------------- |
| 输出日志到文件         | adb logcat -v time > AOSP.log                 |
| 输出日志到控制台       | adb logcat -v time                            |
| 输出日志到文件和控制台 | adb logcat -v time \| AOSP.log                |
| 输出日志(过滤)         | adb logcat -v time \| find "需要包含的字符串" |

其中 `tasks.json`中只给出了一种方式，其他方式可以照猫画虎，因此不做赘述。