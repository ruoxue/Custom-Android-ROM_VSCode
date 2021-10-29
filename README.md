# 准备

## repo工具下载

```bash
mkdir ~/bin
PATH=~/bin:$PATH
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
```

![repo工具下载](https://cdn.jsdelivr.net/gh/easternDay/ReadMe.IMGS//20211022143643.png)

repo的运行过程中会尝试访问官方的git源更新自己，如果想使用tuna的镜像源进行更新，可以将如下内容复制到你的~/.bashrc里，然后重启终端模拟器。

```ini
export REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo'
```

## 下载指定分支

之所以我下载指定分支是因为整个的源码太大了，所以指定分支下载。

执行命令之前请确保自己安装了 `python`和 `git`并设置了 `git`的 `user.email`和 `user.name`，设置命令如下：

```bash
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
```

查看[安卓版本列表](https://source.android.com/setup/start/build-numbers#source-code-tags-and-builds)，此处我选择的是 `android-11.0.0_r46`，支持较多设备。接着创建工作目录：

```bash
mkdir <WORKING_DIRECTORY>
cd <WORKING_DIRECTORY>
```

在工作目录下运行:

```bash
repo init -u https://mirrors.tuna.tsinghua.edu.cn/git/AOSP/platform/manifest -b android-11.0.0_r46
```

同步源码树：

```bash
repo sync
```

# 源码编译

## 安装JDK
在实际编译过程中，其使用`openjdk-8`也可以，但是在生成部分文件的时候会出现一个错误的报 ***(Java Runtime版本错误)*** ，所以我们选择更高版本的JDK，这里也同样是IDEA的高版本选择。
```bash
sudo apt-get install openjdk-11-jdk
```

## 安装其他编译依赖

```bash
sudo apt-get install clang
sudo apt-get install lib32bz2-1.0
sudo apt-get install lib32z1
sudo apt-get install lib32ncurses5
sudo apt-get install lib32stdc++6
sudo apt-get install libcurses5:i386
sudo apt-get install cmake
sudo apt-get install ninja-build
sudo apt-get install cmake-doc
```

## 初始化编译环境

```bash
source build.envsetup.sh
```

## 编译版本选择

```bash
lunch
```

## 编译

```
time make [-c] [-j<线程数>]
```


# AVD迁移

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

# 自编译安卓源码ROM加载以及迁移

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

# Tasks编写(VS Code环境下)

首先给出Task代码：

```json
{
    // See https://go.microsoft.com/fwlink/?LinkId=733558
    // for the documentation about the tasks.json format
    "version": "2.0.0",
    "tasks": [
        {
            "label": "虚拟机启动",
            "type": "process",
            //此处安装目录记得修改为自己的安卓SDK安装目录
            "command": "C:\\Android-SDK\\emulator\\emulator.exe",
            "args": [
                "-avd",
                "MyTestEnv"
            ]
        },
        //Log输出
        {
            "label": "DDMS启动",
            "type": "process",
            "command": "C:\\Android-SDK\\tools\\monitor.bat"
        }
    ]
}
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

其中 `tasks.json`中采用的是安卓SDK自带的monitor(DDMS)，也更加简便。
