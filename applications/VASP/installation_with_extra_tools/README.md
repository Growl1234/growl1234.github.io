## 配置VASP和一些重要工具（vaspkit、VTST）

## <font color=red>This website is no longer updated. Please visit <a href="https://growl1234.readthedocs.io/">https://growl1234.readthedocs.io/</a>.</font>

***Last Updated: 2025-10-30***

### 编译VASP

**【写在前面：一定要检查自己的make、cmake、gcc、g++、gfortran等编译所需要的最基本的程序包有没有安装好，否则无法编译VASP。】**

编译的简要步骤为：将arch文件夹里你希望选择的makefile.include选项的文件复制到软件包主目录下并去掉注释性后缀（即更名为makefile.include），然后仔细检查里面的各种参数（必要时修改），最后执行“make DEPS=1 -jN all”即可（N指编译所用核数；此处使用“-j N”并行时“DEPS=1”不能漏写）。

对于Intel处理器，官网和很多教程都推荐使用Intel oneAPI + MKL，这种方法也是配置起来最简便的。对于这种方法，网上有不少配置编译VASP的教程，相差不大。唯一需要注意的是可能需要根据你所安装的Intel OneAPI版本修改“makefile.include”里面的部分内容（以v2025.0为例，修改“icc”为“icx”，“icpc”为“icpx”，“mpiifort”为“mpiifx”），以及清空MKLROOT后面的示例路径（或在整行前加#）来让编译文件读取系统默认的真实路径。另外，建议将其中的OFLAG参数里加入-xhost，这样编译器会使得编译出的程序能够利用当前机子CPU能支持的最高档次的指令集以加速计算，可以省去一些不必要的麻烦。

*（附注：在个人计算机上的测试表明，即使是在Intel处理器上，Intel oneAPI + MKL这一官方“优化”套餐也并不比用OpenMPI+OpenBLAS+FFTW+ScaLAPACK经典套餐有优势，反而在使用了大小核设计的CPU上可能因调度策略问题反而计算速度更慢）*

根据[官网的这个链接](https://www.vasp.at/wiki/index.php/Personal_computer_installation)，也可以使用GCC+OpenMPI在个人计算机上进行编译。这种方法编译出来的VASP受硬件约束可能较小，但操作要稍麻烦一些，因为需要额外安装一些相关的库，并额外进行一些“makefile.include”文件的编辑工作。对于这类情况，可以参考前面的官网指南或者[这里](https://implant.fs.cvut.cz/vasp-compilation/)的教程来配置编译VASP。此时，上面提到的“-xhost”应当改为加“-march=native”。

对于AMD处理器，AMD也专门提供了针对其优化的[AOCC](https://www.amd.com/zh-cn/developer/aocc.html)和[AOCL](https://www.amd.com/en/developer/aocl.html)。使用AOCC+AOCL以及依靠它们编译的OpenMPI编译VASP的教程[见此](https://implant.fs.cvut.cz/vasp-cpu-aocl/)。

VASP开发者强烈建议加上HDF5的支持，此时需要注意使用的HDF5的编译环境，应与VASP的编译环境相同（即使用相同的编译器，需要在预配置HDF5时手动指定）。

VASP的DFT-D4功能通过向外链接程序包实现。如果想用DFT-D4，需要在“makefile.include”中自行加上链接到DFT-D4程序包的配置选项（见[DFT-D4官网](https://dftd4.github.io/dftd4/)）。

### 配置vaspkit

Vaspkit是一个很方便的用作VASP预-后处理的独立程序包，由国内师生开发，对初学者强烈推荐（尤其在另一优秀的同类程序包p4vasp早已停止开发且无法在现行版本中配置和使用的背景下）。

如果需要配置vaspkit，只需从[SourceForge](https://sourceforge.net/projects/vaspkit/)上下载源码并按“how_to_install”文件的说明或[官网说明](https://vaspkit.com/installation.html#id1)安装即可，非常简单。

**注：一定要把主文件夹“.vaspkit”文件里面PBE、LDA等（具体看你自己安装的VASP带了什么赝势库）的路径改成自己计算机里面的实际路径；为方便调用，建议将安装目录的bin文件夹写入环境变量。**

### 配置VTST

如果你想用VASP进行反应动力学（注意不要与常说的分子动力学模拟混淆）和过渡态的计算，建议将VTST源码配置进VASP中。步骤如下：

1. 确保VASP已经顺利编译并能通过测试，以防万一配置VTST后重新编译出现问题时难以确定问题所在。

2. 仔细阅读并按照[VTST官网的配置说明](https://theory.cm.utexas.edu/vtsttools/installation.html)进行必要的文件修改和覆盖操作（为以防万一，建议提前备份好原有src目录下的文件）。

3. 回到编译时的目录，输入指令“make veryclean”清除之前的编译，然后重新编译。

**注：**

**1. 在修改文件时，一定要细致注意空格和制表符留空的区别，否则在Linux下会造成文件读取方面的问题！**

**2. 最后一步很多教程没有特别说明“make”一步是否需要清除以前的编译，个人亲自实践表明，如果不运行“make veryclean”而直接启动编译，很快就会出现报错信息而失败中断。究其原因，主要是当你之前编译过VASP后，会生成一系列的目标文件（通常以.o结尾，比如main.o、vasp.o等）。如果不执行make veryclean，这些旧的目标文件会残留在编译目录下，在后续重新编译时，编译器可能会尝试基于这些旧目标文件进行部分编译工作，而这些旧目标文件是基于之前未配置VTST或者未进行当前相关修改的状态生成的。这就可能导致新旧配置以及修改前后的代码在函数定义、数据结构等方面不兼容，出现参数不匹配等编译错误。**
