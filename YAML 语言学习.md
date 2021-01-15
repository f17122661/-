# YAML 语言学习

## 什么是YAML

YAML（IPA: /ˈjæməl/，尾音类似camel骆驼）是一个可读性高，用来表达资料序列的YAML参考了其他多种语言，包括：XML、C语言、Python、Perl以及电子邮件格式RFC2822。Clark Evans在2001年在首次发表了这种语言，另外Ingy döt Net与Oren Ben-Kiki也是这语言的共同设计者。目前已经有数种[编程语言](http://zhidao.baidu.com/search?word=%B1%E0%B3%CC%D3%EF%D1%D4&fr=qb_search_exp&ie=gbk)或[脚本语言](http://zhidao.baidu.com/search?word=%BD%C5%B1%BE%D3%EF%D1%D4&fr=qb_search_exp&ie=gbk)支援（或者说解析）这种语言。
YAML是"YAML Ain't a Markup Language"（YAML不是一种置标语言）的递回缩写。
在开发的这种语言时，YAML 的意思其实是："Yet Another Markup Language"（仍是一种置标语言），但为了强调这种语言以数据做为中心，而不是以置标语言为重点，而用返璞词重新命名。

## YAML 作为配置文件

### 1. 基本语法

- 大小写敏感
- 使用缩进表示层级关系
- 缩进时不允许使用TAB建，只允许使用空格
- 缩进的空格数目不重要，只要相同层级的元素左侧对齐即可

### 2. YAML 支持的数据结构

 1. 对象：键值对的集合，又称为映射（mapping）/ 哈希（hashes） / 字典（dictionary）

    - animal :pets

	2. 数组：一组按次序排列的值，又称为序列（sequence） / 列表（list)

    - Cat
    - Dog
    - Goldfish

	3. 符合结构

    ```
    languages:
     - Ruby
     - Perl
     - Python 
     websites:
     YAML: yaml.org 
     Ruby: ruby-lang.org 
     Python: python.org 
     Perl: use.perl.org
    ```

    

    

​	