# airplay协议抓包分析
## 协议
### SSDP (Simple Service Discovery Protocol)
简单服务发现协议提供了在局部网络里面发现设备的机制。控制点（也就是接受服务的客户端）可以通过使用简单服务发现协议，根据自己的需要查询在自己所在的局部网络里面提供特定服务的设备。设备（也就是提供服务的服务器端）也可以通过使用简单服务发现协议，向自己所在的局部网络里面的控制点宣告它的存在。
### ROAP
远程音频输出协议
## 地址
- 投屏地址 
> ipv4 192.168.137.242 \
ipv6 fe80::d870:4d09:4a76:6141
- ipad地址 
> ipv4 192.168.137.248
> mac 10:30:25:a6:e6:b8
- 路由地址 
> ipv4 192.168.137.1
> ipv6 fe80::ad3f::e6df::4e41::2d21
> mac ac:2b:6e:b8:53:99
- i4 相关
    - post log地址 60.9.0.254
    - 302 地址 114.55.188.99
    - 特效 地址 111.161.120.233
## Mirror流程
### 发布Bonjour设备服务

### 相关参数
| 参数 | 含义 |
|:--:|:--:|
| CSeq | int类型，表示请求序号 |
### TCP三次握手之后
```
¬+n¸S0%¦æ¸EB@@¤zÀ¨øÀ¨òÃïæêÚe)µ®PÅdGET /info RTSP/1.0
X-Apple-ProtocolVersion: 1
Content-Length: 70
Content-Type: application/x-apple-binary-plist
CSeq: 0
DACP-ID: 9A8670A3D720AAF6
Active-Remote: 1716416520
User-Agent: AirPlay/406.31.21

bplist00ÑYqualifier¡ZtxtAirPlay"
```
客户端传来第一个plist
> 
```
0%¦æ¸¬+n¸SEä@É?À¨òÀ¨øïæÃµ®êÚfCPÿ×RTSP/1.0 200 OK
Server: AirTunes/150.33
CSeq: 0
Date: Thu Oct 31 02:50:40 2019
Content-Length: 850

bplist00ß	(*


 )+RpkO,AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA= TnameXApple TVRvv[statusFlagsD_keepAliveLowPower_keepAliveSendStatsAsBodyRpi_$b08f5a79-db29-4384-b456-a4784d9e6055]sourceVersionV220.68XdeviceID_54:ee:75:8f:b2:1aZmacAddressUmodelZAppleTV5,3\audioFormats¢Ó_audioInputFormatsÿÿü_audioOutputFormatsTtypedÓe^audioLatencies¢!'Ô"$&#%%YaudioTypeWdefault_inputLatencyMicros_outputLatencyMicrosÔ"$&#%%XfeaturesZÿ÷Xdisplays¡,Ü-/1234568(;<.0%%%0.79:%=VheightFUwidth8Xrotation]widthPhysical^heightPhysical[widthPixels\heightPixels[refreshRate<VmaxFPS[overscannedTuuid_$e5f7a68d-7b0f-4305-984b-974f677a150b),\ajmo{}®±Øæíö
&36=QVkpry{ ¨½¾ÔÝæïøú#&/=LXeqsz|~>¶
```
返回plist，转换成xml如下
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
	<dict>
		<key>audioFormats</key>
		<array>
			<dict>
				<key>audioInputFormats</key>
				<integer>67108860</integer>
				<key>audioOutputFormats</key>
				<integer>67108860</integer>
				<key>type</key>
				<integer>100</integer>
			</dict>
			<dict>
				<key>audioInputFormats</key>
				<integer>67108860</integer>
				<key>audioOutputFormats</key>
				<integer>67108860</integer>
				<key>type</key>
				<integer>101</integer>
			</dict>
		</array>
		<key>audioLatencies</key>
		<array>
			<dict>
				<key>audioType</key>
				<string>default</string>
				<key>inputLatencyMicros</key>
				<false />
				<key>outputLatencyMicros</key>
				<false />
				<key>type</key>
				<integer>100</integer>
			</dict>
			<dict>
				<key>audioType</key>
				<string>default</string>
				<key>inputLatencyMicros</key>
				<false />
				<key>outputLatencyMicros</key>
				<false />
				<key>type</key>
				<integer>101</integer>
			</dict>
		</array>
		<key>deviceID</key>
		<string>80:e8:2c:1e:d5:a3</string>
		<key>displays</key>
		<array>
			<dict>
				<key>features</key>
				<integer>14</integer>
				<key>height</key>
				<integer>1350</integer>
				<key>heightPhysical</key>
				<false />
				<key>heightPixels</key>
				<integer>1350</integer>
				<key>maxFPS</key>
				<integer>30</integer>
				<key>overscanned</key>
				<false />
				<key>refreshRate</key>
				<integer>60</integer>
				<key>rotation</key>
				<false />
				<key>uuid</key>
				<string>e5f7a68d-7b0f-4305-984b-974f677a150b</string>
				<key>width</key>
				<integer>1080</integer>
				<key>widthPhysical</key>
				<false />
				<key>widthPixels</key>
				<integer>1080</integer>
			</dict>
		</array>
		<key>features</key>
		<integer>130367356919</integer>
		<key>keepAliveLowPower</key>
		<integer>1</integer>
		<key>keepAliveSendStatsAsBody</key>
		<integer>1</integer>
		<key>macAddress</key>
		<string>80:e8:2c:1e:d5:a3</string>
		<key>model</key>
		<string>AppleTV5,3</string>
		<key>name</key>
		<string>Apple TV</string>
		<key>pi</key>
		<string>b08f5a79-db29-4384-b456-a4784d9e6055</string>
		<key>pk</key>
		<data>QUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQT0=</data>
		<key>sourceVersion</key>
		<string>220.68</string>
		<key>statusFlags</key>
		<integer>68</integer>
		<key>vv</key>
		<integer>2</integer>
	</dict>
