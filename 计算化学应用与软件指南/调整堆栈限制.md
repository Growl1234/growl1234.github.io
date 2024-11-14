# 调整堆栈限制（适用于用Ubuntu运行VASP的同学）

## 为什么要做这个小教程？
VMWare Workstation安装的Ubuntu虚拟机系统初始设置的堆栈容量仅为8192KB（8MB），对于稍微复杂一点的VASP计算不够用，这也是为什么一些任务会出错，报错内容如下：

forrtl : severe (174) : SIGSEGV, segmentation fault occurred. 

然后下面的列表通篇Unknown或者Error. 

#### 起初make test步骤一直失败，详细检查 "testsuite.log" 发现出现的错误全部都是堆栈容量不足导致的分段错误 (即上面的报错内容). 

## 临时改变堆栈容量

可以事先在该终端窗口输入以下命令：

ulimit -s 262140

【262140】表示大小（KB），也可以设为 "unlimited". 

## 永久改变堆栈容量

如果想永久性调整堆栈容量限制，可以通过以下三步实现：

### Step1:

在【/etc/security/limits.conf】文件中添加下面两行然后保存：

*soft stack 262140

*hard stack 262140

其中【*】表示对所有文件生效；【stack】表示堆栈的最大值；【262140】表示大小（KB）。

#### 注：可以设置为unlimited，但这里的修改是永久性修改，设置成unlimited可能存在运行其他程序时出现错误甚至崩溃的风险，故建议设置具体数字，这里的262140KB（256MB）已经足够几乎所有可以在自己的笔记本电脑运行的VASP计算程序用了. 


### Step2:

在【/etc/pam.d/login】文件中添加下面一行然后保存 **（建议与已有session项对齐）**：

session required pam_limits.so


#### Step3:

重新启动虚拟机.

