---
title: 在 Mac 终端编译 OpenCV 程序
date: 2016-05-30 15:16:43
tags:
top: 2
---

我通常在 Mac OS 的 XCode 上编写 C++ 程序（调用 OpenCV 库），但同事使用 Windows 上的 Visual Studio。然而 XCode 的 clang++ 与 Visual Studio 自带的 C++ 编译器在面对同一份代码时会得出不同的结果，有时候 XCode 能编译通过并顺利运行的程序，在 VS 里会报编译错误。

于是希望找到一种兼容的解决方案，能使 Mac OS 上编写的 C++ OpenCV 程序能不经修改地在 Windows 平台上编译通过。打算尝试 eclipse，并配置兼容性好的 C++ 编译器。但在此之前，先采用更底层的 make 的方式配置 OpenCV 并编译一段简单的程序，以熟悉过程。

<!--more-->

## Terminal 安装 OpenCV

参考文章：[1] [在Mac下使用OpenCV](http://wuzhaoxi1992511.blog.163.com/blog/static/18375811820132213544889/)；[2] [Fix tautologies in calibfilter.cpp](https://github.com/Itseez/opencv/commit/35f96d6da76099d80180439c857a4abe5cb17966)。

从官网下载 OpenCV（我下载的是 2.4.9 版本，貌似用的人最多）并解压，从 Terminal 进入 OpenCV 包解压后的文件夹，然后 make：

```bash
# make a separate dictionary for building
mkdir build
cd build
cmake -G "Unix Makefiles" ..
# make OpenCV
make -j8
sudo make install
```

如果安装顺利的话，lib 文件会放到 /usr/local/lib 文件夹下，头文件会放在 /user/local/include 文件夹下。

然而，在执行 make -j8 的时候会报错：

```bash
make: *** [all] Error 2
```

参考文章 [Fix tautologies in calibfilter.cpp](https://github.com/Itseez/opencv/commit/35f96d6da76099d80180439c857a4abe5cb17966) 修改 modules/legacy/src/calibfilter.cpp 文件后即可 make 成功。直接把网页中的代码复制替换 calibfilter.cpp 中的内容，然后 make -j8，可以顺利通过。

## Terminal 编译 OpenCV 程序

新建 cvtest 文件夹，在里面编写测试程序 cvtext.cpp:

```cpp
#include <cv.h>
#include <highgui.h>
using namespace cv;
int main( int argc, char** argv )
{
  Mat image;
  image = imread( argv[1], 1 );

  if( argc != 2 || !image.data )
    {
      printf( "No image data \n" );
      return -1;
    }

  namedWindow( "Display Image", CV_WINDOW_AUTOSIZE );
  imshow( "Display Image", image );

  waitKey(0);

  return 0;
}
```

新建 CMakeList.txt 文件于 cvtest 文件夹中，内容如下：

```cpp
project( cvtest )
find_package( OpenCV REQUIRED )
add_executable( cvtest cvtest )
target_link_libraries( cvtest ${OpenCV_LIBS} )
```

从终端进入 cvtest 文件夹，执行：

```bash
cmake .
make
```

会看到生成了名为 cvtest 的可执行文件。把图片 image.bmp 放在 cvtest 文件夹中，终端执行：

```bash
./cvtest image.bmp
```

会出现一个窗口显示图像。