</plist>
```
```
¬+n¸S0%¦æ¸Eÿ@@¤½À¨øÀ¨òÃïæêÚfCµ jPTÅPOST /pair-setup RTSP/1.0
Content-Length: 32
Content-Type: application/octet-stream
CSeq: 1
DACP-ID: 9A8670A3D720AAF6
Active-Remote: 1716416520
User-Agent: AirPlay/406.31.21

f8} áµ«CÂA®,/øØ×+zè;ÀePöB
```
```
0%¦æ¸¬+n¸SEÙ@ÌIÀ¨òÀ¨øïæÃµ jêÚgPþº<RTSP/1.0 200 OK
Content-Type: application/octet-stream
Server: AirTunes/150.33
CSeq: 1
Date: Thu Oct 31 02:50:40 2019
Content-Length: 32

cB0Ñr,Ê5¿?ùÐWL6áé
ý
sZ#Û
```
```
¬+n¸S0%¦æ¸ET@@¤hÀ¨øÀ¨òÃïæêÚgµ¡P³POST /pair-verify RTSP/1.0
X-Apple-PD: 1
X-Apple-AbsoluteTime: 594183040
Content-Length: 68
Content-Type: application/octet-stream
CSeq: 2
DACP-ID: 9A8670A3D720AAF6
Active-Remote: 1716416520
User-Agent: AirPlay/406.31.21

pG
ÝX8{?çU±ÏÁÌ·áoªùIôNf8} áµ«CÂA®,/øØ×+zè;ÀePöB
```
```
0%¦æ¸¬+n¸SE@ÌÀ¨òÀ¨øïæÃµ¡êÚhFPý
RTSP/1.0 200 OK
Content-Type: application/octet-stream
Server: AirTunes/150.33
CSeq: 2
Date: Thu Oct 31 02:50:40 2019
Content-Length: 96

