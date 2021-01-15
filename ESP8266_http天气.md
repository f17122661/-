# esp8266 天气

## 实现步骤
1. 链接STA
2. 创建TCP client
3. DNS获取
4. 发送 TCP 请求体



## 1. 建立STA 链接
- 设置esp8266 模式
- 关闭DHCP获取服务
- 初始化STATION模式下的 静态IP
- 初始化STATION链接
``` cpp
void ICACHE_FLASH_ATTR Station_Mode_Init(void)
{
	// station 模式下 会自动分配 dhcp 所以不用配置
	// 也可以手动配置IP地址 建议在第一次获取完毕后 保存这个地址
	struct ip_info info;
	struct station_config STA_Config;
	wifi_set_opmode(0x01);// 配置 esp8266 模式为 station 模式
	wifi_station_dhcpc_stop();
	IP4_ADDR(&info.ip,192,168,1,154); //设置station自己的地址
	IP4_ADDR(&info.gw,192,168,1,1);// 设置网关地址
	IP4_ADDR(&info.netmask,255,255,255,0);//设置子网掩码
	wifi_set_ip_info(STATION_IF,&info);//作为STA模式 主机的
	os_memset(&STA_Config,0,sizeof(struct station_config));
	os_strcpy(STA_Config.ssid,ESP8266_STA_SSID);
	os_strcpy(STA_Config.password,ESP8266_STA_PASS);
	wifi_station_set_config(&STA_Config); //设置相关参数
}
```

## 2. 创建TCP client
- 初始化TCP Client
``` cpp
struct espconn tcp_client_conn; //声明TCP client初始化结构体
void ICACHE_FLASH_ATTR TCP_Client_Init(void)
{
	tcp_client_conn.type = ESPCONN_TCP ;
	tcp_client_conn.proto.tcp = (esp_tcp *)os_zalloc(sizeof(esp_tcp));
	tcp_client_conn.proto.tcp->local_port = 8266;
	tcp_client_conn.proto.tcp->remote_port = 80; //这个web端口的默认端口需要写 先不管有可能是 nginx 代理的 但是默认都是80 
/*	tcp_client_conn.proto.tcp->remote_ip[0] = 114; //这边因为后期用DNS获取IP地址 所以可以不写
	tcp_client_conn.proto.tcp->remote_ip[1] = 55;
	tcp_client_conn.proto.tcp->remote_ip[2] = 186;
	tcp_client_conn.proto.tcp->remote_ip[3] = 19;*/
	espconn_regist_connectcb(&tcp_client_conn,TCPConnectCallBack); //创建链接成功回调函数
	espconn_regist_reconcb(&tcp_client_conn,TCPReconnectBreakCallBack);//创建连接中途短线回调函数
}
```
- 建立回调函数
``` cpp
void ICACHE_FLASH_ATTR TCP_server_recv_cb(void *arg, char *pdata, unsigned short len) //成功连接收到回调函数
{
	os_printf("%s\r\n",Http_message); //接受到HTTP json 并串口发送
    os_printf("%s\r\n",pdata);
}

void ICACHE_FLASH_ATTR TCP_server_send_cb(void *arg) //发送成功回调函数
{
	os_printf("TCP send success!!\r\n");
}

void ICACHE_FLASH_ATTR TCPReconnectBreakCallBack(void *arg,sint8 err) //连接途中断开 重连回调函数
{
	os_printf("\nReconnectBreak\n");
	espconn_connect((struct espconn *)arg);
}

void ICACHE_FLASH_ATTR TCPDisconnectBreakCallBack(void *arg)//连接关闭回调函数
{
	os_printf("\nESP8266_TCP_Disconnect_OK\n");
}

void ICACHE_FLASH_ATTR TCPConnectCallBack(void *arg)//连接成功回调函数
{
	struct espconn *pesp_conn = arg;

	espconn_regist_sentcb(pesp_conn, TCP_server_send_cb);//绑定连接成功后发送成功的回调函数
	espconn_regist_recvcb(pesp_conn, TCP_server_recv_cb);//绑定连接成功后接受的回调函数
	espconn_regist_disconcb(pesp_conn,TCPDisconnectBreakCallBack);//绑定连接失败的回调函数
	os_printf("\n--------------- ESP8266_TCP_Connect_OK ---------------\n");
	espconn_send((struct espconn *)arg,Http_message,os_strlen(Http_message));//发送HTTP 请求体
}
```

