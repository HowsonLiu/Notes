# GET PARAMETER
发送端请求获取音量，信令是**GET PARAMETER**
```
GET_PARAMETER rtsp://192.168.137.1/10555496157292350542 RTSP/1.0
Content-Length: 8
Content-Type: text/parameters
CSeq: 8
DACP-ID: 1FF65A04A9252F60
Active-Remote: 2814835199
User-Agent: AirPlay/409.16

volume

```
接收端回应音量
```
RTSP/1.0 200 OK
CSeq: 8
Server: AirTunes/220.68
Content-Type: text/parameters
Content-Length: 13

volume: 0.0

```
目前**GET PARAMETER**信令支持的参数只有`volume`

# RECORD
发送端请求启动音频流传输。信令是**RECORD**
```
RECORD rtsp://192.168.137.1/10555496157292350542 RTSP/1.0
CSeq: 9
DACP-ID: 1FF65A04A9252F60
Active-Remote: 2814835199
User-Agent: AirPlay/409.16


```
接收端回应**RECORD**两个字段
- Audio-Latency：音频延迟
- Audio-Jack-Status: 音频插孔状态
```
RTSP/1.0 200 OK
CSeq: 9
Server: AirTunes/220.68
Audio-Latency: 11025
Audio-Jack-Status: connected; type=analog


```
# SET PARAMETER
发送端请求设置音量，信令是**SET PARAMETER**
```
SET_PARAMETER rtsp://192.168.137.1/10555496157292350542 RTSP/1.0
Content-Length: 20
Content-Type: text/parameters
CSeq: 10
DACP-ID: 1FF65A04A9252F60
Active-Remote: 2814835199
User-Agent: AirPlay/409.16

volume: -20.000000

```
音量值的范围是**0.0 ~ -144.0**

接收端返回一个空包
```
RTSP/1.0 200 OK
CSeq: 10
Server: AirTunes/220.68


```
# SETUP2
发送端发送第二个**SETUP**包，格式依然为bplist
```
SETUP rtsp://192.168.137.1/10555496157292350542 RTSP/1.0
Content-Length: 188
Content-Type: application/x-apple-binary-plist
CSeq: 11
DACP-ID: 1FF65A04A9252F60
Active-Remote: 2814835199
User-Agent: AirPlay/409.16

bplist00ÑWstreams¡ÓTtype]timestampInfo_streamConnectionIDn¥	Ñ
TnameUSubSuÑ

UBePxTÑ
UAfPxTÑ
UBefEnÑ
UEmEnc!Ûô#àH|!/DFLOTZ]cfloux~
```
解析后如下
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>streams</key>
	<array>
		<dict>
			<key>type</key>
			<integer>110</integer>
			<key>timestampInfo</key>
			<array>
				<dict>
					<key>name</key>
					<string>SubSu</string>
				</dict>
				<dict>
					<key>name</key>
					<string>BePxT</string>
				</dict>
				<dict>
					<key>name</key>
					<string>AfPxT</string>
				</dict>
				<dict>
					<key>name</key>
					<string>BefEn</string>
				</dict>
				<dict>
					<key>name</key>
					<string>EmEnc</string>
				</dict>
			</array>
			<key>streamConnectionID</key>
			<integer>3646029618780522008</integer>
		</dict>
	</array>
</dict>
</plist>
```
## raop初始化
**SETUP 2**包主要初始化了raop的
- **视频缓冲区**
    [前面](airplay_handshake_2.md#rtp初始化)说到，**SETUP 1**记录了16位`fairplay_ekey`以及32位`ecdh_secret`，现在结合包中`streamConnectionID`可以完成初始化了
    1. sha512先后摘要16位`fairplay_ekey`以及32位`ecdh_secret`，得到64位签名`sign`
    2. aes初向量初始化：
        1. 拼接字符串`"AirPlayStreamIV" + streamConnectionID`
        2. sha512先后摘要字符串以及前16位`sign`
        3. sha512得到64位签名`vsign_aesiv`，取前16位作为aes初向量`vaesiv`
    3. aes密钥初始化：
        1. 拼接字符串`"AirPlayStreamKey" + streamConnectionID`
        2. sha512先后摘要字符串以及前16位`sign`
        3. sha512得到64位签名`vsign_aeskey`，取前16位作为aes初向量`vaeskey`
- **镜像socket环境**
    初始化环境并开始监听相关端口，端口由系统动态分配
- **运输层协议**
    检查http头部是否存在Transport字段，值是否取值RTP/AVP/TCP，如果没有就使用udp协议，本例使用udp
## 接收端回应
```
RTSP/1.0 200 OK
CSeq: 11
Server: AirTunes/220.68
Content-Type: application/x-apple-binary-plist
Content-Length: 120

bplist00ÓYeventPortZtimingPortÞ
Wstreams¡Ò
	XdataPortTtypen'*249BEJL
```
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>eventPort</key>
	<integer>5001</integer>
	<key>timingPort</key>
	<integer>58122</integer>
	<key>streams</key>
	<array>
		<dict>
			<key>dataPort</key>
			<integer>9278</integer>
			<key>type</key>
			<integer>110</integer>
		</dict>
	</array>
</dict>
</plist>
```
回应开启的相关端口，格式使用bplist

镜像连接握手流程已全部走通，下面就是镜像传输流程