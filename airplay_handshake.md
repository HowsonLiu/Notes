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

# GET /info
此阶段发送端请求获取接收端相关的信息，报文使用bplist格式
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
接收端返回信息：除了支持的音频格式、视频格式外、还返回硬件信息。也是使用bplist格式回应
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
# pair配对认证
配对认证分为两部分：设置setup与认证verify
- ## pair-setup
    发送端会发送一个32位的数据，但是接收端只会验证长度，并不会使用该数据，原因未知
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
    接收端回应ed25519的公钥，同时把配对状态置为`STATUS_SETUP`状态
    ```
    RTSP/1.0 200 OK
    CSeq: 1
    Server: AirTunes/220.68
    Content-Type: application/octet-stream
    Content-Length: 32

    [àý-HøÃ+$6dK×z¿ëzÕÎz           // 公钥
    ```
    setup完成
- ## pair-verify
    在verify前，接收端需要检查配对状态，如果状态并非为`STATUS_SETUP`，则不回应报文
    接收包一般为68个字节，如果字节数不对，则包异常不回应
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
        前4个字节保留，中间32个字节是发送端的公钥，后32个字节是发送端密文
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
        接收端回应96个字节的报文，前32个字节是EDCH的公钥，后面64个字节是签名（**第二次握手**）
        **实际上这一轮属于密钥协商，并不是加密，使用的算法是[ECDH密钥协商算法](https://www.orchome.com/1049)**
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
## pair总结
pair流程主要是协商一个ed25519加密密钥