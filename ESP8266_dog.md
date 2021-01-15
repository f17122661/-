# ESP8266 DOG


**以下程序会出现一直复位的情况**


- `system_soft_wdt_feed();`
   + 功能: *喂软软件看门狗*
   + 主机: *仅支持再软件看门狗开启的情况下,调用本接口*
   + void: *system_soft_wdt_feed(void)*
   + 参数: *无*
   + 返回: *无*




``` cpp
void ICACHE_FLASH_ATTR
user_init(void)
{
	uint8 *str = "uart0_tx_buffer";
	uart_init(BIT_RATE_115200, BIT_RATE_115200);
	os_printf("hello world!!!!\n");
    os_printf("SDK version:%s\n", system_get_sdk_version());
    uart0_tx_buffer(str, strlen(str));
    while(1)
    {
        system_soft_wdt_feed();
        // 这里应该添加喂狗程序 不然会出现复位的情况
    }
}
```