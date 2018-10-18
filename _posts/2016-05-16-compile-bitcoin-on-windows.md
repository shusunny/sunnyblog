---
layout: post
title: "在windows平台编译比特币客户端"
author: "sun"
categories: Blockchain
tags: [bitcoin, C++, windows]
---

本来不打算在windows上编译bitcoin的，但是上级指示做事要彻底，只好乖乖听话。
虽然比特币程序是跨平台的，在windows上编译显然不如在linux上那么简单快捷，但是经过我长时间的摸索，发现其实也没那么麻烦，只是有很多库需要下载。之前觉得麻烦是因为mingw无法复制粘贴编译需要用的代码。后来发现只需要用mingw shell去运行包括代码的脚本程序就好。
其实也是比较方便的，只需动动手指，复制粘贴就好。

> 本文参考 http://blog.sina.com.cn/s/blog_5922b3960101s5j9.html

### 一、安装MINGW,MSYS
msys是一个在windows平台模拟shell的程序。它可以让我们在windows平台下模拟我们的ubuntu shell进行操作。

首先去
http://sourceforge.net/projects/mingw/files/Installer/mingw-get-setup.exe/download
下载并安装程序。然后通过安装管理程序，选择安装以下内容：
From MinGW installation manager -> All packages -> MSYS
选中以下package

```
msys-base class:bin
msys-autoconf class:bin
msys-automake class:bin
msys-libtool class:bin
```

选完后在Installation里点 apply changes开始安装。他会自动下载安装好。
需要注意的是，确保不要安装**msys-gcc和msys-w32api** ，因为这两个包和我们的编译系统发生冲突。
很多人出现一些莫名其妙的问题，就是因为这两个包。

---

