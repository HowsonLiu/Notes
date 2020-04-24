# 初始化
Airplay接收端初始化主要分为三部分
- airplay
- raop
- mDNS
## Airplay协议初始化
这里的Airplay协议不是指广义的Airplay协议，而是指除去raop以及mDNS部分所剩余的协议

初始化接受几个参数，分别是：
- **max_client** (必选)
    最多可以同时连接几个设备
- **airplay_port** (必选)
    监听的端口
- callback
    回调类
- logger
    日志类

除此之外，接收端默认指定镜像端口**mirror_port**为7100

首先进行socket环境的初始化

接着初始化配对所需要的参数。配对过程使用了[ed25519加密协议](https://blog.csdn.net/u013137970/article/details/84573265)，加密协议的种子`seed`、公钥`ed_public`、私钥`ed_private`都在此初始化

最后根据**max_client**，使用http协议监听**airplay_port**以及**mirror_port**端口，如果有callback回调类或者logger日志类，则同时绑定到http上（可选）

至此，Airplay协议初始化完成。上面有任何一步失败，都被视为初始化失败

## raop协议初始化
raop协议全称是[远程音频输出协议](https://en.wikipedia.org/wiki/Remote_Audio_Output_Protocol)(Romote Audio Output Protocal)

初始化接受几个参数，分别是
- **max_client** (必选)
    最多可以同时连接几个设备，值与上述Airplay协议一致
- **raop_port** (必选)
    监听的端口
- callback
    回调类
- logger
    日志类

流程也类似

首先首先进行socket环境的初始化

接着初始化配对所需要的参数。**注意这里配对加密也是使用ed25519，但是它并没有沿用airplay协议的公私钥**

最后监听**raop_port**端口，绑定相关回调与日志

至此，raop协议初始化完成。上面有任何一步失败，都视为初始化失败

## mDNS协议初始化
mDNS协议全称是[多播DNS协议](https://baike.baidu.com/item/mdns/7544078?fr=aladdin)(Multicast DNS)，用于局域网的地址发现

要想使用此协议，需要安装并运行苹果的Bonjour服务。作为开发者，我们可以直接下载Bonjour SDK([下载地址](https://developer.apple.com/download/more/?=Bonjour%20SDK%20for%20Windows))

初始化需要确保Bonjour服务运行，接着将上面的Airplay服务以及raop服务都注册到mDNS上就好了
```c++
int
dnssd_register_raop(dnssd_t *dnssd, const char *name, unsigned short port, const char *hwaddr, int hwaddrlen, int password)
{
	TXTRecordRef txtRecord;
	char servname[MAX_SERVNAME];
	int ret;

	assert(dnssd);
	assert(name);
	assert(hwaddr);

	dnssd->TXTRecordCreate(&txtRecord, 0, NULL);
	dnssd->TXTRecordSetValue(&txtRecord, "txtvers", strlen(RAOP_TXTVERS), RAOP_TXTVERS);
	dnssd->TXTRecordSetValue(&txtRecord, "ch", strlen(RAOP_CH), RAOP_CH);
	dnssd->TXTRecordSetValue(&txtRecord, "cn", strlen(RAOP_CN), RAOP_CN);
	dnssd->TXTRecordSetValue(&txtRecord, "et", strlen(RAOP_ET), RAOP_ET);
	dnssd->TXTRecordSetValue(&txtRecord, "sv", strlen(RAOP_SV), RAOP_SV);
	dnssd->TXTRecordSetValue(&txtRecord, "da", strlen(RAOP_DA), RAOP_DA);
	dnssd->TXTRecordSetValue(&txtRecord, "sr", strlen(RAOP_SR), RAOP_SR);
	dnssd->TXTRecordSetValue(&txtRecord, "ss", strlen(RAOP_SS), RAOP_SS);
	if (password) {
		dnssd->TXTRecordSetValue(&txtRecord, "pw", strlen("true"), "true");
	} else {
		dnssd->TXTRecordSetValue(&txtRecord, "pw", strlen("false"), "false");
	}
	dnssd->TXTRecordSetValue(&txtRecord, "vn", strlen(RAOP_VN), RAOP_VN);
	dnssd->TXTRecordSetValue(&txtRecord, "tp", strlen(RAOP_TP), RAOP_TP);
	dnssd->TXTRecordSetValue(&txtRecord, "md", strlen(RAOP_MD), RAOP_MD);
	dnssd->TXTRecordSetValue(&txtRecord, "vs", strlen(GLOBAL_VERSION), GLOBAL_VERSION);
	dnssd->TXTRecordSetValue(&txtRecord, "sm", strlen(RAOP_SM), RAOP_SM);
	dnssd->TXTRecordSetValue(&txtRecord, "ek", strlen(RAOP_EK), RAOP_EK);
	dnssd->TXTRecordSetValue(&txtRecord, "sf", strlen(RAOP_SF), RAOP_SF);
	dnssd->TXTRecordSetValue(&txtRecord, "am", strlen(GLOBAL_MODEL), GLOBAL_MODEL);

	/* Convert hardware address to string */
	ret = utils_hwaddr_raop(servname, sizeof(servname), hwaddr, hwaddrlen);
	if (ret < 0) {
		/* FIXME: handle better */
		return -1;
	}

	/* Check that we have bytes for 'hw@name' format */
	if (sizeof(servname) < strlen(servname)+1+strlen(name)+1) {
		/* FIXME: handle better */
		return -2;
	}

	strncat(servname, "@", sizeof(servname)-strlen(servname)-1);
	strncat(servname, name, sizeof(servname)-strlen(servname)-1);

//	int len = dnssd->TXTRecordGetLength(&txtRecord) + 1;
//	char* txt = malloc(len);
//	txt[len - 1] = '\0';
//	memcpy(txt, dnssd->TXTRecordGetBytesPtr(&txtRecord), len);
	/* Register the service */
	ret = dnssd->DNSServiceRegister(&dnssd->raopService, 0, 0,
	                          servname, "_raop._tcp",
	                          NULL, NULL,
	                          htons(port),
	                          dnssd->TXTRecordGetLength(&txtRecord),
	                          dnssd->TXTRecordGetBytesPtr(&txtRecord),
		MyRegisterServiceReply, NULL);

	/* Deallocate TXT record */
	dnssd->TXTRecordDeallocate(&txtRecord);
	return 1;
}
```
```c++
int
dnssd_register_airplay(dnssd_t *dnssd, const char *name, unsigned short port, const char *hwaddr, int hwaddrlen)
{
	TXTRecordRef txtRecord;
	char deviceid[3*MAX_HWADDR_LEN];
	char features[16];
	int ret;

	assert(dnssd);
	assert(name);
	assert(hwaddr);

	/* Convert hardware address to string */
	ret = utils_hwaddr_airplay(deviceid, sizeof(deviceid), hwaddr, hwaddrlen);
	if (ret < 0) {
		/* FIXME: handle better */
		return -1;
	}

	features[sizeof(features)-1] = '\0';
	snprintf(features, sizeof(features)-1, "0x%x", GLOBAL_FEATURES);

	dnssd->TXTRecordCreate(&txtRecord, 0, NULL);
	dnssd->TXTRecordSetValue(&txtRecord, "srcvers", strlen(GLOBAL_VERSION), GLOBAL_VERSION);
	dnssd->TXTRecordSetValue(&txtRecord, "deviceid", strlen(deviceid), deviceid);
	dnssd->TXTRecordSetValue(&txtRecord, "features", strlen("0x5A7FFFF7, 0x1E"), "0x5A7FFFF7,0x1E");
	dnssd->TXTRecordSetValue(&txtRecord, "model", strlen(GLOBAL_MODEL), GLOBAL_MODEL);
	dnssd->TXTRecordSetValue(&txtRecord, "flags", strlen(RAOP_SF), RAOP_SF);
	dnssd->TXTRecordSetValue(&txtRecord, "vv", strlen(RAOP_VV), RAOP_VV);

//	int len = dnssd->TXTRecordGetLength(&txtRecord) + 1;
//	char* txt = malloc(len);
//	txt[len - 1] = '\0';
//	memcpy(txt, dnssd->TXTRecordGetBytesPtr(&txtRecord), len);

	/* Register the service */
	ret = dnssd->DNSServiceRegister(&dnssd->airplayService, 0, 0,
	                          name, "_airplay._tcp",
	                          NULL, NULL,
	                          htons(port),
	                          dnssd->TXTRecordGetLength(&txtRecord),
	                          dnssd->TXTRecordGetBytesPtr(&txtRecord),
	                          MyRegisterServiceReply, NULL);

	/* Deallocate TXT record */
	dnssd->TXTRecordDeallocate(&txtRecord);
	return 0;
}
```
可以看到，密码功能由raop注册的mDNS实现，因为投屏一定推音频流，但推音频流不一定投屏
## 初始化总结
广义的Airplay协议由Airplay协议、raop协议以及mDNS协议组成。其中Airplay负责视频流，raop负责音频流、mDNS负责局域网设备发现。