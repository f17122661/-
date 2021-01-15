# esp8266 smartconfig

## 配置步骤
1. 先尝试读出flash中的SSID 和 密码
2. 打开STA模式 尝试连接
3. 连接失败 打开 smartconfig
4. 配置smartconfig回调函数
5. 回调函数中 等待 airkiss 广播
6. 连接成功后保存SSID 和 密码

## 程序
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
#include "cJSON.h"
#include "smartconfig.h"

#define ProjectName "Smartconfig_WechatAirkiss"

#define		Sector_STA_INFO		0x80			// 【STA参数】保存扇区


#define ESP8266_STA_SSID "ZESHI_2.4G_develop"
#define ESP8266_STA_PASS "YINSHO_WIFI_a8c17d13"


os_timer_t OS_Timer_1;
struct station_config STA_INFO;
struct ip_info ST_ESP8266_IP;

void ICACHE_FLASH_ATTR Station_Mode_Init(void)
{
	// station 模式下 会自动分配 dhcp 所以不用配置
	// 也可以手动配置IP地址 建议在第一次获取完毕后 保存这个地址
	struct ip_info info;
	struct station_config STA_Config;
	wifi_set_opmode(0x01);// 配置 esp8266 模式为 station 模式
    wifi_station_set_config(&STA_INFO);//获取连接参数
    wifi_station_connect();//开始连接
}
// SmartConfig状态发生改变时，进入此回调函数
//--------------------------------------------
// 参数1：sc_status status / 参数2：无类型指针【在不同状态下，[void *pdata]的传入参数是不同的】
//=================================================================================================================
void ICACHE_FLASH_ATTR smartconfig_done(sc_status status, void *pdata)
{
	os_printf("\r\n------ smartconfig_done ------\r\n");

	switch(status)
	{
		case SC_STATUS_WAIT:
			 os_printf("\r\nSC_STATUS_WAIT\r\n");
			 break;
		case SC_STATUS_FIND_CHANNEL: //等待广播
			os_printf("\r\nSC_STATUS_FIND_CHANNEL\r\n");
			os_printf("\r\n---- Please Use WeChat to SmartConfig ------\r\n\r\n");
			break;
		case SC_STATUS_GETTING_SSID_PSWD:
			os_printf("\r\nSC_STATUS_GETTING_SSID_PSWD\r\n");
			sc_type *type = pdata; // 获取【SmartConfig类型】指针
			if (*type == SC_TYPE_ESPTOUCH)
			{ os_printf("\r\nSC_TYPE:SC_TYPE_ESPTOUCH\r\n"); }
			else
			{
				os_printf("\r\nSC_TYPE:SC_TYPE_AIRKISS\r\n"); //收到广播type airkiss
			}
			break;

		case SC_STATUS_LINK:
			os_printf("\r\nSC_STATUS_LINK\r\n");
			struct station_config *sta_conf = pdata; //收到后 接受发来的请求体 是STA 结构体 
			spi_flash_erase_sector(Sector_STA_INFO);//开辟FLASH空间
			spi_flash_write(Sector_STA_INFO*4096, (uint32 *)sta_conf, 96);//保存flash 空间
			wifi_station_set_config(sta_conf);			// 设置STA参数【Flash】
			wifi_station_disconnect();					// 断开STA连接
			wifi_station_connect();						// ESP8266连接WIFI
			break;
		case SC_STATUS_LINK_OVER:
			os_printf("\r\nSC_STATUS_LINK_OVER\r\n");
			smartconfig_stop();		// 停止SmartConfig，释放内存
			struct ip_info esp8266info;
			uint8 esp8266_ip[4];
			if(STATION_GOT_IP == wifi_station_get_connect_status()) //获取到IP 
			{
				os_printf("WIFI STATION_GOT_IP\r\n");
				wifi_get_ip_info(STATION_IF,&esp8266info);
				esp8266_ip[0] = esp8266info.ip.addr;
				esp8266_ip[1] = esp8266info.ip.addr >> 8;
				esp8266_ip[2] = esp8266info.ip.addr >> 16;
				esp8266_ip[3] = esp8266info.ip.addr >> 24;
				os_printf("esp8266 STA IP: %d.%d.%d.%d \r\n",esp8266_ip[0],esp8266_ip[1],esp8266_ip[2],esp8266_ip[3]);


               /**
               *开始做别的事情
               */  

			}
			break;
	}
}

