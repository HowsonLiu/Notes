# Qt for Android
## QtCreator 配置（2019.6.7）
[官网教程](https://doc.qt.io/qtcreator/creator-developing-android.html)
### 遇到的坑：
- Qt Creator更至最新
    当时配置好所有Android的路径后，Qt Version仍然报错显示缺失`arm-linux-android-elf-32bit`编译器，后来被我通过更新Qt Creator解决（4.7.1更新至4.9.1）。[相关讨论区链接](https://forum.qt.io/topic/86288/qt-5-10-0-can-t-compile-to-android-arm-linux-android-elf-32bit/6)
- Qt 5.12前，只能用r10e的Android NDK
    这点在[官方教程Getting Started](https://doc.qt.io/qt-5/android-getting-started.html)中提过，当时不留意还查了很久。说不定上面的错误也有可能由于这个引起
- Qt 5.9.2不支持Android armv8
    老子用了两年Qt 5.9.2，用出感情想继续用5.9.2进行开发。谁知连一下手机才发现，手机是armv8架构的，Qt 5.9.2只支持armv7。只能在Qt Maintenance Tools弄个最新版Qt 5.12了
- Qt 5.12不能使用Android NDK r20
    上面刚说完能用最新的，但是使用android-ndk-r20作为ndk路径时，构建就会报错`cannot find -lc++`。换成当时第二个新的android-ndk-r19c就没问题了。希望日后修复吧
- Qt 5.12不能安装Android SDK Build-tools 29
    这个最坑了，SDK Manager中连安装都不能安装，否则构建会报错`Execution failed for task ':compileDebugAidl'.`并退出代码14。幸亏这位[老哥](https://forum.qt.io/topic/101322/what-s-the-problem-android-compile-error/3)，不然都准备放弃了。也希望以后修复吧
除了这些问题外，到目前还没试过真机调试，里面估计也有一堆坑。但是无论如何，在电脑build出来的apk能在手机上跑，就算ok了。反正总的来说，出现问题都是因为各种各样的版本不匹配造成的，能成功打包成apk已经算是上天的恩赐了。