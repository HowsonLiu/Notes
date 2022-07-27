#NSIS美化
## 设置Unicode
- 在nsi文件中加入`Unicode true`
- VS工程中lib使用 pluginapi-x86-unicode.lib
- VS工程中生成的dll放入NSIS安装目录下Plugins\x86-unicode\下
- 使用makensisw生成？

## Something
- `EXDLL_INIT`函数只需要在第一次调用的函数里初始化