## 3. 读取DNS服务
- 建立DNS获取
    + 参数一`&tcp_client_conn `: TCP连接的初始化结构体地址
    + 参数二 `DNS_SERVER `: DNS域名: `api.thinkpage.cn` 直接用 `#define DNS_SERVER "api.thinkpage.cn"`
    + 参数三 ` &IP_Server`: 获取到域名的IP地址 是个32位的变量 用 结构体 `ip_addr_t IP_Server` 来创建
    + 参数四 `DNS_Found_Callback`: 不管获取是否成功 都会进这个回调 去解析
    ``` cpp
        espconn_gethostbyname(&tcp_client_conn, DNS_SERVER, &IP_Server, DNS_Found_Callback);
    ```

- 创建回调函数 
    ``` cpp
    void ICACHE_FLASH_ATTR DNS_Found_Callback(const	char *name,	ip_addr_t *ipaddr, void	*arg)
    {
        struct espconn * T_arg = (struct espconn *)arg;
        if(ipaddr == NULL)
        {
            os_printf("\r\n---- DomainName Analyse Failed ----\r\n");
            return;
        }else if(ipaddr != NULL && ipaddr->addr != 0)
        {

            IP_Server.addr = ipaddr->addr;
            os_printf("\r\n---- DomainName Analyse Succeed ----\r\n");
            os_printf("user_esp_platform_dns_found	%d.%d.%d.%d\r\n",
            *((uint8	*)&ipaddr->addr),	*((uint8	*)&ipaddr->addr	+	1),
            *((uint8	*)&ipaddr->addr	+	2),	*((uint8	*)&ipaddr->addr	+	3));

            os_memcpy(T_arg->proto.tcp->remote_ip, &IP_Server.addr, 4);

            espconn_connect(T_arg);
        }
    }
    ```
## 程序思路
1. 初始化STA TCP
2. 不打开连接
3. 定时器等待成功连接到AP
4. 读取DNS
5. 成功获取DNS IP
6. 建立TCP 连接域名IP
7. 发送 HTTP 请求体
8. 读取返回的JSON

## 程序编写

