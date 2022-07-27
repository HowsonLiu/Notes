# airplay协议的其他信令
有些信令在抓包没有抓到，但是在接收端做了处理，在这里记录一下
# OPTION
发送端向接收端询问支持的信令类型

接收端在报文头部返回支持的信令类型
- **SETUP**
- **RECORD**
- **PAUSE**
- **FLUSH**
- **TEARDOWN**
- **OPTIONS**
- **GET_PARAMETER**
- **SET_PARAMETER**

# FLUSH
发送端要求接收端清空缓冲区。此信令只在raop有效，镜像无效

接收端按照FLUSH包中的序号`seq`清空缓冲区