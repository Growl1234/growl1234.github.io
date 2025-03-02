## 配置VASP和VTST

### 编译VASP

网上有不少教程，相差不大。

一般推荐使用OneAPI。对于使用这一MPI库的同学，唯一需要注意的是可能需要根据你所安装的Intel OneAPI版本修改“makefile.include”里面的部分内容（以v2025.0为例，修改“icc”为“icx”，“icpc”为“icpx”，“mpiifort”为“mpiifx”），以及清空MKLROOT后面的示例路径（或在整行前加#）。

根据[官网的这个链接](https://www.vasp.at/wiki/index.php/Personal_computer_installation)，也可以使用OpenMPI类库在自己的个人计算机上进行编译，如选用此方法，一字不差严格按照链接中的教程操作即可。

### 配置VTST

首先，确保VASP已经顺利编译并能通过测试，以防万一配置VTST后重新编译出现问题时难以确定问题所在。

然后，仔细阅读并按照[VTST官网的配置说明](https://theory.cm.utexas.edu/vtsttools/installation.html)进行必要的文件修改和覆盖操作（为以防万一，建议提前备份好原有src目录下的文件）。

最后，回到编译时的目录，输入指令“make veryclean”清除之前的编译，然后“make”或“make all”（取决于你之前的编译指令选择）重新编译。

【也可以参考[Learn VASP the Hard Way里的相关文章](https://www.bigbrosci.com/2022/05/23/A31/?highlight=vtst)中的一些提示。】

**注：**

**1. 在修改文件时，一定要细致注意空格和制表符留空的区别，否则在Linux下很容易造成文件读取方面的问题！**

**2. 最后一步很多教程没有特别说明“make”一步是否需要清除以前的编译，个人亲自实践表明，如果不运行“make veryclean”而直接启动编译，很快就会出现报错信息而失败中断。究其原因，主要是当你之前编译过VASP后，会生成一系列的目标文件（通常以.o结尾，比如main.o、vasp.o等）。如果不执行make veryclean，这些旧的目标文件会残留在编译目录下，在后续重新编译时，编译器可能会尝试基于这些旧目标文件进行部分编译工作，而这些旧目标文件是基于之前未配置VTST或者未进行当前相关修改的状态生成的。这就可能导致新旧配置以及修改前后的代码在函数定义、数据结构等方面不兼容，出现参数不匹配等编译错误。**