``` cpp
#include "ets_sys.h"
#include "osapi.h"
#include "user_interface.h"
#include "driver/uart.h"
#include "c_types.h"
#include "mem.h"
#include "os_type.h"
#include "osapi.h"
#include "queue.h"
#include "ip_addr.h"
#include "espconn.h"
#include "eagle_soc.h"

#define ProjectName "http_GET"

//初始化域名
#define DNS_SERVER "api.thinkpage.cn"

//初始化请求的url
#define Http_message "GET https://api.thinkpage.cn/v3/weather/now.json?key=wcmquevztdy1jpca&location=huzhou&language=en&unit=c HTTP/1.1\r\n"	\
	"Host: api.thinkpage.cn\r\n"	\
	"Connection: close\r\n\r\n"		

#define ESP8266_STA_SSID "ZESHI_2.4G_develop"
#define ESP8266_STA_PASS "YINSHO_WIFI_a8c17d13"

/*
SSID:	ZESHI_2.4G_develop
PASS:	YINSHO_WIFI_a8c17d13
*/

os_timer_t OS_Timer_1;
struct espconn tcp_client_conn; //声明TCP client初始化结构体
struct espconn ST_NetCon;
ip_addr_t IP_Server;

void ICACHE_FLASH_ATTR Station_Mode_Init(void)
{
	// station 模式下 会自动分配 dhcp 所以不用配置
	// 也可以手动配置IP地址 建议在第一次获取完毕后 保存这个地址
	struct ip_info info;
	struct station_config STA_Config;
	wifi_set_opmode(0x01);// 配置 esp8266 模式为 station 模式
	wifi_station_dhcpc_stop();
	IP4_ADDR(&info.ip,192,168,1,154); //设置station自己的地址
	IP4_ADDR(&info.gw,192,168,1,1);// 设置网关地址
	IP4_ADDR(&info.netmask,255,255,255,0);//设置子网掩码
	wifi_set_ip_info(STATION_IF,&info);//作为STA模式 主机的
	os_memset(&STA_Config,0,sizeof(struct station_config));
	os_strcpy(STA_Config.ssid,ESP8266_STA_SSID);
	os_strcpy(STA_Config.password,ESP8266_STA_PASS);
	wifi_station_set_config(&STA_Config); //设置相关参数
}
void ICACHE_FLASH_ATTR TCP_server_recv_cb(void *arg, char *pdata, unsigned short len)
{
    //espconn_sent(pesp_conn,	pdata,	os_strlen(pdata));//返回接受到的数据
	os_printf("%s\r\n",Http_message);
    os_printf("%s\r\n",pdata);
}
void ICACHE_FLASH_ATTR TCP_server_send_cb(void *arg)
{
	os_printf("TCP send success!!\r\n");
}
void ICACHE_FLASH_ATTR TCPReconnectBreakCallBack(void *arg,sint8 err)
{
	os_printf("\nReconnectBreak\n");
	espconn_connect((struct espconn *)arg);
	//os_timer_arm(&OS_Timer_1, 3000, 1);
}
void ICACHE_FLASH_ATTR TCPDisconnectBreakCallBack(void *arg)
{
	os_printf("\nESP8266_TCP_Disconnect_OK\n");
	os_timer_arm(&OS_Timer_1, 3000, 1);
}
void ICACHE_FLASH_ATTR TCPConnectCallBack(void *arg)
{
	struct espconn *pesp_conn = arg;

	espconn_regist_sentcb(pesp_conn, TCP_server_send_cb);
	espconn_regist_recvcb(pesp_conn, TCP_server_recv_cb);
	espconn_regist_disconcb(pesp_conn,TCPDisconnectBreakCallBack);
	os_printf("\n--------------- ESP8266_TCP_Connect_OK ---------------\n");
	espconn_send((struct espconn *)arg,Http_message,os_strlen(Http_message));
}
void ICACHE_FLASH_ATTR TCP_Client_Init(void)
{
	tcp_client_conn.type = ESPCONN_TCP ;
	tcp_client_conn.proto.tcp = (esp_tcp *)os_zalloc(sizeof(esp_tcp));
	tcp_client_conn.proto.tcp->local_port = 8266;
	tcp_client_conn.proto.tcp->remote_port = 80;
	espconn_regist_connectcb(&tcp_client_conn,TCPConnectCallBack);
	espconn_regist_reconcb(&tcp_client_conn,TCPReconnectBreakCallBack);

}
void ICACHE_FLASH_ATTR DNS_Found_Callback(const	char *name,	ip_addr_t *ipaddr, void	*arg)
{
	struct espconn * T_arg = (struct espconn *)arg;
	if(ipaddr == NULL)
	{
		os_printf("\r\n---- DomainName Analyse Failed ----\r\n");
		return;
	}else if(ipaddr != NULL && ipaddr->addr != 0)
	{
		os_printf("\r\n---- DomainName Analyse Succeed ----\r\n");
		os_printf("user_esp_platform_dns_found	%d.%d.%d.%d\r\n",
		*((uint8	*)&ipaddr->addr),	*((uint8	*)&ipaddr->addr	+	1),
		*((uint8	*)&ipaddr->addr	+	2),	*((uint8	*)&ipaddr->addr	+	3));
        IP_Server.addr = ipaddr->addr;//获取到的IP地址存下来
		os_memcpy(T_arg->proto.tcp->remote_ip, &IP_Server.addr, 4);  //并设置重新设置TCP连接请求体
		espconn_connect(T_arg); //打开TCP 连接
	}
}
void ICACHE_FLASH_ATTR OS_Timer_1_cb(void)
{
	uint8 WIFI_STA_Connect;
	struct ip_info esp8266info;
	uint8 esp8266_ip[4];
	WIFI_STA_Connect = wifi_station_get_connect_status();
	if(WIFI_STA_Connect == STATION_GOT_IP)
	{
		os_printf("WIFI STATION_GOT_IP\r\n");
		wifi_get_ip_info(STATION_IF,&esp8266info);
		esp8266_ip[0] = esp8266info.ip.addr;
		esp8266_ip[1] = esp8266info.ip.addr >> 8;
		esp8266_ip[2] = esp8266info.ip.addr >> 16;
		esp8266_ip[3] = esp8266info.ip.addr >> 24;
		os_printf("esp8266 STA IP: %d.%d.%d.%d \r\n",esp8266_ip[0],esp8266_ip[1],esp8266_ip[2],esp8266_ip[3]);
		os_timer_disarm(&OS_Timer_1);
		espconn_gethostbyname(&tcp_client_conn, DNS_SERVER, &IP_Server, DNS_Found_Callback);
	}
}
void OS_Time_Init(void)
{
	os_timer_disarm(&OS_Timer_1);
	os_timer_setfn(&OS_Timer_1,(os_timer_func_t *)OS_Timer_1_cb, NULL);
	os_timer_arm(&OS_Timer_1, 3000, 1);
}
void ICACHE_FLASH_ATTR
user_init(void)
{
	uart_init(115200,115200);
	os_printf("\r\n");
	os_printf("project name:%s\r\n",ProjectName);
    os_printf("SDK version:%s\r\n", system_get_sdk_version());
    Station_Mode_Init();
    TCP_Client_Init();
    OS_Time_Init();
}
```