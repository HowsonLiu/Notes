# PC微信逆向：微信Log研究（二）
经过上文的研究，虽然我们不能解密整个log目录，但是我们可以借助hook的手段，实时展示
```c++
std::string logPath = "Log"; //use your log path
std::string pubKey = ""; //use you pubkey for log encrypt

#if _DEBUG
xlogger_SetLevel(kLevelDebug);
appender_set_console_log(true);
#else
xlogger_SetLevel(kLevelInfo);
appender_set_console_log(false);
#endif
appender_open(kAppednerAsync, logPath.c_str(), "Sample", pubKey.c_str());
```