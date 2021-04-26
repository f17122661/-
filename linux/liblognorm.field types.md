# liblognorm.field types

##  1. number

一个或多个十进制数

.log

```shell
1 number:"123456"
```

.rb:

``` shell	
  1 version=2
  2 rule=:number:"%
  3     value:number
  4     %%
  5     s:rest
  6     %
```

通过命令行解析

``` shell
[root@localhost test]# sed -n 1p test.log  |lognormalizer  -r test.rb -U
{ "s": "\"", "value": "123456" }
```

## 2.float

以非科学形式表示的浮点pt数

.log

```shell
1 number:"1.23456"
```

.rb:

```shell	
  1 version=2
  2 rule=:number:"%
  3     value:float
  4     %%
  5     s:rest
  6     %
```

通过命令行解析

```shell
[root@localhost test]# sed -n 1p test.log  |lognormalizer  -r test.rb -U
{ "s": "\"", "value": "1.23456" }
```

## 3.hexnumber

.log

```shell
1 number:"0x30,0x32,0x11"
```

.rb:

```shell	
  1 version=2
  2 rule=:number:"%
  3     value:hexnumber
  4     %%
  5     s:rest
  6     %
```

通过命令行解析

```shell
[root@localhost test]# sed -n 1p test.log  |lognormalizer  -r test.rb -U
{ "s": "\"", "value": "0x30,0x32,0x11" }
```

## 4.whitespace

.log

```shell
1 number:"      asd"
```

.rb:

```shell	
  1 version=2
  2 rule=:number:"%
  3     value:whitespace
  4     %%
  5     s:rest
  6     %
```

通过命令行解析

```shell
[root@localhost test]# sed -n 1p test.log  |lognormalizer  -r test.rb -U
{ "s": "asd\"", "value": "    " }
```

## 5.string

到空格为止

.log

```shell
1 number:"string1@$#@D2dd ,.&^"
```

.rb:

```shell	
  1 version=2
  2 rule=:number:"%
  3     value:string
  4     %%
  5     s:rest
  6     %
```

通过命令行解析

```shell
[root@localhost test]# sed -n 1p test.log  |lognormalizer  -r test.rb -U
{ "s": " ,.&^\"", "value": "string1@$#@D2dd" }

```

## 6.whitespace





 