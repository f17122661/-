# sntp
**获取网络服务器的时间** 

## 什么叫时间戳?
 -  "1574668214" 这一串数字表示的是 *unix时间戳是从1970年1月1日（UTC/GMT的午夜）开始所经过的秒数，不考虑闰秒*

## esp8266 获取联网获取时间
- 初始化 sntp 服务器地址 一共三组 0-2  可以选择url 的方式 或者IP地址
``` cpp
ip_addr_t	*addr	=	(ip_addr_t	*)os_zalloc(sizeof(ip_addr_t));

sntp_setservername(0,	us.pool.ntp.org");	//	set	server	0	by	domain	name"

sntp_setservername(1,	"ntp.sjtu.edu.cn");	//	set	server	1	by	domain	name

ipaddr_aton("210.72.145.44",	addr);

sntp_setserver(2,	addr);	//	set	server	2	by	IP	address

sntp_init();//打开

os_free(addr);
```
- 用定时器的方式去扫 sntp
``` cpp
LOCAL	os_timer_t	sntp_timer;

os_timer_disarm(&sntp_timer);

os_timer_setfn(&sntp_timer,	(os_timer_func_t	*)user_check_sntp_stamp,	NULL);

os_timer_arm(&sntp_timer,	100,	0);
```
- 定时器回调获取 时间戳 和 当前时间
``` cpp

void	ICACHE_FLASH_ATTR	user_check_sntp_stamp(void	*arg){

	 uint32	current_stamp;

	 current_stamp	=	sntp_get_current_timestamp();

	 if(current_stamp	==	0){

	 	 os_timer_arm(&sntp_timer,	100,	0);

	 }	else{

	 	 os_timer_disarm(&sntp_timer);

	 	 os_printf("sntp:	%d,	%s	\n",current_stamp,	
sntp_get_real_time(current_stamp));

	 }
} 

```