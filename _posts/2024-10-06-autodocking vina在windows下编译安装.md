---
layout: post
title: Autodock vina在windows下python编译安装
katex: True
tags: 软件工具
---

# <center>Autodock vina在windows下python编译安装</center> 


<div STYLE="page-break-after: always;"></div>

## 一、Autodock vina
AutoDock Vina是一款开源的分子对接软件，由Oleg Trott和Arthur J. Olson领导的研究团队在Scripps研究所开发。它可以用于预测小分子（如药物或配体）与生物大分子（如蛋白质）之间的相互作用和结合模式。该软件能够快速进行大量的对接计算，能够较准确地预测小分子与蛋白质的结合位置和结合能。

使用AutoDock Vina进行分子对接的基本步骤通常包括：
- 准备蛋白质受体结构文件，通常是PDB格式。
- 准备小分子配体的结构文件，通常是MOL2或SDF格式。
- 定义对接盒子（binding box），即预计配体与受体结合的区域。
- 运行AutoDock Vina进行对接计算。
- 分析对接结果，包括结合能、结合模式等。


## 二、windows下使用python编译安装

由于项目的需要，需要在windows下使用python对vina进行调用，这里基于Linux下python编译的setup.py文件进行修改，使得其可以在windows下顺利安装。安装前，需要配置好python环境，建议使用Anaconda，网上有大量教程可以参考，这里不再赘述。

配置好python环境后，即可开始vina模块的安装。首先第一步参照vina在github中的命令，安装编译必要的C++库:
```
conda install -c conda-forge numpy swig boost-cpp sphinx sphinx_rtd_theme
```
这里库文件会安装在Anaconda当前默认的python环境中，位置在Library文件夹。接下来下载Autodock vina的源文件，并切换到安装目录：
```
git clone https://github.com/ccsb-scripps/AutoDock-Vina
cd AutoDock-Vina/build/python
```
在Linux环境下，可以直接运行setup.py文件进行安装，然而在windows环境下需要对setup.py进行额外的配置和修改，因为直接运行会找不到链接的库文件以及编译器信息错误。下面对setup.py进行修改：
首先是几个库文件的定位，最关键的是boost库，定位到locate_boost函数，位于setup.py文件的112行，该函数的作用是查找boost库的路径。在windows环境下，需要修改该函数，使其能够正确地定位boost库。这里给出我根据Anaconda安装路径简单修改后的代码如：


```python
def locate_boost():
    """Try to locate boost."""
    if in_conda():
        if "CONDA_PREFIX" in os.environ.keys():
            data_pathname = os.environ["CONDA_PREFIX"] + "\\Library"
        else:
            data_pathname = sysconfig.get_path("data") # just for readthedocs build
        include_dirs = data_pathname + os.path.sep + 'include'
        library_dirs = data_pathname + os.path.sep + 'lib'
        
        if os.path.isdir(include_dirs + os.path.sep + 'boost'):
            print('Boost library location automatically determined in this conda environment.')
            return include_dirs, library_dirs
        else:
            print('Boost library is not installed in this conda environment.')
    else:
        include_dirs = 'D:\\Anaconda\\Library\\include\\'
        library_dirs = 'D:\\Anaconda\\Library\\lib\\'
        return include_dirs, library_dirs
```
修改方法无论is_conda判断为True还是False，均能够确定头文件和库文件正确位置。除了上述的修改，还有一个需要注意的地方，在setup.py文件CustomBuildExt类中，需要对python的扩展编译函数进行修改，在CustomBuildExt类中build_extensions函数259行开始添加下述语句：
```python
print('- extra link args: %s' % self.extensions[0].extra_link_args)
self.compiler.set_executable("compiler_so", "g++")
self.compiler.set_executable("compiler_cxx", "g++")
self.compiler.set_executable("linker_so", "g++")
# Replace current compiler to g++
self.compiler.compiler_so[0] = "g++"
self.compiler.compiler_so.insert(2, "-shared")
```
上述语句对默认的编译器进行修改，将其替换为g++，该编译器可以直接通过命令行进行调用，推荐使用Anaconda安装的g++，并将g++添加到环境变量中，具体方法可以参考网上教程。

在上述修改完成后，即可运行setup.py文件进行安装，安装完成后即可在python中调用vina模块。