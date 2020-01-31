# Airplay协议中的配对认证(pair verify)
## Server端准备
### 1. 生成TOKEN
首先生成非对称加密密钥对，将其私钥由字节转换成16进制字符串。
接着随机生成16字节的ClientID。
最后将返回ClientID@16进制私钥作为TOKEN。
```java
/**
     * Use this method once to generated an "authToken" which is the
     * combination of a random "user" and the hash of an EdDSA-key.
     * Persist the authToken somewhere, either in a static final, inside a property etc.
     *
     * @return An authToken to be used with the constructor of AirPlayAuth.
     */
    public static String generateNewAuthToken() {
        String clientId = AuthUtils.randomString(16);
        return clientId + "@" + net.i2p.crypto.eddsa.Utils.bytesToHex(new KeyPairGenerator().generateKeyPair().getPrivate().getEncoded());
    }
```
此TOKEN需要被妥善保存
### 2. 初始化
接收一个TOKEN，按照1步骤逆序分解。
对私钥(字节)进行`PKCS#8`标准编码转换，将转换结果作为EdDSA签名私钥`authKey`保存
## 开始验证
### 1. 建立TCP连接
### 2. 随机生成32位Server私钥，接着用Curve25519根据Server私钥生成Server公钥
### 3. 发送第一个/pair-verify
类型是"application/octet-stream"
数据由三部分：
第一部分是四个字节{1,0,0,0}
第二部分是Server公钥
第三部分是EdDSA签名公钥(从`authKey`获取)
### 4. 解析回应包
回应包数据段[0-32]为AppleTV公钥
1. 将ATV公钥与Server私钥进行Curve25519加密获取共享密钥(32字节)
2. SHA512消息填充常量字段"Pair-Verify-AES-Key"(UTF-8)，再填充共享密钥(UTF-8),最后摘要，并截取摘要前16字节作为Aes加密密钥`AesKey`
3. SHA512消息填充常量字段"Pair-Verify-AES-IV"(UTF-8)，再填充共享密钥(UTF-8),最后摘要，并截取摘要前16字节作为Aes加密初始向量`AesIV`
4. 使用`AesKey`与`AesIV`初始化AesCtr128，然后对回应包32字节后的剩余内容消息填充
5. EdDSA使用`authKey`初始化，并对{Server公钥，ATV公钥}的内存段签名
6. AesCtr128对5的结果消息填充
### 5. 发送第二个/pair-verify
类型是"application/octet-stream"
数据由两部分：
第一部分是四个字节{0,0,0,0}
第二部分是6的结果