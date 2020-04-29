# 镜像连接
windows开启接收端服务：**raop_port**为5001、**airplay_port**为7001、**mirror_port**为7100

开始投屏，通过抓包发现，通讯端口均为5001，原因未知

下面根据抓包分析镜像连接的过程，流程如下：
1. **GET /info**
2. **pair**
    1. **pair-setup**
    2. **pair-verify**
3. **fairplay**
4. **SETUP1**
5. **GET PARAMETER**
6. **RECORD**
7. **SET PARAMETER**
8. **SETUP2**

# GET /info
发送端请求接收端信息，报文格式是bplist
```
GET /info RTSP/1.0
X-Apple-ProtocolVersion: 1
Content-Length: 70
Content-Type: application/x-apple-binary-plist
CSeq: 0
DACP-ID: 1FF65A04A9252F60
Active-Remote: 2814835199
User-Agent: AirPlay/409.16

bplist00ÑYqualifier¡ZtxtAirPlay"
```
解析后如下
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>qualifier</key>
	<array>
		<string>txtAirPlay</string>
	</array>
</dict>
</plist>
```
IOS11设备内容不一样

接收端返回信息
```
RTSP/1.0 200 OK
CSeq: 0
Server: AirTunes/220.68
Content-Type: application/x-apple-binary-plist
Content-Length: 836

