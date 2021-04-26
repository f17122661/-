# esp8266 C json 处理
## 一. JSON介绍

JSON (JavaScript Object Notation, JS 对象简谱) 是一种轻量级的数据交换格式。它基于 ECMAScript (欧洲计算机协会制定的js规范)的一个子集，采用完全独立于编程语言的文本格式来存储和表示数据。

### 1.1 JSON 语法规则

在 JS 语言中，一切都是对象。 因此，任何支持的类型都可以通过 JSON 来表示，例如字符串、数字、对象、数组等。但是对象和数组是比较特殊且常用的两种类型：

- 对象表示为键值对
- 数据由逗号分隔
- 花括号保存对象
- 方括号保存数组

### 1.2 SJON 键/值对

JSON 键值对是用来保存 JS 对象的一种方式，键/值对组合中的键名写在前面并用双引号 "" 包裹，使用冒号 : 分隔，然后紧接着值：

`{"firstName": "Json"}`

## 二. 添加cJSON

链接：https://pan.baidu.com/s/1t4hn6CHpqK94OJpk4b0Bjw 提取码：zyjb
将 cJSON.h 添加到工程的 include/ 目录下
将 cJSON.c 添加到工程的 user/ 目录下

## 三. 生成JSON 数据

流程: 创建JSON结构体-> 添加数据->释放内存

### 3.1 创建JSON 结构体
``` cpp
cJSON *pRoot = cJSON_CreateObject();                         // 创建JSON根部结构体
cJSON *pValue = cJSON_CreateObject();                        // 创建JSON子叶结构体
```
### 3.2 添加字符串类型数据
``` cpp
cJSON_AddStringToObject(pRoot,"mac","65:c6:3a:b2:33:c8");    // 添加字符串类型数据到根部结构体
cJSON_AddItemToObject(pRoot, "value",pValue);
cJSON_AddStringToObject(pValue,"day","Sunday");              // 添加字符串类型数据到子叶结构体
```

### 3.3 添加整型数据
``` cpp
cJSON_AddNumberToObject(pRoot,"number",2);                   // 添加整型数据到根部结构体
```

### 3.4 添加数组类型数据

####  3.4.1 整型数组
``` cpp
int hex[5]={51,15,63,22,96};
cJSON *pHex = cJSON_CreateIntArray(hex,5);                   // 创建整型数组类型结构体
cJSON_AddItemToObject(pRoot,"hex",pHex);                     // 添加整型数组到数组类型结构体
```

#### 3.4.2 JSON对象数组
``` cpp
cJSON * pArray = cJSON_CreateArray();                        // 创建数组类型结构体
cJSON_AddItemToObject(pRoot,"info",pArray);                  // 添加数组到根部结构体
cJSON * pArray_relay = cJSON_CreateObject();                 // 创建JSON子叶结构体
cJSON_AddItemToArray(pArray,pArray_relay);                   // 添加子叶结构体到数组结构体            
cJSON_AddStringToObject(pArray_relay, "relay", "on");        // 添加字符串类型数据到子叶结构体
```

### 3.5 格式化JSON对象
``` cpp
char *sendData == cJSON_Print(pRoot);                        // 从cJSON对象中获取有格式的JSON对象
os_printf("data:%s\n", sendData);                            // 打印数据
```

## 生成JSON格式：
``` json
{
    "mac": "65:c6:3a:b2:33:c8",
    "value": 
     {
        "day": "Sunday"                
     },
    "number": 2,
    "hex": [51,15,63,22,96],
    "info": 
    [
        {
            "relay": "on"        
        }
    ]
}
```

### 3.6 释放内存
``` cpp
cJSON_free((void *) sendData);                             // 释放cJSON_Print ()分配出来的内存空间
cJSON_Delete(pRoot);                                       // 释放cJSON_CreateObject ()分配出来的内存空间
```

## 四. 解析JSON数据

流程：判断JSON格式 --> 解析数据 --> 释放内存

### 4.1 判断是否JSON格式
``` cpp
// receiveData是要剖析的数据
//首先整体判断是否为一个json格式的数据
cJSON *pJsonRoot = cJSON_Parse(receiveData);
//如果是否json格式数据
if (pJsonRoot !=NULL)
{
    ···
}
```

### 4.2 解析字符串类型数据
``` cpp
char bssid[23] = {0};
cJSON *pMacAdress = cJSON_GetObjectItem(pJsonRoot, "mac");    // 解析mac字段字符串内容
if (!pMacAdress) return;                                      // 判断mac字段是否json格式
else
{
    if (cJSON_IsString(pMacAdress))                           // 判断mac字段是否string类型
    {
        strcpy(bssid, pMacAdress->valuestring);               // 拷贝内容到字符串数组
    }
}
```
### 4.3 解析子叶结构体
``` cpp
char strDay[23] = {0};
cJSON *pValue = cJSON_GetObjectItem(pJsonRoot, "value");      // 解析value字段内容
if (!pValue) return;                                          // 判断value字段是否json格式
else
{
    cJSON *pDay = cJSON_GetObjectItem(pValue, "day");         // 解析子节点pValue的day字段字符串内容
    if (!pDay) return;                                        // 判断day字段是否json格式
    else
    {
        if (cJSON_IsString(pDay))                             // 判断day字段是否string类型
        {
            strcpy(strDay, pDay->valuestring);                // 拷贝内容到字符串数组
        }
    }
}
```
### 4.4 解析整型数组数据
``` cpp
cJSON *pArry = cJSON_GetObjectItem(pJsonRoot, "hex");        // 解析hex字段数组内容
if (!pArry) return;                                          // 判断hex字段是否json格式
else
{
    int arryLength = cJSON_GetArraySize(pArry);              // 获取数组长度
    int i;
    for (i = 0; i < arryLength; i++)
    {                                                        // 打印数组内容
        os_printf("cJSON_GetArrayItem(pArry, %d)= %d\n",i,cJSON_GetArrayItem(pArry, i)->valueint);        
    }
}
```

