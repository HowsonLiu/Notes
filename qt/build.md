# Windows下Qt源码编译
源码获得的两种方式：
- Qt Maintenance Tool  
    下载对应版本源码，可以直接编译
- github [https://github.com/qt/qt5.git](https://github.com/qt/qt5.git)  
    下载之后需要安装PERL([https://www.activestate.com/products/perl/downloads/](https://www.activestate.com/products/perl/downloads/))才能编译。建议使用这种方法，因为编译过程中会产生很多临时文件影响下次编译，使用git比较好恢复  

## 1. 打开VS命令行
只能通过x64或者x86的Visual Studio的命令行Command Prompt选择目标架构是x64或者x86  
  
例如使用`x64 Native Tools Command Prompt for VS 2017`命令行程序编出来的等于在Maintenance Tool里下载的msvc2017_64

## 2. 设置环境变量
```
SET _ROOT=C:\Qt\Qt-5
SET PATH=%_ROOT%\qtbase\bin;%_ROOT%\gnuwin32\bin;%PATH%
SET _ROOT=
```
## 3. Configure
这是我自己用的配置，跳过挺多的，打包必备  
`configure -prefix d:\Qt\5.12.9\msvc2017_64_static -release -opensource -static -opengl desktop -platform win32-msvc -c++std c++11 -skip qtsensors -skip qtwebengine -skip qtgamepad -nomake examples -nomake tests -mp -skip qtlocation -skip qtserialbus -confirm-license`
- `-prefix` 指定输出路径
- `-release` `-debug-and-release` `-debug` 选择release或者debug版
- `-static` 静态链接，不含dll
- `-static-runtime` 运行时库的静态链接，等同于编译选项中的`/MT`和`/MTD`

## 3. nmake
## 4. nmake install
  
参考：  
[https://doc.qt.io/qt-5/windows-building.html](https://doc.qt.io/qt-5/windows-building.html)