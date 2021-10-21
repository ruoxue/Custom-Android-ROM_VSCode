# 安卓源码编译环境(VS Code)

## AVD迁移

我们已有的AVD，一般存放在 `C:\Users`下，例如我的AVD名字叫做**MyTestEnv***，那么在此目录下会存在一个 `MyTestEnv.ini`和 `MyTestEnv.avd`的文件夹，`MyTestEnv.ini`的位置不可以更改，我们要做的就是将 `MyTestEnv.avd`文件夹移动到一个特定的地方 `<Path U to save this avd>/MyTestEnv.avd/`，记住这个路径，然后打开，修改为如下模板即可：

```ini
avd.ini.encoding=UTF-8
# 主要是修改path
path=<Path U to save this avd>\MyTestEnv.avd
# path.rel改不改都可以（原来是什么就是什么，不该也没关系，改了好像也不影响）
path.rel=<Path U to save this avd>\MyTestEnv.avd
# 不要更改target（原来是什么就是什么），不过好像随便改也没什么问题
target=android_30

```

这样的话我们再次启动AVD便是从这个目录加载了。

## 自编译安卓源码ROM加载以及迁移

自己编译好的安卓源码将会输出在`<Android Source Code Path>\out\target\product\`下，一般来说用虚拟机就是编译x86版本，因此目录就是`<Android Source Code Path>\out\target\product\generic_x86\`。

找到我们的安卓系统镜像存放目录`<Your Android Home>\system-images`，找到AVD配置文件ini中target所对应的那个API镜像，进入此目录（可能含有多重子目录，进入包含`.img`文件的第一个目录），按照下述对应关系替换（以x86为示例）：

| 原本的文件        | 替换为（如果有-qemu后缀则替换为包含后缀的，没有则默认）                                  |
| ----------------- | ---------------------------------------------------------------------------- |
| build.prop        | \<Android Source Code Path>\out\target\product\generic_x86\System\build.prop |
| encryptionkey.img | \<Android Source Code Path>\out\target\product\generic_x86\encryptionkey.img |
| kernel-ranchu-64  | \<Android Source Code Path>\out\target\product\generic_x86\kernel-ranchu-64  |
| ramdisk.img       | \<Android Source Code Path>\out\target\product\generic_x86\ramdisk-qemu.img  |
| system.img        | \<Android Source Code Path>\out\target\product\generic_x86\system-qemu.img   |
| userdata.img      | \<Android Source Code Path>\out\target\product\generic_x86\userdata.img      |
| vendor.img        | \<Android Source Code Path>\out\target\product\generic_x86\vendor-qemu.img   |

此后如果想将镜像迁移到别处，则移动/复制该文件夹(依旧是包含`.img`的那个文件夹)到自己喜欢的目录，例如我将我自己的安卓环境设定为***MyTestImg***，所以目录为`<Path U to save system-imgs>\MyTestImg`，此时发现启动AVD报错，这是因为我们的AVD找不到我们的镜像了，进入AVD文件夹(如果迁移了则是迁移的文件夹，例如我的`<Path U to save this avd>/MyTestEnv.avd/`)，更改`config.ini`，更改`image.sysdir.1`，更改为`system.img`所在目录.
例如我的镜像存放在`C:\Users\<Username>\Desktop\AndroidEnv\MyTestImg\x86\`，则将其更改为`image.sysdir.1 = C:\Users\<Username>\Desktop\AndroidEnv\MyTestImg\x86\`。

```ini
# ……
image.sysdir.1 = <Path U restore ur custom-system-img>\
# ……
```

再次启动，等待一段时间，即可成功加载自己编译的安卓镜像。

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

| 内容                   | 命令                                         |
| ---------------------- | -------------------------------------------- |
| 输出日志到文件         | adb logcat -v time > AOSP.log                |
| 输出日志到控制台       | adb logcat -v time                           |
| 输出日志到文件和控制台 | adb logcat -v time\| AOSP.log                |
| 输出日志(过滤)         | adb logcat -v time\| find "需要包含的字符串" |

其中 `tasks.json`中只给出了一种方式，其他方式可以照猫画虎，因此不做赘述。
