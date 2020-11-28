# 镜像音频传输握手
镜像传输在没有音频流传输的时候，并不会进行音频相关的握手以及传输。音频握手跟挥手的时机似乎跟APP有关系。比如说
- 库乐队
	打开应用时握手，关闭应用时挥手
- 相册
	播放视频时握手，停止播放视频后几秒挥手

握手流程
	1. **SETUP 3**
	2. **SET PARAMETER**
	3. **FLUSH**

# 音频SETUP 3
```
þ\hßE×0%¦æ¸E@@¥fÀ¨&À¨Â~HÞouD·
 ÜV>"SETUP rtsp://192.168.137.1/3201618686068539433 RTSP/1.0
Content-Length: 265
Content-Type: application/x-apple-binary-plist
CSeq: 18
DACP-ID: D13001AA7498695C
Active-Remote: 4109011333
User-Agent: AirPlay/420.45

bplist00ÑWstreams¡Ü	

[usingScreen[audioFormat_supportsDynamicStreamID^redundantAudioYaudioModeRctSspfZlatencyMaxTtype[controlPortWisMediaZlatencyMin		Wdefaultà¤`ÚI	.:F`oy|¤¯°µ¶¸ÀÂÅÈÊÍÎ
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>streams</key>
	<array>
		<dict>
			<key>usingScreen</key>
			<true/>
			<key>audioFormat</key>
			<integer>16777216</integer>
			<key>supportsDynamicStreamID</key>
			<true/>
			<key>redundantAudio</key>
			<integer>2</integer>
			<key>audioMode</key>
			<string>default</string>
			<key>ct</key>
			<integer>8</integer>
			<key>spf</key>
			<integer>480</integer>
			<key>latencyMax</key>
			<integer>3748</integer>
			<key>type</key>
			<integer>96</integer>
			<key>controlPort</key>
			<integer>59155</integer>
			<key>isMedia</key>
			<true/>
			<key>latencyMin</key>
			<integer>3748</integer>
		</dict>
	</array>
</dict>
</plist>
```
报文属于**SETUP**类型

接收端在收到报文后，进行相关socket的初始化，接着把时间同步端口、数据端口、控制端口返回给发送端
```
0%¦æ¸þ\hßE×E)H=@À¨À¨&Âou~HÃFr
>ã ÜVRTSP/1.0 200 OK
CSeq: 18
Server: AirTunes/220.68
Content-Type: application/x-apple-binary-plist
Content-Length: 122

bplist00ÒZtimingPortÐ<Wstreams¡Ó
	XdataPortÐ=Ttype`[controlPortÐ;
#%,58=?KN
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>timingPort</key>
	<integer>57344</integer>
	<key>streams</key>
	<array>
		<dict>
			<key>dataPort</key>
			<integer>57345</integer>
			<key>type</key>
			<integer>96</integer>
			<key>controlPort</key>
			<integer>57343</integer>
		</dict>
	</array>
</dict>
</plist>

```
发送端后续使用三个端口进行音频传输
# SET PARAMETER & FLUSH
在正式传输音频前，最后同步一下音量以及刷一下音频缓冲区