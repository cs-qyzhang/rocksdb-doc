---
title: "安装"
date: 2020-05-15
lastmod: 2020-05-17
authors: ["张丘洋"]
---

## 获取源代码

RocksDB在Ubuntu等发行版本的仓库中都没有编译好的二进制包，所以需要自行编译。首先你需要获取RocksDB的源代码，可以通过git clone命令来获取。本文档是基于RocksDB 6.4.6版本介绍的，你可以通过以下命令来获取源代码：

    git clone https://github.com/facebook/rocksdb.git --branch v6.4.6

你也可以直接下载发布的release包，可前往<https://github.com/facebook/rocksdb/releases>下载对应版本源代码的.zip或.tar.gz压缩包。

## 安装依赖

RocksDB使用了很多库，如压缩会用到snappy或zlib等。在编译RocksDB的时候可以进行配置，选择使用哪些库。不过刚开始的时候最好把所有依赖都安装好。可以在<https://github.com/facebook/rocksdb/blob/master/INSTALL.md#supported-platforms>上面找到需要的依赖及安装方法。

## 编译

RocksDB根目录下有`Makefile`文件和`CMakeLists.txt`文件，这意味着可以使用`make`命令或者`cmake`进行编译。在<https://github.com/facebook/rocksdb/blob/master/INSTALL.md#compilation>中作者提到，直接使用`make`命令编译的库是debug版本的，这意味着性能会比较差，不应该在真实工作场景中使用。我在编译时刚开始使用的`make`命令，输入命令之后发现编译的非常慢，而且编译生成的文件体积非常大，在我的电脑上编译出来的文件达到了30多个G，最终因为硬盘满而编译失败。之后我换成了`cmake`命令，发现还是`cmake`好使，编译的速度要快一些而且最后只用了70MB。(TODO: 这可能是因为`cmake`编译的是Release版本，之后有时间好好看一下`Makefile`和`CMakeLists.txt`)

使用`cmake`编译时的命令为：

    mkdir build
    cd build
    cmake ..
    make

{{< alert note >}}
如果你使用**Visual Studio Code**，且安装了CMake Tools扩展，那么在打开项目时该扩展可能会自动进行cmake配置，自动生成build文件但却没有生成`Makefile`，无法编译。这时只需要将build文件删除重新运行上面叙述的几个命令即可。(TODO: 了解一下为什么自动进行的cmake配置不包含Makefile)
{{< \alert >}}

编译好之后你就会在build文件夹下发现`librocksdb.a`和`librocksdb.so`文件。
