## 配置VASP和VTST

### 编译VASP

网上有不少教程，相差不大，唯一需要注意的是可能需要根据你所安装的Intel OneAPI版本修改“makefile.include”里面的部分内容（以v2025.0为例，修改“icc”为“icx”，“icpc”为“icpx”，“mpiifort”为“mpiifx”），以及清空MKLROOT后面的示例路径。别的不再多言。

### 配置VTST

首先，确保VASP已经顺利编译，以防万一配置VTST后编译出现问题时无法确定问题所在。

然后，按照“https://theory.cm.utexas.edu/vtsttools/installation.html ”上的说明进行必要文件的修改。

最后，回到编译时的目录，输入指令“make veryclean”清除之前的编译，然后“make all”重新编译。