### 4.5 解析JSON对象数组数据
``` cpp
cJSON *pArryInfo = cJSON_GetObjectItem(pJsonRoot, "info");   // 解析info字段数组内容
cJSON *pInfoItem = NULL;
cJSON *pInfoObj = NULL;
char strRelay[23] = {0};
if (!pArryInfo) return;                                      // 判断info字段是否json格式
else
{
    int arryLength = cJSON_GetArraySize(pArryInfo);          // 获取数组长度
    int i;
    for (i = 0; i < arryLength; i++)
    {
        pInfoItem = cJSON_GetArrayItem(pArryInfo, i);        // 获取数组中JSON对象
        if(NULL != pInfoItem)
        {
            pInfoObj = cJSON_GetObjectItem(pInfoItem,"relay");// 解析relay字段字符串内容   
            if(pInfoObj)
            {
                strcpy(strRelay, pInfoObj->valuestring);      // 拷贝内容到字符串数组
            }
        }                                                   
    }
}
```
### 4.6 释放内存
``` cpp
cJSON_Delete(pJsonRoot);                                      // 释放cJSON_Parse()分配出来的内存空间
```

## 五. 常用库函数

1. 从给定的JSON字符串中得到cJSON对象
cJSON *cJSON_Parse(const char *value)
2. 从cJSON对象中获取有格式的JSON对象
3. char *cJSON_Print(cJSON *item)
删除cJSON对象，释放链表占用的内存空间
4. void cJSON_Delete(cJSON *c)
获取cJSON对象数组成员的个数
5. int cJSON_GetArraySize(cJSON *array)
根据下标获取cJSON对象数组中的对象
6. cJSON *cJSON_GetArrayItem(cJSON*array,int item)
根据键获取对应的值（cJSON对象）
7. cJSON *cJSON_GetObjectItem(cJSON*object,const char *string)
新增一个字符串类型字段到JSON格式的数据
8. cJSON_AddStringToObject(object,name,s)
新增一个新的子节点cJSON到根节点
9. void cJSON_AddItemToObject(cJSON *object,const char *string,cJSON *item)




### 解析实例

``` json
{
    "results":[
        {
            "location":{
                "id":"WTMRU3PTDC3X",
                "name":"Huzhou",
                "country":"CN",
                "path":"Huzhou,Huzhou,Zhejiang,China",
                "timezone":"Asia/Shanghai",
                "timezone_offset":"+08:00"
            },
            "now":{
                "text":"Light rain",
                "code":"13",
                "temperature":"8"
            },
            "last_update":"2019-11-27T12:50:00+08:00"
        }
    ]
}
```


``` cpp
void ICACHE_FLASH_ATTR TCP_server_recv_cb(void *arg, char *pdata, unsigned short len)
{
	char *p;
	int  i;
	cJSON *pJson  = NULL;
	cJSON *pInfoItem = NULL;
	cJSON *pLocation = NULL;
	cJSON *pWeather = NULL;
	int Arraylength;
    p = os_strchr(pdata,'{'); //找到json 开始的地址
    if(p)
    {
    	pJson= cJSON_Parse(p);
		if(pJson != NULL)
		{
			os_printf("捕捉到JSON!");
			cJSON *pArrayResults = cJSON_GetObjectItem(pJson, "results");
			Arraylength = cJSON_GetArraySize(pArrayResults);
			os_printf("查找到JSON数组对象成员个数:%d",Arraylength);
			for( i = 0; i<Arraylength; i ++)
			{
				pInfoItem = cJSON_GetArrayItem(pArrayResults, i);
				switch(i)
				{
					case 0 :

						pLocation = cJSON_GetObjectItem(pInfoItem,"location");
						pLocation = cJSON_GetObjectItem(pLocation,"name");

						pWeather = cJSON_GetObjectItem(pInfoItem,"now");
						pWeather = cJSON_GetObjectItem(pWeather,"text");

						os_printf("===========================\r\n");
						os_printf("===========================\r\n");
						os_printf("===========================\r\n");
						os_printf("%s Weather:%s\r\n",pLocation->valuestring,pWeather->valuestring);
						os_printf("===========================\r\n");
						os_printf("===========================\r\n");
						os_printf("===========================\r\n");
						break;
				}
			}

		}
    	 cJSON_Delete(pJson); //清空JSON内存
    }
    else
    {
    	os_printf("没有捕捉到JSON!");
    }
}

```
