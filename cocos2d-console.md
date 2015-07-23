# [【cocos2d-x 从 c++ 到 js】16：使用 cocos2d-console 工具转换脚本为字节码](http://goldlion.blog.51cto.com/4127613/1359012)

从 Cocos2D-X v2.1.4 版本开始，增加了 Cocos2D-console 命令行工具，该工具的其中一个功能是：把 .js 文件转换为 .jsc 文件，该文件是字节码格式，可以提高代码的安全性。

使用这个工具的方式很简单。以引擎自带的 TestJavaScript 项目为例：
首先我们 cd 到 Cocos2D-console 的目录

```
goldliontekiMacBook-Pro:~ goldlion$ cd /Users/goldlion/Documents/developer/cocos2d-x-3.0beta/tools/cocos2d-console/console
```
  
然后可以看到里面有很多 .py 脚本  
cocos2d_jscompile.py  
cocos2d_version.py  
cocos2d.py  
cocos2d_new.py  

其中 cocos2d.py 是我们要使用的主脚本文件。使用命令 ./cocos2d.py jscompile --help 查看编译字节码的命令格式

```
goldliontekiMacBook-Pro:console goldlion$ ./cocos2d.py jscompile --help
Usage: cocos2d.py jscompile -s src_dir -d dst_dir [-c -o COMPRESSED_FILENAME -j COMPILER_CONFIG]
Options:
  -h, --help            show this help message and exit
  -s SRC_DIR_ARR, --src=SRC_DIR_ARR
                        source directory of js files needed to be compiled,
                        supports mutiple source directory
  -d DST_DIR, --dst=DST_DIR
                        destination directory of js bytecode files to be
                        stored
  -c, --use_closure_compiler
                        Whether to use closure compiler to compress all js
                        files into just a big file
  -o COMPRESSED_FILENAME, --output_compressed_filename=COMPRESSED_FILENAME
                        Only available when '-c' option was True
  -j COMPILER_CONFIG, --compiler_config=COMPILER_CONFIG
                        The configuration for closure compiler by using JSON,
                        please refer to compiler_config_sample.json
```

参数非常简单，一个输入目录，一个输出目录，后面加一组可选参数。该工具在遍历 .js 文件时支持文件夹递归访问，在输出 .jsc 文件时支持按照源文件夹的结构全部新建文件夹。易用性还是不错的。

对 TestJavaScript 其中一个文件夹 ExtensionsTest 使用 Cocos2D-console 工具进行加密来测试。输出路径设置为桌面

```
./cocos2d.py jscompile -s /Users/goldlion/Documents/developer/cocos2d-x-3.0beta/samples/Javascript/Shared/tests/ExtensionsTest -d /Users/goldlion/Desktop/ExtensionsTest
```

打开输出的 ExtensionsTest 文件夹看到，所有 .js 都变成了 .jsc，并且体积都大幅度减小。

下面说一下可选参数，可选参数的意思是使用 closure compiler 工具压缩代码为一个文件。  
COMPRESSED_FILENAME 是压缩后的文件名，最好使用 xxx.js，因为工具会自动再后面加个 c  
COMPILER_CONFIG 是压缩时调用的配置文件，需要根据项目需求自己填写，在 bin 目录下有一个做好的缺省例子可以使用，compiler_config_sample.json

我并不建议使用这种做法，因为：  
1. 如果将所有脚本都压缩为一个文件，那么每次更新都要重新下载这个文件，对于一些对省流量要求很高的公司不适合。  
2. 压缩的目的是隐藏文件目录结构，但是这个工具只压缩了脚本部分，对于图片，动画，数据，音频视频等等都是不考虑的。而一般开发的方式需要把所有资源都压缩成一个文件，然后在游戏在线更新时只下载更新档，通过程序将更新档中的文件打入到大文件中。注意这涉及到二进制级别的比较删除以及合并，需要做非常仔细的设计，