P¨ù{6/j¶8ÐBÚ¦#BQ?ÄÚîa+Ì/92C *§ÔèºÍmë³¾fÉMeT?,ºûuâó÷å5	ÓQÇúÁÆ7ëÕNKÚ>u|Vêl,=ªúÑ
```
```
¬+n¸S0%¦æ¸ET@@¤hÀ¨øÀ¨òÃïæêÚhFµ¢PuÎPOST /pair-verify RTSP/1.0
X-Apple-PD: 1
X-Apple-AbsoluteTime: 594183040
Content-Length: 68
Content-Type: application/octet-stream
CSeq: 3
DACP-ID: 9A8670A3D720AAF6
Active-Remote: 1716416520
User-Agent: AirPlay/406.31.21

TÓÇÎkß8ì)ø2úZÀg¸fº¦ñS+¿MïLrN>õ´S3×N`¶ôLºõ±7êâ=dQ
```
```
0%¦æ¸¬+n¸SE¥@Ì{À¨òÀ¨øïæÃµ¢êÚirPüÙRTSP/1.0 200 OK
Content-Type: application/octet-stream
Server: AirTunes/150.33
CSeq: 3
Date: Thu Oct 31 02:50:40 2019

```
```
¬+n¸S0%¦æ¸Eý@@¤¿À¨øÀ¨òÃïæêÚirµ¢PFÁPOST /fp-setup RTSP/1.0
X-Apple-ET: 32
Content-Length: 16
Content-Type: application/octet-stream
CSeq: 4
DACP-ID: 9A8670A3D720AAF6
Active-Remote: 1716416520
User-Agent: AirPlay/406.31.21

FPLY»
```
```
0%¦æ¸¬+n¸SE @ËÿÀ¨òÀ¨øïæÃµ¢êÚjGPûy1RTSP/1.0 200 OK
Server: AirTunes/150.33
CSeq: 4
Date: Thu Oct 31 02:50:40 2019
Content-Length: 142

FPLYÏ2¢W²RO zñdã{ÏD$â~ü
ÖzüÙ]í'0»Y.Ö:MíºÇæMÌý\{VÚã\Î¯ÇC e¥N9Ò[Ûd¹ä]>jð~V+ú@BuêZDÙYrV¹ûæQ8¸'rWP*ÙFh
```
```
¬+n¸S0%¦æ¸E@@¤*À¨øÀ¨òÃïæêÚjGµ£P¹cPOST /fp-setup RTSP/1.0
X-Apple-ET: 32
Content-Length: 164
Content-Type: application/octet-stream
CSeq: 5
DACP-ID: 9A8670A3D720AAF6
Active-Remote: 1716416520
User-Agent: AirPlay/406.31.21

FPLYìÁúuLP'CX)¶þì¥ÐzÁÌ:vÛ.Xö9'÷êéúÆxÕ{r1Î-*G&«câö/Õ
]" úwL¦èÁq{ø¤d'QÝô4ÃD´ÿÄ
AZÄU;FÔ²ô*a«WÇM>ÿ±BÆFà¿ºÛë¢øa
```
```
0%¦æ¸¬+n¸SE±@ÌmÀ¨òÀ¨øïæÃµ£êÚk±PÝ`RTSP/1.0 200 OK
Server: AirTunes/150.33
CSeq: 5
Date: Thu Oct 31 02:50:40 2019
Content-Length: 32

FPLY>ÿ±BÆFà¿ºÛë¢øa
```
```
¬+n¸S0%¦æ¸E@@¢À¨øÀ¨òÃïæêÚk±µ¤
PÚSETUP rtsp://192.168.137.242/11639746425181978695 RTSP/1.0
Content-Length: 533
Content-Type: application/x-apple-binary-plist
CSeq: 6
DACP-ID: 9A8670A3D720AAF6
Active-Remote: 1716416520
User-Agent: AirPlay/406.31.21

bplist00ß	

RetSeiv^timingProtocol[sessionUUIDVosName^osBuildVersion]sourceVersionZtimingPort_isScreenMirroringSessionYosVersionTekeyXdeviceIDUmodelTnameZmacAddress OHI*eÌ«±Á-º5@4.¤SNTP_$A188AFB4-2498-4047-A315-AB33A94CD886YiPhone OSV17A844Y406.31.21Ã	T13.1OHFPLY<ñrj9°s2mj"49£LrfÐ	7 ÀQW£(¨(hÈü|Õ$
¨Ã_10:30:25:B1:D7:4BWiPad7,5jrainv iPad_10:30:25:A6:E6:B8),0?KRaoz¤³¸ÃÅØÜ
!"'r£·
```
```
0%¦æ¸¬+n¸SEÀ@Ì]À¨òÀ¨øïæÃµ¤
êÚn§Pý'oRTSP/1.0 200 OK
Content-Type: application/x-apple-binary-plist
Content-Length: 0
Server: AirTunes/150.33
CSeq: 6
Date: Thu Oct 31 02:50:40 2019

```
```
¬+n¸S0%¦æ¸E¸@@¥À¨øÀ¨òÃïæêÚn§µ¤¢PvGET /info RTSP/1.0
X-Apple-ProtocolVersion: 1
CSeq: 7
DACP-ID: 9A8670A3D720AAF6
Active-Remote: 1716416520
User-Agent: AirPlay/406.31.21

