# pair中的edch
```c++
typedef enum {
	STATUS_INITIAL,
    STATUS_SETUP,
	STATUS_HANDSHAKE,
	STATUS_FINISHED
} status_t;

struct pairing_session_s {
	status_t status;

	unsigned char ed_private[64];                       // ed25519私钥
	unsigned char ed_ours[32];                          // ed25519公钥
	unsigned char ed_theirs[32];                        // ed25519对方公钥

	unsigned char ecdh_ours[32];                        // edch公钥
	unsigned char ecdh_theirs[32];                      // edch对方公钥
	unsigned char ecdh_secret[32];                      // edch共享密钥
};
```

# raop协议
```c++
struct raop_rtp_s {
    logger_t *logger;                                   // 日志类 在SETUP 1完成
    raop_callbacks_t callbacks;                         // 回调类 在SETUP 1完成

    /* Buffer to handle all resends */
    raop_buffer_t *buffer;                              // 音频缓冲区 在SETUP 1完成

    // mirror_buffer_t *mirror;     lhs deleted
    /* Remote address as sockaddr */
    struct sockaddr_storage remote_saddr;               // 发送端socket相关 在SETUP 1完成
    socklen_t remote_saddr_len;
    const char remoteName[128];                         // 设备名 在SETUP 1完成
    const char remoteDeviceId[128];                     // 设备ID 在SETUP 1完成

    /* MUTEX LOCKED VARIABLES START */
    /* These variables only edited mutex locked */      // 下面变量都需要加锁访问
    int running;
    int joined;

    float volume;                                       // 音量值 0.0 ~ -144.0
    int volume_changed;                                 // 音量改变标记
    unsigned char *metadata;
    int metadata_len;
    unsigned char *coverart;
    int coverart_len;
    char *dacp_id;
    char *active_remote_header;
    unsigned int progress_start;
    unsigned int progress_curr;
    unsigned int progress_end;
    int progress_changed;

    int flush;
    thread_handle_t thread;
    thread_handle_t thread_time;
    mutex_handle_t run_mutex;
    mutex_handle_t time_mutex;
    cond_handle_t time_cond;
    /* MUTEX LOCKED VARIABLES END */

    /* Remote control and timing ports */               // 端口相关
    unsigned short control_rport;
    unsigned short timing_rport;                        // 时间同步协议远端端口 在SETUP 1完成

    /* Sockets for control, timing and data */
    int csock, tsock, dsock;

    /* Local control, timing and data ports */
    unsigned short control_lport;
    unsigned short timing_lport;
    unsigned short data_lport;

    /* Initialized after the first control packet */    // 接收端socket相关
    struct sockaddr_storage control_saddr;
    socklen_t control_saddr_len;
    unsigned short control_seqnum;
};
```
```c++
struct raop_buffer_s {                          // 音频缓冲区实现
    logger_t *logger;
    unsigned char aeskey[RAOP_AESKEY_LEN];      // aes密钥 在SETUP 1完成
    unsigned char aesiv[RAOP_AESIV_LEN];        // aes初偏移 在SETUP 1完成

    HANDLE_AACDECODER phandle;                  //aac解码器句柄 在SETUP 1完成

    /* First and last seqnum */
    int is_empty;
    unsigned short first_seqnum;                // 播放的序号
    unsigned short last_seqnum;                 // 收到的序号

    /* RTP buffer entries */
    raop_buffer_entry_t entries[RAOP_BUFFER_LENGTH];

    /* Buffer of all audio buffers */
    int buffer_size;
    void *buffer;
};
```
# raop镜像
```c++
struct raop_rtp_mirror_s {
    logger_t *logger;                               // 日志类 在SETUP 1完成
    raop_callbacks_t callbacks;                     // 回调类 在SETUP 1完成

    /* Buffer to handle all resends */
    mirror_buffer_t *buffer;                        // 镜像缓冲区 在SETUP 1完成

    //raop_rtp_mirror_t *mirror;    lhs deleted
    /* Remote address as sockaddr */
    struct sockaddr_storage remote_saddr;           // 发送端socket 在SETUP 1完成
    socklen_t remote_saddr_len;
	const char remoteName[128];                     // 设备名 在SETUP 1完成
	const char remoteDeviceId[128];                 // 设备ID 在SETUP 1完成

    /* MUTEX LOCKED VARIABLES START */
    /* These variables only edited mutex locked */
    int running;
    int joined;

    int flush;
    thread_handle_t thread_mirror;
    thread_handle_t thread_time;
    // For thread_mirror exit unexpeced.
    thread_handle_t thread_exit_exception;
    mutex_handle_t run_mutex;

    mutex_handle_t time_mutex;
    cond_handle_t time_cond;
    /* MUTEX LOCKED VARIABLES END */
    int mirror_data_sock, mirror_time_sock;

    unsigned short mirror_data_lport;               // 镜像数据本机端口 在SETUP 2完成，动态分配
    unsigned short mirror_timing_rport;             // 时间同步协议远端端口 在SETUP 1完成
    unsigned short mirror_timing_lport;             // 时间同步协议本机端口 在SETUP 2完成，动态分配
};
```
```c++
struct mirror_buffer_s {
    logger_t *logger;
    struct AES_ctx aes_ctx;
    int nextDecryptCount;
    uint8_t og[16];
    /* AES key and IV */
    // 需要二次加工才能使用
    unsigned char aeskey[RAOP_AESKEY_LEN];
    unsigned char ecdh_secret[32];
};
```
