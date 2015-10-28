---
layout: post
title: 通过CMake支持Qt5的外部资源文件
category : CMake
tags : [CMake，Qt，Qt5]
---
{% include JB/setup %}

最近因为某些原因需要使用到Qt5的外部资源文件，之前一直使用的都是编译到程序内部的资源文件。

在CMake中使用内部的资源文件非常方便，因为Qt5提供了CMake的实现：

    qt5_add_resources(outfiles inputfile ... OPTIONS ...)
	
最后只要将`outfiles`传递给`target`就可以了。

同事找到github上也有个兄弟有这个需求，他自己做了一个CMake函数用于生成`rcc`文件，并且给出了为什么要自己实现的理由：
> Even if forcing this function to pass the -binary option to rcc tool, which is possible, the result is that the linker tries to link also these binary resource files, which obviously is not a good idea.

刚开始没多想，就直接把他的实现直接拿过来用了，在写这篇文档的时候仔细想了想，发现他说的这句话的后半句是不对的，如果你没有将`outfiles`传递给`target`，这些生成的文件根本就不会参与编译连接。之后仔细的看了下`qt5_add_resources`的实现，发现唯一的问题是`outfiles`默认是命名为`qrc_*.cpp`，不过这是个小问题，因为既然是外部资源文件，就一定会被拷贝到可执行文件相对的某个目录或是打包到安装文件中，所以在执行这个操作时重命名就好了。


### 参考资料
[1] [CMake support for Qt5's external resources](http://anadoxin.org/blog/cmake-support-for-qt5s-external-resources.html)