```
```
0%¦æ¸¬+n¸SEä@É8À¨òÀ¨øïæÃµ¤¢êÚo7Pý}uRTSP/1.0 200 OK
Server: AirTunes/150.33
CSeq: 7
Date: Thu Oct 31 02:50:40 2019
Content-Length: 850

bplist00ß	(*


 )+RpkO,Y0IAMNFyLMqMNb8/+dBXTIw24ZPpi5kN/QEKc1oj2xA= TnameXApple TVRvv[statusFlagsD_keepAliveLowPower_keepAliveSendStatsAsBodyRpi_$b08f5a79-db29-4384-b456-a4784d9e6055]sourceVersionV220.68XdeviceID_54:ee:75:8f:b2:1aZmacAddressUmodelZAppleTV5,3\audioFormats¢Ó_audioInputFormatsÿÿü_audioOutputFormatsTtypedÓe^audioLatencies¢!'Ô"$&#%%YaudioTypeWdefault_inputLatencyMicros_outputLatencyMicrosÔ"$&#%%XfeaturesZÿ÷Xdisplays¡,Ü-/1234568(;<.0%%%0.79:%=VheightFUwidth8Xrotation]widthPhysical^heightPhysical[widthPixels\heightPixels[refreshRate<VmaxFPS[overscannedTuuid_$e5f7a68d-7b0f-4305-984b-974f677a150b),\ajmo{}®±Øæíö
&36=QVkpry{ ¨½¾ÔÝæïøú#&/=LXeqsz|~>¶
```
```
¬+n¸S0%¦æ¸E@@¤¶À¨øÀ¨òÃïæêÚo7µ¨^P¨ÍGET_PARAMETER rtsp://192.168.137.242/11639746425181978695 RTSP/1.0
Content-Length: 8
Content-Type: text/parameters
CSeq: 8
DACP-ID: 9A8670A3D720AAF6
Active-Remote: 1716416520
User-Agent: AirPlay/406.31.21

volume
```
```
0%¦æ¸¬+n¸SEÎ @ÌMÀ¨òÀ¨øïæÃµ¨^êÚpPü±ÑRTSP/1.0 200 OK
Audio-Jack-Status: connected; type=analog
Server: AirTunes/150.33
CSeq: 8
Date: Thu Oct 31 02:50:40 2019
Content-Length: 18

Volume: 1.000000
```
```
¬+n¸S0%¦æ¸EÅ@@¤÷À¨øÀ¨òÃïæêÚpµ©PWRECORD rtsp://192.168.137.242/11639746425181978695 RTSP/1.0
CSeq: 9
DACP-ID: 9A8670A3D720AAF6
Active-Remote: 1716416520
User-Agent: AirPlay/406.31.21

```
```
0%¦æ¸¬+n¸SE½¡@Ì]À¨òÀ¨øïæÃµ©êÚp²PûBRTSP/1.0 200 OK
Audio-Jack-Status: connected; type=analog
Audio-Latency: 4410
Server: AirTunes/150.33
CSeq: 9
Date: Thu Oct 31 02:50:40 2019

```
```
¬+n¸S0%¦æ¸E@@¤ªÀ¨øÀ¨òÃïæêÚp²µ©PÝSET_PARAMETER rtsp://192.168.137.242/11639746425181978695 RTSP/1.0
Content-Length: 18
Content-Type: text/parameters
CSeq: 10
DACP-ID: 9A8670A3D720AAF6
Active-Remote: 1716416520
User-Agent: AirPlay/406.31.21

volume: 0.000000
```
```
0%¦æ¸¬+n¸SE©¢@ÌpÀ¨òÀ¨øïæÃµ©êÚqPíRTSP/1.0 200 OK
Audio-Jack-Status: connected; type=analog
Server: AirTunes/150.33
CSeq: 10
Date: Thu Oct 31 02:50:40 2019

```
```
¬+n¸S0%¦æ¸EÆ@@£öÀ¨øÀ¨òÃïæêÚqµªP7SETUP rtsp://192.168.137.242/11639746425181978695 RTSP/1.0
Content-Length: 188
Content-Type: application/x-apple-binary-plist
CSeq: 11
DACP-ID: 9A8670A3D720AAF6
Active-Remote: 1716416520
User-Agent: AirPlay/406.31.21