然后安装 [MinGW-builds](https://sourceforge.net/projects/mingw-w64/files/Toolchains%20targetting%20Win32/Personal%20Builds/mingw-builds/4.8.2/threads-posix/dwarf/i686-4.8.2-release-posix-dwarf-rt_v3-rev3.7z/download)
下载并解压缩 i686-4.8.2-release-posix-dwarf-rt_v3-rev3.7z 到C盘根目录 C:\
注意我的目录结构，请尽量和我的一样。
然后设置PATH环境变量，将C:\mingw32\bin;添加到第一个

**在我的电脑上右键——属性——高级系统设置——高级——环境变量——系统变量里
找到PATH——编辑——新建——填入C:\mingw32\bin。然后把所有窗口点确定以保存。**

在命令行模式下输入 gcc -v 会得到以下内容

```
c:\gcc -v
Using built-in specs.
COLLECT_GCC=c:\mingw32\bin\gcc.exe
COLLECT_LTO_WRAPPER=c:/mingw32/bin/../libexec/gcc/i686-w64-mingw32/4.8.2/lto-wrapper.exe
Target: i686-w64-mingw32
Configured with: ../../../src/gcc-4.8.2/configure –host=i686-w64-mingw32 –build=i686-w64-mingw32 –target=i686-w64-mingw32 –prefix=/mingw32 –with-sysroot=/c/mingw482/i686-482-posix-dwarf-rt_v3-rev3/mingw32 –with-gxx-include-dir=/mingw32/i686-w64-mingw32/include/c++ –enable-shared –enable-static –disable-multilib –enable-languages=ada,c,c++,fortran,objc,obj-c++,lto –enable-libstdcxx-time=yes –enable-threads=posix –enable-libgomp –enable-libatomic –enable-lto –enable-graphite –enable-checking=release –enable-fully-dynamic-string –enable-version-specific-runtime-libs –disable-sjlj-exceptions –with-dwarf2 –disable-isl-version-check –disable-cloog-version-check –disable-libstdcxx-pch –disable-libstdcxx-debug –enable-bootstrap –disable-rpath –disable-win32-registry –disable-nls –disable-werror –disable-symvers –with-gnu-as –with-gnu-ld –with-arch=i686 –with-tune=generic –with-libiconv –with-system-zlib –with-gmp=/c/mingw482/prerequisites/i686-w64-mingw32-static –with-mpfr=/c/mingw482/prerequisites/i686-w64-mingw32-static –with-mpc=/c/mingw482/prerequisites/i686-w64-mingw32-static –with-isl=/c/mingw482/prerequisites/i686-w64-mingw32-static –with-cloog=/c/mingw482/prerequisites/i686-w64-mingw32-static –enable-cloog-backend=isl –with-pkgversion=’i686-posix-dwarf-rev3, Built by MinGW-W64 project’ –with-bugurl=http://sourceforge.net/projects/mingw-w64 CFLAGS=’-O2 -pipe -I/c/mingw482/i686-482-posix-dwarf-rt_v3-rev3/mingw32/opt/include -I/c/mingw482/prerequisites/i686-zlib-static/include -I/c/mingw482/prerequisites/i686-w64-mingw32-static/include’ CXXFLAGS=’-O2 -pipe -I/c/mingw482/i686-482-posix-dwarf-rt_v3-rev3/mingw32/opt/include -I/c/mingw482/prerequisites/i686-zlib-static/include -I/c/mingw482/prerequisites/i686-w64-mingw32-static/include’ CPPFLAGS= LDFLAGS=’-pipe -L/c/mingw482/i686-482-posix-dwarf-rt_v3-rev3/mingw32/opt/lib -L/c/mingw482/prerequisites/i686-zlib-static/lib -L/c/mingw482/prerequisites/i686-w64-mingw32-static/lib -Wl,–large-address-aware’
Thread model: posix
gcc version 4.8.2 (i686-posix-dwarf-rev3, Built by MinGW-W64 project)
```

至此，你的开发环境已经搭建好了，很简单吧

### 二、下载bitcoin引用的外部库
**我们把它们全部放在 C:\deps目录下**

2.1 安装OpenSSL下载：http://www.openssl.org/source/openssl-1.0.1g.tar.gz
进入启动 MinGw shell 比如目录：(C:\MinGW\msys\1.0\msys.bat)运行这个msys.bat，就会启动一个shell环境，提示符是$
这里为了以后文件编译的方便，我们在deps目录中新建一个compile.sh文件。用记事本打开，然后将下面的代码复制进去

```cpp
cd /c/deps/
tar xvfz openssl-1.0.1g.tar.gz
cd openssl-1.0.1g
Configure no-shared no-dso mingw
make
```
然后在msys的shell环境中进入deps文件夹，输入`./compile.sh`命令这样就可以直接编译了,这样做省时省力，否则后面要输入大量的代码，很容易输错，而且脚本也便于我们修改。
等待几分钟后，就把openssl编译好了。

2.2 下载Berkeley DB 访问：http://download.oracle.com/berkeley-db/db-4.8.30.NC.tar.gz
我们推荐使用4.8版本
用同样将刚才的脚本代码替换成下面的，然后在msys shell环境下运行

```cpp
cd /c/deps/
tar xvfz db-4.8.30.NC.tar.gz
cd db-4.8.30.NC/build_unix
../dist/configure --enable-mingw --enable-cxx --disable-shared --disable-replication
make
```

2.3 安装Boost，下载地址： https://sourceforge.net/projects/boost/files/boost/1.55.0/boost_1_55_0.tar.gz/download
脚本代码

```cpp
cd /c/deps/
tar xvzf boost_1_55_0.tar.gz
cd boost_1_55_0
bootstrap.bat mingw
b2 --build-type=complete --with-chrono --with-filesystem --with-program_options --with-system --with-thread toolset=gcc variant=release link=static threading=multi runtime-link=static stage
```

2.4 安装Miniupnpc 下载地址： http://miniupnp.free.fr/files/download.php?file=miniupnpc-1.9.tar.gz

```cpp
cd /c/deps/
tar xvzf miniupnpc-1.9.tar.gz
cd miniupnpc-1.9
mingw32-make -f Makefile.mingw init upnpc-static
```

2.5 下载 protoc 和 libprotobuf: http://protobuf.googlecode.com/files/protobuf-2.5.0.tar.gz

```cpp
cd /c/deps/
tar xvzf protobuf-2.5.0.tar.gz
cd protobuf-2.5.0
configure --disable-shared
make
```

2.6 qrencode:
下载地址： http://prdownloads.sourceforge.net/libpng/libpng-1.6.10.tar.gz?download
```cpp
cd /c/deps/
tar xvzf libpng-1.6.10.tar.gz
cd libpng-1.6.10
configure --disable-shared
make
```
下载 http://fukuchi.org/works/qrencode/qrencode-3.4.3.tar.gz:
```cpp
cd /c/deps/qrencode-3.4.3
LIBS="../libpng-1.6.10/.libs/libpng16.a ../../mingw32/i686-w64-mingw32/lib/libz.a" \
png_CFLAGS="-I../libpng-1.6.10" \
png_LIBS="-L../libpng-1.6.10/.libs" \
configure --enable-static --disable-shared --without-tools
make
```

2.7 安装 Qt 5 库
下载和解压缩
http://download.qt-project.org/official_releases/qt/5.2/5.2.1/submodules/qtbase-opensource-src-5.2.1.7z
http://download.qt-project.org/official_releases/qt/5.2/5.2.1/submodules/qttools-opensource-src-5.2.1.7z
在 windows命令行输入：

```ruby
set INCLUDE=C:\deps\libpng-1.6.10;C:\deps\openssl-1.0.1g\include
set LIB=C:\deps\libpng-1.6.10\.libs;C:\deps\openssl-1.0.1g

cd C:\Qt\5.2.1
configure.bat -release -opensource -confirm-license -static -make libs -no-sql-sqlite -no-opengl -system-zlib -qt-pcre -no-icu -no-gif -system-libpng -no-libjpeg -no-freetype -no-angle -no-vcproj -openssl-linked -no-dbus -no-audio-backend -no-wmf-backend -no-qml-debug

mingw32-make

set PATH=%PATH%;C:\Qt\5.2.1\bin

cd C:\Qt\qttools-opensource-src-5.2.1
qmake qttools.pro
mingw32-make
```

### 三、下载Bitcoin 0.9.1
> 我只测试过0.10._ 版本和0.9._版本。最新的版本由于文件变化亲测无效。想编译最新版本的看看哪里报错并逐步排除。
地址： https://github.com/bitcoin/bitcoin/archive/v0.9.1.zip

将compile.sh脚本换成下面的，在msys里运行:

```cpp
cp /c/deps/libpng-1.6.10/.libs/libpng16.a /c/deps/libpng-1.6.10/.libs/libpng.a

cd /c/bitcoin-0.9.1

./autogen.sh

CPPFLAGS="-I/c/deps/boost_1_55_0 \
-I/c/deps/db-4.8.30.NC/build_unix \
-I/c/deps/openssl-1.0.1g/include \
-I/c/deps \
-I/c/deps/protobuf-2.5.0/src \
-I/c/deps/libpng-1.6.10 \
-I/c/deps/qrencode-3.4.3" \
LDFLAGS="-L/c/deps/boost_1_55_0/stage/lib \
-L/c/deps/db-4.8.30.NC/build_unix \
-L/c/deps/openssl-1.0.1g \
-L/c/deps/miniupnpc \
-L/c/deps/protobuf-2.5.0/src/.libs \
-L/c/deps/libpng-1.6.10/.libs \
-L/c/deps/qrencode-3.4.3/.libs" \
./configure \
--disable-upnp-default \
--disable-tests \
--with-qt-incdir=/c/Qt/5.2.1/include \
--with-qt-libdir=/c/Qt/5.2.1/lib \
--with-qt-bindir=/c/Qt/5.2.1/bin \
--with-qt-plugindir=/c/Qt/5.2.1/plugins \
--with-boost-system=mgw48-mt-s-1_55 \
--with-boost-filesystem=mgw48-mt-s-1_55 \
--with-boost-program-options=mgw48-mt-s-1_55 \
--with-boost-thread=mgw48-mt-s-1_55 \
--with-boost-chrono=mgw48-mt-s-1_55 \
--with-protoc-bindir=/c/deps/protobuf-2.5.0/src

make

strip src/bitcoin-cli.exe
strip src/bitcoind.exe
strip src/qt/bitcoin-qt.exe
```

这样，你就能得到编译好的 bitcoin-cli.exe和bitcoind.exe ,bitcoin-qt.exe(windows QT图形界面的钱包软件)。

是不是还挺简单的？毕竟Ctrl+C/V就好了。^-^
