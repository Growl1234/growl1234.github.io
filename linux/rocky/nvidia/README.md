## <font color=red>This website is no longer updated. Please visit <a href="https://growl1234.readthedocs.io/">https://growl1234.readthedocs.io/</a> instead.</font>

## Rocky Linux安装NVIDIA显卡驱动的基本步骤和注意事项

***Last Updated: 2025-09-21***

**【写在前面：本文只是一些对官网教程的整理和补充，部分步骤官网给了很详细的教程我就直接贴链接了，不再赘述。】**

### 1. 安装驱动

严格按照[NVIDIA官网链接相应页面](https://docs.nvidia.com/datacenter/tesla/driver-installation-guide)操作（正式安装部分Rocky Linux属于“Red Hat Enterprise Linux”一节），至重新启动之前（完成下面的步骤2和3前不要重启）。注意一些指令里“$distro”“$(uname -i)”“$arch”等本应被识别的系统变量可能在终端执行时识别不出来或返回错误值而导致命令执行失败，此时根据教程章节开头的对应关系自行修改成相应值即可（本例中$distro是rhel10或rhel9，$(uname -i)和$arch都是x86_64）。

**特别提醒：<font color=red>不要自以为是或者听信一些低质量教程到NVIDIA驱动下载页面直接下载和运行“.run”后缀的程序来安装驱动！</font>这种安装方式早已因兼容性差和乱改系统文件而闻名遐迩。**


### 2. 禁用系统自带的开源驱动Nouveau，避免可能的冲突

运行以下指令：
```
sudo grubby --args="nouveau.modeset=0 rd.driver.blacklist=nouveau" --update-kernel=ALL
```

### 3. 关于安全启动（Secure Boot）的配置

大多数个人电脑通常都默认开启安全启动（Secure Boot），但这会阻碍NVIDIA显卡驱动在Linux下的运行。鉴于安全启动能起到的实际作用已经微乎其微，可以直接在重启时进入BIOS将其关掉；如果不想关安全启动，则需要导入dkms的密钥让设备进入Linux时允许运行该驱动程序，操作步骤如下：

（1）运行指令：
```
sudo mokutil --import /var/lib/dkms/mok.pub
```

（2）按照提示设置密码，建议设简单些（一个0都行），反正这个密码只会用一次。

（3）重启系统，此时会弹出MOK蓝色对话框。按任意键进入MOK管理界面，依次选Enroll MOK → Continue → Yes，然后输入上面你自己设置的密码，输入正确后选Reboot即可。


### 4. 重启系统

（这一步在步骤3中已涉及，若执行了步骤3则无需重复操作）

### 5. 运行指令“nvidia-smi”检查驱动是否被正确加载

如果正确加载，会输出显卡信息和使用显卡的进程信息。

如果未正确加载且报错显示有“cannot communicate with driver”，说明该指令压根就没有跟驱动程序正确对接上。出现这个问题，八成是关于安全启动的那一步没有弄，或者一根筋不信邪地通过“.run”驱动安装程序装了驱动；前者的话把涉及安全启动那一步完整弄了就行，后者的话考虑重装系统从头开始吧（如果你问只是卸载从“.run”安装的驱动然后重新按上述步骤装显卡是否能解决这个问题，回答是我没试过也不知道）。剩下两成可能是你的系统和软件环境跟驱动版本不兼容（本文所用的Rocky Linux 10绝对不存在这个问题），或者驱动程序版本和显卡硬件不兼容。

如果未正确加载且报错显示为“No devices were found”，说明该指令与驱动程序正确对接了，但却完全没有识别到显卡设备。这个问题不太容易归因和解决，应当自行查找系统上相关的日志文件对症下药。

对于一些电脑，如果BIOS中设置为独显直连（关闭集显或核显），当Linux系统没有正确加载驱动时，图形界面会直接以800*600等极低分辨率显示，且无法调节亮度。此时虽然可以确定显卡驱动加载出了问题，但具体问题仍然需要根据nvidia-smi的输出结果进行判断。



