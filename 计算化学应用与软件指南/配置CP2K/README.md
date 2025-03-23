## 从源代码配置CP2K

**写在前面：**

- **下面代码中可能涉及到的一些路径都是以我本人的Ubuntu虚拟机上安放的路径为例，实际操作时千万不要忘记改成自己的！**

- **强烈建议检查自己的make、cmake、gcc、g++、gfortran等绝大多数编译都需要的最基本的程序包和库有没有安装好，否则编译过程（不局限于CP2K本身）很可能会从一开始就失败。大部分以lib开头的库都包含在anaconda中，因此吐血建议在linux上安装anaconda并在安装时选择同意写入环境变量（虽然对于CP2K的配置不是必需）。**

### 1. 下载并解压CP2K源码

从[官方GitHub项目页面](https://github.com/cp2k/cp2k/releases/)直接下载即可。请选择发行文件中的cp2k-X.Y.tar.bz2文件。下载好后，解压到自己计划放置CP2K源码程序的位置（我自己的是/home/uw/CP2K/src/cp2k-2025.1）

### 2. 通过源码的toochain脚本预安装需要的库

切到/home/uw/CP2K/src/cp2k-2025.1/tools/toolchain文件夹，里面有一个文件“install_cp2k_toolchain.sh”，这就是我们接下来要执行的文件。

下面的指令会告诉你运行该脚本会涉及的库和程序，以及默认的配置选项：

```shell
./install_cp2k_toolchain.sh --help
```

**请务必仔细阅读指令运行后的帮助文本！** 往往你需要根据自己的需要和计算机情况自行决定让一部分选项不以默认值进行。

给两个例子：我本人是以思想家公社（sobereva）的一篇文章[《CP2K第一性原理程序在Linux中的安装方法》](http://sobereva.com/586)里面的参数为主要参考的，文章作者设置了如下参数：

```shell
./install_cp2k_toolchain.sh --with-sirius=no --with-openmpi=install --with-plumed=install
```

这是我自己的设置：

```shell
./install_cp2k_toolchain.sh --with-sirius=no --with-plumed=install --with-cmake=system --with-mkl --with-hdf5=system -j 6
```

下面给出一些说明，建议大家了解（大部分内容搬运自sobereva的文章，我自己根据自己的情况改写/补充了点内容）：

* --with-sirius=no选项代表不装本来自动会装的SIRIUS库。这个库使得CP2K可以像VASP、Quantum ESPRESSO（免费）这类程序一样完全基于平面波+赝势做计算，但一般这用不上，想做这种计算的人一般直接就用VASP或者QE了。

* 这里我没有--with-openmpi=install一项，因为我的电脑已经事先安装好了OpenMPI（默认选项是system）。如果你的电脑上没有安装任何MPI，请加上这一选项。**不要使用Intel OneAPI，目前这一MPI不受支持，虽然toolchain一步会成功但后续编译过程会导致系统崩溃（官网信息+亲身实践教训）。**

* --with-cmake一项默认是install，因为toolchain默认自动下载和编译cmake。前面我已经建议大家装上cmake，所以这里加上--with-cmake=system用当前系统里的cmake，能节约编译时间。

* --with-mkl指使用Intel MKL数学库，设置此项会代替默认的--with-openblas=install项。其实这个一般不需要特别设置，因为如果toolchain脚本检测到了MKLROOT就会默认将Intel MKL作为首选。

* --with-hdf5=system默认是install，一般不要更改，我已经事先装好了所以这里设置了system。

* --with-plumed=install代表安装默认不自动装的PLUMED库，这使得CP2K可以结合PLUMED做增强采样的从头算动力学。如果你不需要此功能的话可以不加这个选项，可以节约少量编译时间。

* 从CP2K 2024.2开始支持了DFT-D4色散校正，这种校正的常识见[《DFT-D4色散校正的简介与使用》](http://sobereva.com/464)。想用DFT-D4的话必须再额外带上--with-ninja --with-dftd4。

* toolchain默认用所有CPU核心并行编译，可以自行加上-j [并行核数]来明确指定用多少核（如我就用了-j 6，之所以不敢用-j 8是因为被卡闪退过）。编译的耗时和CPU核数关系很大，我本人编译了近两个小时（其中libint库的安装花了40分钟之久）。

* 注意硬盘的空余空间应当足够。本人在上述命令执行完毕后，toolchain/build目录约占7GB，toolchain/install目录占约1.4GB。如果硬盘吃紧，建议toolchain运行成功后把这个build目录删掉，里面的文件之后用不着。

* 如果toolchain运行过程中某个库编译失败，可以检查终端的显示信息，或者去toolchain/build目录下的那个库的目录中去找编译过程输出的log文件，在里面搜error，根据报错试图分析原因并解决问题。toolchain运行失败后可以重新运行，**它会根据toolchain/build目录的内容做判断，之前已经下载和编译成功的库会自动跳过，而从失败的库继续编译。** 如果把build和install目录都删了，则toolchain会从头执行。

* 如果在安装某个库的过程中提示wget失败（failed to download）（这也是我自己遇到的问题），那么几乎一定是因为网速原因导致那些库的压缩包下载不了（在大陆区域不可描述的访问国际互联网的条件下尤为常见）。去toochain/build目录下看正在装的这个库的压缩包，往往发现大小为0，说明就是这个问题所致。解决方法是挂梯子加速访问国际互联网。当然一个更好的办法是直接去[官网的这个链接](https://www.cp2k.org/static/downloads/)预先下载好CP2K编译过程中要用到的各种包放到toolchain/build目录下（当然，同样需要梯子），这样压缩包被检测到，wget步骤就被自动跳过了。

### 3. 编译

（大部分内容搬运自sobereva的文章，我自己根据自己的情况改写/补充了点内容）

接着上一节，现在把/home/uw/CP2K/src/cp2k-2025.1/tools/toolchain/install/arch目录下所有文件拷到/home/uw/CP2K/src/cp2k-2025.1/arch目录下。这些文件定义了编译不同版本的CP2K所需的参数，其内容是toolchain脚本根据装的库和当前环境自动产生的。

然后在/home/uw/CP2K/src/cp2k-2025.1目录（编译目录）下运行以下命令：

```shell
source /home/uw/CP2K/src/cp2k-2025.1/tools/toolchain/install/setup
make -j 6 ARCH=local VERSION=psmp
```

-j后面的数字是并行编译用的核数。第一个命令可以直接去相应路径source setup后，再退回编译目录执行第二个命令。

***注：如果编译中途报错，并且从后往前找error的时候看到“找不到-lz”的报错提示，则运行sudo apt install zlib1g命令装上zlib库，再重新运行上面的make那行命令即可。若出现错误："找不到 -lsz"，则运行sudo apt install libsz2把szip库安装好，若仍未解决，请检查“libhdf5-openmpi-dev”是否正确安装，安装好后重新make。***

现在，编译出的可执行程序都产生在了/home/uw/CP2K/src/cp2k-2025.1/exe/local目录下，这里面cp2k.popt、cp2k.psmp、cp2k.sopt、cp2k.ssmp就是我们所需要的CP2K的可执行文件了（popt和sopt其实分别是psmp和ssmp的符号链接）。

随后，在主文件夹的.bashrc中添加如下两行：

```shell
source /home/uw/CP2K/src/cp2k-2025.1/tools/toolchain/install/setup
export PATH=/home/uw/CP2K/src/cp2k-2025.1/exe/local:$PATH
```

然后重新进入终端或者在当前终端运行指令：

```shell
source ~/.bashrc
```

就可以通过cp2k.ssmp等命令运行cp2k了。运行诸如cp2k.ssmp -v可以查看CP2K的版本、编译时用的库和参数信息。

**注：上面.bashrc里面添加的两行，第二行应该不难理解（环境变量）；至于为什么需要第一行也就是source那行，原因如下：有的库提供的.so文件是CP2K启动时所需的，source这个脚本使得相应的库的目录被加入到动态库的搜索路径中；另外，对于运行toolchain脚本时使用了--with-openmpi=install的，用了这个source指令之后toolchain过程中装的OpenMPI的可执行文件mpirun等才能直接用。**

### 4. 运行、测试

Sobereva提供了一个小的测试性输入文件，可以自己去网站下载。也可以自己用一些简单任务的输入文件做测试。测试成功，说明安装的CP2K没有问题，恭喜你，可以放心开始使用了。

**注：（来自sobereva）如果你是自己编译的CP2K，建议默认用popt版而不要用ssmp版，因为在某些情况下后者运行效率远不及popt版（但也有些任务二者速度差异不大，看具体情况）。为了运行popt版省事，建议在~/.bashrc里面加入一行alias cp2k='mpirun -np 8 cp2k.popt'。重新进入终端后，只要输入“cp2k test.inp \|tee test.out”就等价于输入“mpirun -np 8 cp2k.popt test.inp \|tee test.out”了，用起来方便多了。（这里后面的“\|tee test.out”指将终端上的输出信息保存在同目录下的“test.out”文件里，可以没有但个人建议加上。）**
