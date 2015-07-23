# [【cocos2d-x 从 c++ 到 js】07：cxx-generator JS 绑定工具](http://goldlion.blog.51cto.com/4127613/1137291)

## 第一部分：配置安装环境

cxx-generator 是由 Zynga 工程师贡献的 C++ 代码绑定到 js 工具。用于将 cocos2d－x 的 c＋＋ 代码，生成相应的 js 绑定代码（由 c＋＋ 写成），然后将这些函数注册到 spidermonkey 的解释器中。通过将 js 代码映射成 c＋＋ 代码，就可以使用相应的 js 接口了。
所需要的环境
mac os x 系统   
python2.7  
py-yaml  
cheetah (for target language templates)  
libclang, from clang 3.1  

前三个可以通过 macports 自动安装  
macports 下载地址  
<http://www.macports.org/install.php>  
注意选择适合你的系统版本，另外该页也注明了安装中常见的系统问题，一共四条。  
在安装 macports 时，有可能会卡在最后一分钟，那么需要重启后断网安装即可。  
 
在终端上运行此命令，安装前三个软件  
sudo port install python27 py27-yaml py27-cheetah  
安装对网络有一定要求，部分地区可能要自备梯子  

下载clang
 
<http://llvm.org/releases/3.1/clang+llvm-3.1-x86_64-apple-darwin11.tar.gz>

下载NDK  
绑定例子中，用到了部分 c＋＋ 标准库接口，所以需要提供相应代码实现，工具中，采用 ndk 实现。不太明白为什么没有直接用 xcode 中的标准库。
 
<http://dl.google.com/android/ndk/android-ndk-r8d-darwin-x86.tar.bz2>

## 第二步，生成绑定代码

复制 userconf.ini.sample 和 user.cfg.sample 并去掉 sample 后缀
 
添加自己的路径，我的是多系统所以路径有点特别
 
／／user.cfg  
PYTHON_BIN=/opt/local/bin/python2.7  
 
／／userconf.ini   
[DEFAULT]  
androidndkdir=/Volumes/data/Mac_OS_X/android-ndk-r8b
clangllvmdir=/Volumes/data/Mac_OS_X/clang+llvm-3.1-x86_64-apple-darwin11
cxxgeneratordir=/Volumes/data/Workspace/cocos2d-2.1beta3-x-2.1.0/tools/cxx-generator

最后，由终端运行

sudo ./test.sh
 
生成 simple_test_bindings 文件夹，下面就是绑定好的 c＋＋ 代码了。

## 第三步，集成测试
 
懒省事直接拿 TestJavaScript 例子开刀，倒入两个文件夹 simple_test 和 simple_test_bindings

在 AppDelegate.cpp 中，倒入头文件

```
#include "autogentestbindings.hpp"
```
并注册

sc->addRegisterCallback(register_all_autogentestbindings);

在 tests-boot-jsb.js 中，添加测试代码

```
var myClass=new ts.SimpleNativeClass();
var myStr=myClass.returnsACString();
cc.log(myStr); 
```

控制台输出
 
```
this is a c-string
```

参考文献  
<https://github.com/funkaster/cxx-generator>  
<http://www.macports.org/install.php>  
<http://cn.cocos2d-x.org/bbs/forum.php?mod=viewthread&tid=10226&extra=page%3D1>

 