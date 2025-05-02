## 从源代码配置CP2K

**写在前面：**

- **下面代码中可能涉及到的一些路径都是以我本人的Ubuntu系统上安放的路径为例，实际操作时千万不要忘记改成自己的！**

- **如果你使用的不是Ubuntu，而是Rocky Linux等，无妨，对于这个教程，除涉及直接从apt包管理器安装的步骤换成dnf或yum等（属于Linux基本的操作常识，更何况下面的正式步骤也不怎么涉及），其他操作都是一样的。想在Windows系统上用，只能通过WSL子系统环境的Linux控制台来用，对于这种情况，我无法提供帮助（而且有什么理由不装VMware虚拟机呢？）。**

- **强烈建议检查自己的make、cmake、gcc、g++、gfortran等绝大多数编译都需要的最基本的程序包和库有没有安装好，否则编译过程（不局限于CP2K本身）很可能会从一开始就失败。大部分以lib开头的库都包含在anaconda中，因此非常推荐在Linux上安装anaconda并在安装时选择同意写入环境变量（虽然对于CP2K的配置应该不是必需）。**

### 1. 下载并解压CP2K源码

从[官方GitHub项目页面](https://github.com/cp2k/cp2k/releases/)直接下载即可。请选择发行文件中的cp2k-X.Y.tar.bz2文件。下载好后，解压到自己计划放置CP2K源码程序的位置（我自己的是/home/uw/CP2K/src/cp2k-2025.1）。你也可以选择git clone方式下载实时开发版本，但安装途中出现错误的几率会增大。

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
./install_cp2k_toolchain.sh --with-sirius=no --with-plumed=install --with-cmake=system --with-hdf5 --with-ninja=system --with-dftd4 -j 4
```

下面给出一些说明，建议大家了解（部分内容搬运自sobereva的文章，部分内容是我根据自己的情况改写/补充的）：

* toolchain会自动检查系统中是否存在MPI、编译器等应有配置。如果没有检测到MPI，也没有设置MPI的install选项，toolchain会直接跳过MPI阶段和一些其他库（如Scalapack、COSMA、ELPA）的安装阶段而继续进行。如果没有检测到编译器，默认toolchain会自动安装gcc。

* toolchain也会自动检查系统是否存在MKL配置，如果没有检测到MKL（包括新的oneMKL，下同），一般默认安装OpenBLAS、Scalapack和fftw3等，此时如果你已经事先有安装它们（包括其中任意一个）且环境变量配置正确，最好写上例如\--with-openblas=system这样的选项；如果检查到MKL，则Scalapack和fftw3等的安装将被跳过。**注意：无论OpenBLAS是否需要作为数学库安装，也无论其是否被标为“system”，其源码都会被下载并解压用以执行另外一个必要的检查步骤，而关于OpenBLAS的一切直接或间接设定只能影响其会不会被编译和安装。**

* **不要使用Intel oneAPI做并行化编译器，因为CP2K（截至v2025.1）还没有做好对ifx的支持（而新的oneAPI已经不再支持较旧的ifort），** 虽然toolchain一步可能会成功但后续编译步骤会出现内存爆浆问题而中断（亲身实践教训）。与oneAPI的并行编译器不同，**新的Intel oneMKL是受支持的**，但不要忘记在安装好oneMKL后进入fftw3xf目录（我的是/opt/intel/oneapi/mkl/2025.1/share/mkl/interfaces/fftw3xf）手动编译产生fftw3库文件（在该目录运行“make libintel64”；根据Makefile的设定，编译过程使用icx和gcc都可以，然而实际启动编译时如果没有检测到icx就跳过gcc检查直接报错，所以如果想用gcc必须显示指定“CC=gcc”）。根据个人测试经验，OpenMPI并行结合oneMKL数学库选项配置的CP2K在运行时会出现难以解决的内存错误，而MPICH不会，因此**总建议在使用oneMKL作为数学库时搭配MPICH作为并行化工具来配置和运行CP2K。**

* \--with-cmake一项默认是install，即无论系统是否装有cmake，只要没有显式指定\--with-cmake=system，toolchain都将默认自动下载和编译cmake（其他默认install的程序和库同理）。前面我已经建议大家装上cmake，所以这里加上\--with-cmake=system用当前系统里的cmake，能节约编译时间。

* \--with-plumed=install代表安装默认不自动装的PLUMED库，这使得CP2K可以结合PLUMED做增强采样的从头算动力学。如果你不需要此功能的话可以不加这个选项，可以节约少量编译时间。

* \--with-dftd4代表安装DFT-D4程序，要加这个必须同时加上\--with-ninja。CP2K官网库给的DFT-D4源码包不完整，我建议去GitHub上DFT-D4官方仓库的发行页把相同版本dftd4源码包（.tar.xz）下载、解压并重新压缩为tar.gz格式，复制到/tools/toolchain/build目录下，让toolchain直接读取和安装，绕过从官网链接下载，能够避免一些莫名其妙的报错。

* \--with-sirius=no选项代表不装本来自动会装的SIRIUS库。这个库使得CP2K可以像VASP、Quantum ESPRESSO（免费）这类程序一样完全基于平面波+赝势做计算，但一般这用不上，想做这种计算的人一般直接就用VASP或者QE了。

* toolchain默认用所有CPU核心并行编译，可以自行加上-j [并行核数]来明确指定用多少核。编译的耗时和CPU核数关系很大，我本人编译了近两个小时（其中libint库的安装花了40多分钟之久）。

* 注意硬盘的空余空间应当足够。本人在上述命令执行完毕后，toolchain/build目录约占7GB，toolchain/install目录占约1.4GB。如果硬盘吃紧，建议toolchain运行成功后把这个build目录删掉，里面的文件之后用不着。（install目录千万别删！）

* 如果toolchain运行过程中某个库编译失败，可以检查终端的显示信息，或者去toolchain/build目录下的那个库的目录中去找编译过程输出的log文件，在里面搜error，根据报错试图分析原因并解决问题。toolchain运行失败后可以重新运行，**它会根据toolchain/build目录的内容做判断，之前已经下载和编译成功的库会自动跳过，而从失败的库继续编译。** 如果把build和install目录都删了，则toolchain会从头执行，因此千万不要toolchain一有报错就随意删东西。

* 如果在安装某个库的过程中提示wget失败（failed to download）（这也是我自己遇到的问题），那么几乎一定是因为网速原因导致那些库的压缩包下载不了（在大陆区域不可描述的访问国际互联网的条件下尤为常见）。去toochain/build目录下看正在装的这个库的压缩包，往往发现大小很小且增速极为缓慢（甚至有大小为0的情况），说明就是这个问题所致。解决方法不言而喻。如果懒得去查Linux终端如何实现那个东西，那就到[官网的这个链接](https://www.cp2k.org/static/downloads/)逐一下载正确版本的所需库文件（需要对照script目录下的安装脚本），或者干脆问已经安装好CP2K且没删build目录的人要CP2K编译过程中要用到的各种包放到toolchain/build目录下，这样压缩包被检测到，wget步骤就被自动跳过了。

### 3. 编译

（大部分内容搬运自sobereva的文章，我根据自己的情况改写/补充了点内容）

接着上一节，现在把/home/uw/CP2K/src/cp2k-2025.1/tools/toolchain/install/arch目录下所有文件拷到/home/uw/CP2K/src/cp2k-2025.1/arch目录下。这些文件定义了编译不同版本的CP2K所需的参数，其内容是toolchain脚本根据装的库和当前环境自动产生的。

然后在/home/uw/CP2K/src/cp2k-2025.1目录（编译目录）下运行以下命令：

```shell
source /home/uw/CP2K/src/cp2k-2025.1/tools/toolchain/install/setup
make -j 4 ARCH=local VERSION="ssmp psmp"
```

-j后面的数字是并行编译用的进程数（核数）。第一个命令可以直接去相应路径source setup后，再退回编译目录执行第二个命令。

***注：如果编译中途报错，并且从后往前找error的时候看到“找不到-lz”的报错提示，则运行sudo apt install zlib1g命令装上zlib库，再重新运行上面的make那行命令即可。***

现在，编译出的可执行程序都产生在了/home/uw/CP2K/src/cp2k-2025.1/exe/local目录下，这里面cp2k.popt、cp2k.psmp、cp2k.sopt、cp2k.ssmp就是我们所需要的CP2K的可执行文件了（popt和sopt在形式上分别是psmp和ssmp的符号链接，但psmp和ssmp支持OpenMP共享内存方式并行而popt和sopt不能）。

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

Sobereva提供了一个小的测试性输入文件，可以自己去网站下载。也可以自己用一些简单任务的输入文件做测试。

### 5. 可执行文件类型

最后，对按照上述流程编译得到的四个CP2K可执行文件版本作简要介绍：

sopt：只支持单机单核运行，调用这个版本跑任务等于找死。

ssmp：支持OpenMP共享内存方式的多核并行，但不支持MPI并行，因此在计算集群中无法跨节点并行，且大多数情况下并行效率明显不如popt；CP2K的GitHub发行页面也提供了一个预编译版本的ssmp文件，该预编译版本考虑到硬件软件兼容性等问题，编译选项较为保守，自然也就更不如从源码自行编译的了。通常也不推荐使用该版本。

popt：支持MPI并行，但不支持OpenMP并行，MPI支持使其可以跨节点并行。一般运行计算任务时建议默认使用该版本。

psmp：同时支持MPI和OpenMP并行。如果一个任务对内存占用需求很大（比如杂化泛函的计算），使用该版本并恰当设置MPI进程数a和OpenMP线程数b（此时是每个MPI进程下的；总占用核数为a*b）将是性价比不错的选择。

从文件资源管理器中的形式可以看到，popt和sopt分别是psmp和ssmp的符号链接。实际上，popt严格等于psmp结合OMP_NUM_THREADS=1（即OpenMP线程数强行设定为1），sopt同理。

除此之外，还有sdbg和pdbg两类可执行文件版本，但这是基于开发目的用的版本，对我们拿CP2K为了跑计算任务的人没用，所以前面我也没有编译它们。