void ICACHE_FLASH_ATTR OS_Timer_1_cb(void)
{
	uint8 WIFI_STA_Connect;
	struct ip_info esp8266info;
	uint8 esp8266_ip[4];
	WIFI_STA_Connect = wifi_station_get_connect_status();

	switch(WIFI_STA_Connect)
	{
		case STATION_GOT_IP://判断是否获取到IP
			os_printf("WIFI STATION_GOT_IP\r\n");
			wifi_get_ip_info(STATION_IF,&esp8266info);
			esp8266_ip[0] = esp8266info.ip.addr;
			esp8266_ip[1] = esp8266info.ip.addr >> 8;
			esp8266_ip[2] = esp8266info.ip.addr >> 16;
			esp8266_ip[3] = esp8266info.ip.addr >> 24;
			os_printf("esp8266 STA IP: %d.%d.%d.%d \r\n",esp8266_ip[0],esp8266_ip[1],esp8266_ip[2],esp8266_ip[3]);
			os_timer_disarm(&OS_Timer_1);
			break;

		case STATION_CONNECTING: //一直在获取 或者获取错误 获取之后断开 一律 打开 smartconfig
			os_printf("WIFI STATION_CONNECTING\r\n");
			os_timer_disarm(&OS_Timer_1);
			smartconfig_set_type(SC_TYPE_AIRKISS);//打开smartconfig smartconfig  = AIRKISS
			smartconfig_start(smartconfig_done);//绑定打开后的回调函数 会一直在回调函数中等待 手机 airkiss广播

			break;
		case STATION_WRONG_PASSWORD:
			os_printf("WIFI STATION_WRONG_PASSWORD\r\n");
			os_printf("WIFI OPEN SMARTCONFIG\r\n");
			os_timer_disarm(&OS_Timer_1);
			smartconfig_set_type(SC_TYPE_AIRKISS);
			smartconfig_start(smartconfig_done);



			break;

		default:break;


	}





}
void OS_Time_Init(void)
{
	os_timer_disarm(&OS_Timer_1);
	os_timer_setfn(&OS_Timer_1,(os_timer_func_t *)OS_Timer_1_cb, NULL);
	os_timer_arm(&OS_Timer_1, 3000, 1);
}



/******************************************************************************
 * FunctionName : user_init
 * Description  : entry of user application, init user function here
 * Parameters   : none
 * Returns      : none
*******************************************************************************/
void ICACHE_FLASH_ATTR
user_init(void)
{
	uart_init(115200,115200);
	os_printf("\r\n");
	os_printf("project name:%s\r\n",ProjectName);
    os_printf("SDK version:%s\r\n", system_get_sdk_version());

    spi_flash_erase_sector(Sector_STA_INFO); //清除flash中的WIFI参数
    os_memset(&STA_INFO,0,sizeof(struct station_config));//初始化STA_INFO 都设为0
    spi_flash_read(Sector_STA_INFO*4096,(uint32 *)&STA_INFO, 96); //读取flash中的WIFI参数
    STA_INFO.ssid[31] = 0;//设置参数ssid参数最后一位为'\0' 方便os_printf();
    STA_INFO.password[63] = 0;//设置参数password参数最后一位为'\0' 方便os_printf();
    os_printf("\r\nSTA_INFO.ssid=%s\r\nSTA_INFO.password=%s\r\n",STA_INFO.ssid,STA_INFO.password);

    Station_Mode_Init(); //开始连接 第一次坑定连接失败的
    OS_Time_Init();//初始化定时器 捕捉是否连接成功



}

uint32 ICACHE_FLASH_ATTR
user_rf_cal_sector_set(void)

{
    enum flash_size_map size_map = system_get_flash_size_map();
    uint32 rf_cal_sec = 0;
    switch (size_map) {
        case FLASH_SIZE_4M_MAP_256_256:
            rf_cal_sec = 128 - 5;
            break;

        case FLASH_SIZE_8M_MAP_512_512:
            rf_cal_sec = 256 - 5;

            break;

        case FLASH_SIZE_16M_MAP_512_512:
            rf_cal_sec = 512 - 5;

            break;
        case FLASH_SIZE_16M_MAP_1024_1024:
            rf_cal_sec = 512 - 5;

            break;

        case FLASH_SIZE_32M_MAP_512_512:
            rf_cal_sec = 1024 - 5;

            break;
        case FLASH_SIZE_32M_MAP_1024_1024:
            rf_cal_sec = 1024 - 5;

            break;

        case FLASH_SIZE_64M_MAP_1024_1024:
            rf_cal_sec = 2048 - 5;

            break;
        case FLASH_SIZE_128M_MAP_1024_1024:
            rf_cal_sec = 4096 - 5;

            break;
        default:
            rf_cal_sec = 0;

            break;
    }

    return rf_cal_sec;
}

void ICACHE_FLASH_ATTR
user_rf_pre_init(void)
{
}

void ICACHE_FLASH_ATTR delay_ms(uint16 t)
{
	while(t --)
	{
		os_delay_us(1000);
	}
}

```