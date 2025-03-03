## 配置VASP和一些重要工具（vaspkit、VTST）

### 编译VASP

**【写在前面：一定要检查自己的make、cmake、gcc、g++、gfortran等编译所需要的最基本的程序包有没有安装好，否则无法编译VASP。】**

一般推荐使用Intel OneAPI。对于这种方法，网上有不少配置编译VASP的教程，相差不大。唯一需要注意的是可能需要根据你所安装的Intel OneAPI版本修改“makefile.include”里面的部分内容（以v2025.0为例，修改“icc”为“icx”，“icpc”为“icpx”，“mpiifort”为“mpiifx”），以及清空MKLROOT后面的示例路径（或在整行前加#）。

根据[官网的这个链接](https://www.vasp.at/wiki/index.php/Personal_computer_installation)，也可以使用OpenMPI在自己的个人计算机上进行编译，对于这种方法，操作要稍麻烦一些，因为需要额外安装一些相关的库。可以参考前面的官网指南或者[这里](https://implant.fs.cvut.cz/vasp-compilation/)的教程来配置编译VASP。

### 配置vaspkit

Vaspkit是一个很方便的用作VASP预-后处理的独立程序包，由国内师生开发，对初学者强烈推荐（尤其在另一优秀的同类程序包p4vasp早已停止开发且无法在现行版本中安装和使用的背景下）。

如果需要配置vaspkit，只需从[SourceForge](https://sourceforge.net/projects/vaspkit/)上下载源码并按“how_to_install”文件的说明安装即可，非常简单。

**注：一定要把主文件夹“.vaspkit”文件里面PBE、LDA等（具体看你自己安装的VASP带了说明赝势库）的路径改成自己计算机里面的实际路径；为方便调用，建议将安装目录的bin文件夹写入环境变量。**

### 配置VTST

如果你想用VASP进行反应动力学和过渡态的计算，则需要将VTST源码配置进VASP中。步骤如下：

1. 确保VASP已经顺利编译并能通过测试，以防万一配置VTST后重新编译出现问题时难以确定问题所在。

2. 仔细阅读并按照[VTST官网的配置说明](https://theory.cm.utexas.edu/vtsttools/installation.html)进行必要的文件修改和覆盖操作（为以防万一，建议提前备份好原有src目录下的文件）。

3. 回到编译时的目录，输入指令“make veryclean”清除之前的编译，然后“make”或“make all”（取决于你之前的编译指令选择）重新编译。

**注：**

**1. 在修改文件时，一定要细致注意空格和制表符留空的区别，否则在Linux下会造成文件读取方面的问题！**

**2. 最后一步很多教程没有特别说明“make”一步是否需要清除以前的编译，个人亲自实践表明，如果不运行“make veryclean”而直接启动编译，很快就会出现报错信息而失败中断。究其原因，主要是当你之前编译过VASP后，会生成一系列的目标文件（通常以.o结尾，比如main.o、vasp.o等）。如果不执行make veryclean，这些旧的目标文件会残留在编译目录下，在后续重新编译时，编译器可能会尝试基于这些旧目标文件进行部分编译工作，而这些旧目标文件是基于之前未配置VTST或者未进行当前相关修改的状态生成的。这就可能导致新旧配置以及修改前后的代码在函数定义、数据结构等方面不兼容，出现参数不匹配等编译错误。**