bplist00ÿÿüYaudioTypeß
$&(*	

%')+TtypeXdisplaysTuuid_audioInputFormatsXfeatures[refreshRateÔ "!!_aa:54:01:af:c3:c1dUmodel<VheightZAppleTV2,1]sourceVersion_keepAliveLowPowerÜ-/123456(9;<.0!!!0.78:!=]widthPhysicalV220.68Ó[overscanned[widthPixelsO °w'ÖöÍnµÞR^ÃÍê¢Rh?ë!.ø¢$eTçZmacAddress¡,¢\audioFormatsTnameRvvZÿ÷_inputLatencyMicros[statusFlagsWAppleTVÔ "!!Wdefault_$2e388006-13ba-4041-9a67-25dd4a43d536Ó_outputLatencyMicros^audioLatenciesXrotation\heightPixelsVmaxFPSXdeviceID_audioOutputFormats_$e0ff8a27-6738-3d56-8a16-cc53aacee925_keepAliveSendStatsAsBody^heightPhysicaleUwidthRpiRpk¢#8±úRC"djN¿g°WT+
:M×ô¨viÞv ¦am?PÓ¢¥ìjH@>¨
```
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>sourceVersion</key>
	<string>220.68</string>
	<key>statusFlags</key>
	<integer>4</integer>
	<key>macAddress</key>
	<string>aa:54:01:af:c3:c1</string>
	<key>deviceID</key>
	<string>aa:54:01:af:c3:c1</string>
	<key>name</key>
	<string>AppleTV</string>
	<key>vv</key>
	<integer>2</integer>
	<key>keepAliveLowPower</key>
	<integer>1</integer>
	<key>keepAliveSendStatsAsBody</key>
	<integer>1</integer>
	<key>pi</key>
	<string>2e388006-13ba-4041-9a67-25dd4a43d536</string>
	<key>audioFormats</key>
	<array>
		<dict>
			<key>audioOutputFormats</key>
			<integer>33554428</integer>
			<key>type</key>
			<integer>100</integer>
			<key>audioInputFormats</key>
			<integer>33554428</integer>
		</dict>
		<dict>
			<key>audioOutputFormats</key>
			<integer>33554428</integer>
			<key>type</key>
			<integer>101</integer>
			<key>audioInputFormats</key>
			<integer>33554428</integer>
		</dict>
	</array>
	<key>audioLatencies</key>
	<array>
		<dict>
			<key>audioType</key>
			<string>default</string>
			<key>inputLatencyMicros</key>
			<false/>
			<key>outputLatencyMicros</key>
			<false/>
			<key>type</key>
			<integer>100</integer>
		</dict>
		<dict>
			<key>audioType</key>
			<string>default</string>
			<key>inputLatencyMicros</key>
			<false/>
			<key>outputLatencyMicros</key>
			<false/>
			<key>type</key>
			<integer>101</integer>
		</dict>
	</array>
	<key>pk</key>
	<data>
	sHcn1vbNbgi1jt5SXsPN6qJSrZ9oP+shLviiBSRlVOc=
	</data>
	<key>model</key>
	<string>AppleTV2,1</string>
	<key>features</key>
	<integer>130367356919</integer>
	<key>displays</key>
	<array>
		<dict>
			<key>height</key>
			<integer>1080</integer>
			<key>width</key>
			<integer>1920</integer>
			<key>rotation</key>
			<false/>
			<key>widthPhysical</key>
			<false/>
			<key>heightPhysical</key>
			<false/>
			<key>widthPixels</key>
			<integer>1920</integer>
			<key>heightPixels</key>
			<integer>1080</integer>
			<key>refreshRate</key>
			<integer>60</integer>
			<key>features</key>
			<integer>14</integer>
			<key>maxFPS</key>
			<integer>30</integer>
			<key>overscanned</key>
			<false/>
			<key>uuid</key>
			<string>e0ff8a27-6738-3d56-8a16-cc53aacee925</string>
		</dict>
	</array>
</dict>
</plist>
```
目前在这一步还没有发现关键的字段
# pair配对认证
配对认证分为两部分：设置setup与认证verify

**pair所使用到的一种算法是[ECDH密钥协商算法](https://www.orchome.com/1049)，属于[DC密钥协商算法](https://www.cnblogs.com/qcblog/p/9016704.html)**
- ## pair-setup
    发送端会发送一个32位的数据
    ```
    POST /pair-setup RTSP/1.0
    Content-Length: 32
    Content-Type: application/octet-stream
    CSeq: 1
    DACP-ID: 1FF65A04A9252F60
    Active-Remote: 2814835199
    User-Agent: AirPlay/409.16

    f8} áµ«CÂA®,/øØ×+zè;ÀePöB       // 32位的数据
    
    ```
    接收端并不会实际处理这个数据，而是检查它的位数是否为32位。如果正确，则回应**协商用公钥**`ed_ours`,同时把配对状态置为`STATUS_SETUP`状态
    ```
    RTSP/1.0 200 OK
    CSeq: 1
    Server: AirTunes/220.68
    Content-Type: application/octet-stream
    Content-Length: 32

    [àý-HøÃ+$6dK×z¿ëzÕÎz
    
    // 16进制
    00b0   xx xx xx 5b 95 e0 1e 01 fd 2d 48 8d 08 f8 04 c3
    00c0   2b 24 1d 36 64 4b 0b d7 7a 1b 81 86 bf eb 7a d5
    00d0   ce 0e 7a
    ```
    setup完成
    ### 为什么只检查位数而不使用数据呢？
    我猜测这跟ECDH机制有关。发送端与接收端在ECDH（pair配对流程）之前，需要各自生成公钥与私钥。在这个例子里，接收端生成了公钥`ed_ours`与`ed_private`。生成公私钥是需要一个随机数作为种子的，而接收端是使用32位数据作为随机数。因此这个包应该是确认，发送端也是使用32位数据作为随机数生成公私钥。确保大家的公私钥格式一致，才能正确执行下面的ECDH认证流程。

- ## pair-verify
    在verify前，接收端检查配对状态`STATUS_SETUP`以及报文字节**68**，不正确则不回应
    **verify有点类似三次握手，发送方的报文有两种形式，根据报文的首个字节判断：**
    - ### 首字节为1（第一次握手）
        ```
        POST /pair-verify RTSP/1.0
        X-Apple-PD: 1
        X-Apple-AbsoluteTime: 609411011
        Content-Length: 68
        Content-Type: application/octet-stream
        CSeq: 2
        DACP-ID: 1FF65A04A9252F60
        Active-Remote: 2814835199
        User-Agent: AirPlay/409.16

        // 报文
        .ÚÃGÚòHÉLú¡¾ÐÈ×¨¨¹b/²´D3Ïf8} áµ«CÂA®,/øØ×+zè;ÀePöB

        // 报文16进制展示
        0120   xx xx xx xx xx xx xx 01 00 00 00 2e da c3 13 47
        0130   15 da f2 48 c9 4c fa a1 be 15 d0 c8 d7 a8 80 06
        0140   a8 b9 62 2f b2 b4 44 33 10 cf 0f 66 38 7d a0 e1
        0150   b5 ab 87 43 c2 41 ae 2c 94 1d 2f f8 d8 d7 8a 2b
        0160   7a e8 3b c0 ad 83 65 50 1f f6 42
        ```
        前4个字节保留，中间32个字节是公钥`ed_their`，后32个字节是公钥`ecdh_theirs`
        ```
        // 回应包
        RTSP/1.0 200 OK
        CSeq: 2
        Server: AirTunes/220.68
        Content-Type: application/octet-stream
        Content-Length: 96

        // 报文
        ê¨¥.ËpDî²Yl÷|ÎP=¡0ÑÞj:TIÝx%´
        2\\Rm¡ ¹äÏ_Ýz<ÅLØºoè¿B?H[:ÀØnöÆðI!pø¯Tà<Ò

        // 报文16
        00b0   xx xx xx 88 ea a8 a5 2e 8c cb 18 70 87 44 ee b2
        00c0   59 6c f7 7c 14 ce 50 8a 3d a1 30 91 02 d1 de 6a
        00d0   3a 07 54 49 dd 17 03 9c 78 25 b4 0d 12 32 5c 5c
        00e0   88 52 9b 6d a1 a0 b9 e4 91 cf 5f 9c dd 7a 3c c5
        00f0   4c 1c d8 ba 8b 6f 17 e8 bf 42 3f 48 5b 1f 3a 83
        0100   c0 83 d8 6e f6 c6 f0 49 21 93 0e 70 f8 af 54 e0
        0110   10 3c d2
        ```
        接收端回应96个字节的报文，前32个字节是公钥`edch_ours`，后面64个字节是签名（**第二次握手**）
        握手结束，接收端将状态设置为`STATUS_HANDSHAKE`
    - ### 首字节为0（第三次握手）
        ```
        // 接收包
        POST /pair-verify RTSP/1.0
        X-Apple-PD: 1
        X-Apple-AbsoluteTime: 609411011
        Content-Length: 68
        Content-Type: application/octet-stream
        CSeq: 3
        DACP-ID: 1FF65A04A9252F60
        Active-Remote: 2814835199
        User-Agent: AirPlay/409.16

        // 报文
        áp=s_
        2TàÐiv®yÜùeÈj,(ãk±½áéë2°JÛ×7æßþcQkc»	Y2eh&O?

        // 报文16进制
        0120   xx xx xx xx xx xx xx 00 00 00 00 0c 00 e1 70 3d
        0130   73 5f 02 1b 0a 32 54 e0 d0 69 76 ae 79 dc 8a f9
        0140   65 c8 1d 6a 2c 28 e3 6b 90 90 b1 bd e1 e9 eb 32
        0150   b0 94 4a db d7 37 e6 df fe 9f 63 51 6b 0b 63 98
        0160   bb 09 59 32 65 68 26 06 4f 3f 13
        ```
        发送方发送68位数据，前4位保留，后64位是签名
        接收端先检查状态`STATUS_HANDSHAKE`，没问题后验证签名。验证签名成功，则三次握手成功，配对成功
        ```
        RTSP/1.0 200 OK
        CSeq: 3
        Server: AirTunes/220.68
        ```
        接收端回空包结束，将状态置为`STATUS_FINISHED`
## pair协商密钥流程
流程中用了几套加密协议：AES、ED25519、EDCH、SHA512。其中ED25519与EDCH十分容易让人混淆

假设发送端为Alice，接收端为Blob
1. Alice根据32位随机数生成公钥`PublicA`与私钥`PrivateA`，Blob同样根据32位随机数生成公钥`PublicB`与私钥`PrivateB`。（**ed25519**）
2. **pair-setup**确保**ed25519**随机数位数一致
3. 进行标准**ECDH**流程
    1. Alice生成32位随机数作为私钥`ECDH_Private_A`
    2. Alice通过[Curve25519椭圆曲线](https://blog.csdn.net/u011897062/article/details/89633193)与常量`basepoint`生成公钥`EDCH_Public_A`
    3. Alice将`PublicA`以及`ECDH_Public_A`发送给Blob（**pair-verify 1**）
    *此时Blob知道Alice全部公钥，但是Alice一无所知*
    4. Blob重复1、2步生成私钥`ECDH_Private_B`以及公钥`EDCH_Public_B`
    5. Blob收到`ECDH_Public_A`后，通过Curve25519椭圆曲线与私钥`ECDH_Private_B`生成共享密钥`EDCH_Key`
        **注意这是EDCH的关键，它保证了双方都能通过对方公钥以及己方私钥生成相同的共享密钥**
    6. Blob以此拼接己方公钥`EDCH_Public_B`与对方公钥`EDCH_Public_A`作为**64 bit**信息`msg`
    7. Blob使用对方公钥`PublicA`与己方私钥`PrivateB`进行**ED25519**签名信息`msg`，得到`ed_msg`
    8. Blob使用**SHA512**签名常量字符串`Pair-Verify-AES-Key`与`Pair-Verify-AES-IV`，得到`aes_key`(16 bit)与`aes_iv`(16 bit)
    9. Blob使用**AES**算法，参数是`aes_key`、`aes_iv`，加密签名`ed_msg`，得到**64 bit**加密签名`aes_ed_msg`
    10. Blob 传输**96 bit**报文，前32位是`PublicB`，后64位是`aes_ed_msg`。（**pair-verify 2**）
    *此时Alice也知道Blob所有公钥了*
    11. Alice重复Blob 5-10步的步骤，复现出`aes_ed_msg`验签
    12. Alice验签成功后，拼接己方公钥`EDCH_Public_A`与对方公钥`EDCH_Public_B`（**恰好与第6步反过来了**），得到`msg2`
    13. Alice重复7-9步，加密`msg2`得到签名`aes_ed_msg2`，传送给Blob（**pair-verify 3**）
    14. Blob验签，回复空包完成**ECDH**（**pair-verify 4**）

之后，大家都使用`EDCH_Key`作为密钥加解密