bplist00ÑWstreams¡ÓTtype]timestampInfo_streamConnectionIDn¥	Ñ
TnameUSubSuÑ

UBePxTÑ
UAfPxTÑ
UBefEnÑ
UEmEncbb}_0A!/DFLOTZ]cfloux~
```
```
0%¦æ¸¬+n¸SE£@ÌÀ¨òÀ¨øïæÃµªêÚs:PÿgöRTSP/1.0 200 OK
Content-Type: application/x-apple-binary-plist
Server: AirTunes/150.33
CSeq: 11
Date: Thu Oct 31 02:50:40 2019
Content-Length: 85

bplist00ÑWstreams¡ÒXdataPortïçTtypen#&+-
```
```
¬+n¸S0%¦æ¸E@@¤ªÀ¨øÀ¨òÃïæêÚs:µ«	PåSET_PARAMETER rtsp://192.168.137.242/11639746425181978695 RTSP/1.0
Content-Length: 18
Content-Type: text/parameters
CSeq: 12
DACP-ID: 9A8670A3D720AAF6
Active-Remote: 1716416520
User-Agent: AirPlay/406.31.21

volume: 0.000000
```
```
0%¦æ¸¬+n¸SE©¥@ÌmÀ¨òÀ¨øïæÃµ«	êÚt$Pþy÷RTSP/1.0 200 OK
Audio-Jack-Status: connected; type=analog
Server: AirTunes/150.33
CSeq: 12
Date: Thu Oct 31 02:50:40 2019

```
## 实现流程
### 发布mDNS服务
在苹果Bonjour SDK中的Example中，有C语言的Demo，在main函数中简单地发布服务
```C
#define kRaopPort 50001
#define kAirplayPort 50002

	// mdns
	static DNSServiceRef airplayRef = NULL;
	static DNSServiceRef raopRef = NULL;
	Opaque16 AirplayPort = { {kAirplayPort >> 8, kAirplayPort & 0xFF} };
	Opaque16 RaopPort = { {kRaopPort >> 8, kRaopPort & 0xFF} };
	static const char AirplayTXT[] = "\x1A" "deviceid=80:e8:2c:1e:d5:a3" \
		"\x18" "features=0x5A7FFFF7,0x1E" \
		"\x11" "rmodel=Android1,0" \
		"\x0E" "srcvers=220.68" \
		"\x10" "model=AppleTV5,3" \
		"\x09" "flags=0x4"
		"\x43" "pk=66be68d67f1bf2d7f304dc87fba345629ac02434817e5b9de77b5fca926d6763" \
		"\x27" "pi=b08f5a79-db29-4384-b456-a4784d9e6055" \
		"\x04" "vv=2";
	static const char RaopTXT[] = "\x06" "tp=UDP" "\x08" "sm=false" "\x08" "sv=false" "\0x04" "ek=1" "\x06" "cn=0,1" "\x04" "ch=2" "\x05" "ss=16" \
		"\x08" "sr=44100" "\x08" "pw=false" "\x04" "vn=3" "\x09" "txtvers=1";
	err = DNSServiceRegister(&airplayRef, 0, opinterface, "Howson", "_airplay._tcp.", "", NULL, AirplayPort.NotAnInteger, 0, NULL, reg_reply, NULL);
	if (!err) err = DNSServiceUpdateRecord(airplayRef, NULL, 0, sizeof(AirplayTXT) - 1, AirplayTXT, 0);

	err = DNSServiceRegister(&raopRef, 0, opinterface, "80E82C1ED5A3@Howson", "_raop._tcp.", "", NULL, RaopPort.NotAnInteger, 0, NULL, reg_reply, NULL);
	if (!err) err = DNSServiceUpdateRecord(raopRef, NULL, 0, sizeof(RaopTXT) - 1, RaopTXT, 0);
```
### 建立TCP连接
使用WinSock建立TCP连接
### bplist解析
成功建立TCP连接后，收到的第一个包是RTSP通信包。第一个包里面含有二进制plist，因此我们需要在TCP层上对他进行解析，解析地库使用github上的PlistCPP库

## 参考博客
[https://blog.csdn.net/u011897062/article/details/79446001](https://blog.csdn.net/u011897062/article/details/79446001)