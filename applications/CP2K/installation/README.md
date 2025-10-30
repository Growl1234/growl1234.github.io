## 从源代码配置CP2K

## <font color=red>This website is no longer updated. Please visit <a href="https://growl1234.readthedocs.io/">https://growl1234.readthedocs.io/</a>.</font>

***Last Updated: 2025-10-30***

**看思想家公社（sobereva）的文章[《CP2K第一性原理程序在Linux中的安装方法》](http://sobereva.com/586)即可，toolchain一步可以根据自己的实际需求作修改。**

下面给一点补充说明：

* toolchain会自动检查系统是否存在MKL配置，如果没有检测到MKL（包括新的oneMKL，下同），默认会把OpenBLAS、ScaLAPACK和FFTW一起安装下来，此时如果你已经事先有安装它们（包括其中任意一个）且环境变量配置正确，最好写上例如\--with-openblas=system这样的选项。**注意：无论OpenBLAS是否需要作为数学库安装，也无论其是否被标为“system”，其源码都会被下载并解压用以执行另外一个必要的检查步骤，而关于OpenBLAS的一切直接或间接设定只能影响其会不会被编译和安装。**

* **不要使用Intel oneAPI做并行化编译器，因为CP2K（截至v2025.1）还没有做好对ifx的支持（而新的oneAPI已经不再支持较旧的ifort），** 虽然toolchain一步可能会成功但后续编译步骤会出现内存爆浆问题而中断（亲身实践教训）。与oneAPI的并行编译器不同，**新的Intel oneMKL是受支持的**；建议安装好oneMKL后进入fftw3xf目录（例如/opt/intel/oneapi/mkl/2025.1/share/mkl/interfaces/fftw3xf）手动编译产生fftw3库文件（在该目录运行“make libintel64”；根据Makefile的设定，编译过程使用icx和gcc都可以，然而实际启动编译时如果没有检测到icx就跳过gcc检查直接报错，所以如果想用gcc必须显式指定“CC=gcc”）。

* 根据个人测试经验，OpenMPI并行结合oneMKL数学库选项配置的CP2K在运行时会出现内存配置错误，而MPICH不会，原因不明。鉴于这一情况，我暂且建议在使用Intel oneMKL作为数学库时用MPICH作为并行化工具来配置和运行CP2K，而在使用开源的OpenBLAS、ScaLAPACK和FFTW组合当数学库时放心用社区流行度更高的OpenMPI。

* \--with-tblite代表安装Grimme的tblite程序，要加这个必须同时加上\--with-ninja。按照CP2K的说明，tblite同时包含DFT-D4，因此这时也就不用刻意加\--with-dftd4了（即使加上了也会被自动跳过）。

* 如果你使用自行事先编译的HDF5并加了“\--with-hdf5=system”选项，toolchain脚本默认会自动把“-lsz”加到arch设置里，此时应当在正式编译CP2K前先运行“sudo dnf install libaec-devel”命令装上libsz，否则会出现“找不到-lsz”的错误提示。

