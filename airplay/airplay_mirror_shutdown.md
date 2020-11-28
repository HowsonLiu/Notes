# 镜像结束
## 由发送端结束
流程如下：
1. **结束RTP流**
2. **TEARDOWN**
3. **TCP四次挥手**
## 结束RTP流
关闭镜像RTP流
## TEARDOWN
TEARDOWN信令是四次挥手，发送端结束的话，第一次由发送端发出
- ### 发送端第一次挥手
    ```
    þ\hßE×0%¦æ¸EU@@¥iÀ¨çÀ¨À[I)Ío=¶¾3
    Pw)"TEARDOWN rtsp://192.168.137.1/4493862425083725759 RTSP/1.0
    Content-Length: 69
    Content-Type: application/x-apple-binary-plist
    CSeq: 16
    DACP-ID: DDFFAC568D171725
    Active-Remote: 83994535
    User-Agent: AirPlay/420.45

    bplist00ÑWstreams¡ÑTtypen
    ```
    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
    <plist version="1.0">
    <dict>
        <key>streams</key>
        <array>
            <dict>
                <key>type</key>
                <integer>110</integer>
            </dict>
        </array>
    </dict>
    </plist>
    ```
- ### 接收端第二次挥手
    接收端检查报文`type`字段
    - **96** 属于音频流，接收端重置raop相关socket
    - **110** 属于镜像流，接收端重置mirror相关socket

    接收端回应空包作为第二次挥手
    ```
    RTSP/1.0 200 OK
    CSeq: 16
    Server: AirTunes/220.68


    ```
- ### 发送端第三次挥手
    发送端发送信令是**TEARDOWN**的bplist格式空包
    ```
    TEARDOWN rtsp://192.168.137.1/4493862425083725759 RTSP/1.0
    Content-Length: 42
    Content-Type: application/x-apple-binary-plist
    CSeq: 17
    DACP-ID: DDFFAC568D171725
    Active-Remote: 83994535
    User-Agent: AirPlay/420.45

    bplist00Ð	
    ```
    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
    <plist version="1.0">
    <dict/>
    </plist>
    ```
- ### 接收端第四次挥手
    接收端回应空包
    ```
    RTSP/1.0 200 OK
    CSeq: 17
    Server: AirTunes/220.68


    ```

## TCP四次挥手
进行标准的TCP四次挥手流程

## 由接收端结束
接收端关闭所有TCP连接

# 镜像音频结束