# **第零章 综述**

最近因为工作需要研究了rsyslog，主要是把官网的文档看了一遍，因为学习的过程中，发现中文资料很少，所以研究的差不多以后，决定拿出来分享一下。

rsyslog官网的文档还是挺烂的（在我看完后，官网有了一次改版，可能好一些了），尤其是配置文件部分，每个版本都有改动，但是官网只有最新版本的参数介绍，好几次我按照文档的参数写配置文件，或者报错，或者参数不起作用，因此大家使用哪个版本的rsyslog，最好是下个该版本的源代码，以供核查参数是否准确。

本文的结构，来自我的内部分享。

使用的rsyslog版本为：7.2.5

# **第一章 rsyslog整体架构**

我主要使用rsyslog来保证日志的可靠性传输，日志中心宕机，client端可以本地保存日志，因此rsyslog与数据库结合方面的知识在文章中没有涉及。

![img](https://images0.cnblogs.com/blog/493056/201303/10231124-47833e3a70d24191aeb6435264c4392f.png)

Rsyslog架构，这是rsyslog官网上的一张图，用来介绍rsyslog的架构，rsyslog的消息流是从输入模块->预处理模块->主队列->过滤模块->执行队列->输出模块。

在这个流程图中，输入、输出、过滤三个部分称为module，输入模块有imklg、imsock、imfile。输出模块有omudp、omtcp、omfile、omprog、ommysql、omruleset（后两者我没有研究，本文不会涉及）。过滤模块研究不多，只会提到mmnormalize。

预处理模块主要解决各种syslog协议实现间的差异，举例说明如果日志系统client端使用rsyslog、server端使用syslog-ng，如果自己不做特殊处理syslog-ng是无法识别的。但是反过来，rsyslog的server端就可以识别syslog-ng发过来的消息。

Input模块包括imklg、imsock、imfile、imtcp等，是消息来源。

Filetr模块处理消息的分析和过滤，rsyslog可以根据消息的任何部分进行过滤，后面会介绍到具体的做法。

Output模块包括omfile、omprog、omtcp、ommysql等。是消息的目的地。

Queue模块负责消息的存储，从Input传入的未经过滤的消息放在主队列中，过滤后的消息放入到不同action queue中，再由action queue送到各个输出模块。

# 1.1启动参数

正常启动：指定-f和-i就可以了，新版本不需要-c 5 这样的参数。

```
rsyslogd -f /root/rsyslog_worker_dir/rsyslog.conf -i /root/rsyslog_worker_dir/rsyslog.pid
```

debug版本：debug消息会输出到标注输出，如果出现未预期的结果，可以尝试使用debug方式，查看处理流程

```
rsyslogd -f /root/rsyslog_worker_dir/rsyslog.conf -i /root/rsyslog_worker_dir/rsyslog.pid -dn >debuglog
```

测试配置文件是否正确：

```
rsyslogd -N1 -f file
```

# 第二章 rsyslog的概念

# 2.1属性替代

Rsyslog 预定义了一些属性，来代表消息中的信息，我们可以在定义输出格式、动态文件名的时候使用到这些属性。这里面比较重要的属性比如：msg（消息体）、hostname、pri（消息等级和类别）、time（时间有关），属性的名称中以$开头的是从本地系统获得的变量、不带$是从消息中获得变量。

属性替代的语法格式：

```
%propname:fromChar:toChar:options:fieldname%
```

属性替换的功能很强大，你可以使用起始字符获取自己所需的字段，也可以使用正则表达式，也可以使用分隔符。举几个例子：为了兼容一个rfc协议，rsyslog规定如果用户输入的msg不是以空格开头，rsyslog会自动补充一个空格，因此如果你希望输出的时候去掉这个空格，就可以使用

```
%msg:2:$%    #选取msg变量中，起始位置为2，终止位置为结尾
```

我们经常需要根据空格来分析字符串，F表示使用字符分割，32是空格的ascii码，例：

```
%msg:F,32:3%  #按照空格分隔，取第三个子串，
```

正则匹配可以匹配特定的文字和格式，我的正则比较差， 避免了使用这部分的内容，所以这部分没有例子了。

属性替代中还用到了一类特殊的以$!开头的变量，这是使用mmnormalize模块时特有的，可以实现类似于syslog-ng中parser模块的功能。后面再讲

# 2.2模板

模板的功能是定义输出格式，或者定义omfile模块的动态路径、动态文件。需要使用上面提到的属性替换。

模板定义的形式有四种，适用于不同的输出模块，一般简单的格式，可以使用string的形式，复杂的格式，建议使用list的形式，使用list的形式，可以使用一些额外的属性字段（property statement），例如：position.from、position.end。

如果不指定输出模板，rsyslog会默认使用RSYSLOG_DEFAULT。

如果你只想输出msg，可以定义模板

```
$template  t_msg, “%msg\n%” 
```

如果想按日期保存输出，需要使用动态路径。可以定义模板

```
$template  f_debug, “/data0/logs/%$year%-%$month%-%$day%/debug.log”
```

# 2.3Ruleset

Ruleset实现的是多实例的功能，可以针对syslog的来源使用不同的过滤规则。需要注意的是，在配置文件中需要先定义ruleset，才可以使用。比较典型的一个例子，针对不同的端口使用不同的过滤规则。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
$Ruleset tcp1999

$RulesetCreateMainQueue on

Local3.*      @@10.0.0.44:1999

$Ruleset tcp2000

$RulesetCreateMainQueue on

Local4.*      @@10.0.0.44:2000
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

在定义好ruleset后，各个输出模块就可以指定自己使用的ruleset了，具体如何指定，可以查看输出模块的手册，一般会有一个ruleset的参数，用来实现这个功能。

# 2.4Filter模块

Rsyslog可以使用syslog标准的过滤规则，同时自己添加了一些扩展。比如可以在输出中指定rsyslog自己的处理方式，可以指定输出template,方法是在规则后面添加template的名字，用分号隔开。

例如我们可以编写一个规则

```
Local3.*    -/data0/logs/local3.log;t_msg   #在这个输出中使用t_msg的模板

Local4.*    -?f_local3_test;t_msg         #问号表示要使用模板定义的动态路径
```

除了syslog标准的规则，rsyslog的作者还自己开发了一个叫做rainerscript的脚本语言，来定义更复杂的过滤过则，rainerscript可以对属性进行startwith、contains、%（取余）等过滤规则，例如

```
If $pri-txt == local3.* and $msg contains “abc” then{  #pri为local3，且在消息中包含子串‘abc’

       *.*   -/data0/logs/local3.log;t_msg

}
```

还有第三种方式是使用属性的表示方式，例如

```
:msg, regex, "^ [g-z]"    /root/rsyslog_worker_dir/2000.log   #以字母g到z开头的消息，注意msg开头有个空格
```

# 2.5队列

队列是rsyslog中比较重要的一个部分，作为使用者，我们需要了解的是队列的种类：主队列和工作队列。从输入模块接收的消息会进入主队列，主队列中的消息，经过过滤模块，会进入到相应的工作队列；队列的四种工作模式：direct mode、disk mode、FixedArray mode和LinkedList mode，前两种是磁盘队列，更可靠，但是性能也较差，后两种是内存队列，区别是前者是预分配队列长度，后者是动态分配，如果你的系统日志流量比较平稳，可以使用预分配队列，如果日志属于突发型，可以使用动态队列。此外，内存队列还可以通过指定一个queuename来添加DA模式，DA模式主要是为了防止意外情况（进程关闭、server端宕机）下，内存队列可以不丢失。

通过查看rsyslog的系统命令，可以知道rsyslog对队列进行大量的可配参数，来定义队列的行为。可以根据需要来进行优化。

# 第三章 实例

# 3.1 Tcp+DA模式实现可靠消息传输。

Rsyslog单独使用了一篇文档来介绍实现可靠消息传输。

首先rsyslog阐述了单独使用tcp协议的不可靠性，比如server端宕机等等情况。为此如上面介绍队列时提到的内容，我们需要在client配置一个本地文件，用来在server端宕机这种情况下，暂时保存消息。需要注意的是，队列名是和过滤规则对应的，一个队列只能用于一个过滤规则，例：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
$ActionQueueType LinkedList

$ActionQueueFileName local3

$ActionResumeRetryCount -1

$ActionQueueSaveOnShutdown on

Local3.*                                            @@10.0.0.44:1999

$ActionQueueType LinkedList

$ActionQueueFileName local4

$ActionResumeRetryCount -1

$ActionQueueSaveOnShutdown on

Local4.*                                            @@10.0.0.44:1999
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

​       此处只是为了说明问题，更简单的写法，是把两条过滤规则写成一条：local3.*;local4.*。还有一点需要强调的是，本地队列只有在需要使用的时候才会创建，当后端出现短暂不可用是，rsyslog的内存队列就可以保存消息，内存队列不够用时，才会创建本地队列。

# 3.3 mmnormalize模块

当我们需要定义一个复杂的输出时，单纯使用模板会显得比较笨拙，这时候我们就可以使用mmnormalize这个模块来帮助我们简化模板。例如我们部门本身使用一套日志格式，但是某个重要客户，需要将他自己的日志按照他所规定的格式保留一份，为此，我们需要定义一个过滤规则，过滤该用户的日志，然后打散日志，重排，下面我会用模板和mmnormalize两种方式来解决这个问题。

首先使用模板，我使用list的方式定义：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
template(name="t_weiyouxi_rewrite" type="list"){

property(name="msg" field.Delimiter="32" field.Number="2")

constant(value=" ")

property(name="msg" field.delimiter="32" field.number="3")

constant(value=" ")

property(name="msg" field.delimiter="32" field.number="4")

constant(value="us - ")

property(name="msg" field.delimiter="32" field.number="6")

constant(value=" ")

property(name="msg" field.delimiter="32" field.number="7")

constant(value=" \"")

property(name="msg" field.delimiter="34" field.number="2")

constant(value="\"")

property(name="msg" field.delimiter="34" field.number="3")

constant(value="\"")

property(name="msg" field.delimiter="34" field.number="4")

constant(value="\" - \"UTRS1=")

property(name="msg" field.delimiter="34" field.number="7" position.from="2")

constant(value=";U_TRS2=-;SUP=-\" \"")

property(name="msg" field.delimiter="34" field.number="6")

constant(value="\" ")

property(name="hostname")

constant(value="\n")

}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这种方式写出的模板是很冗长的，下面使用mmnormalize模块实现，为此，我们需要先定义一个消息格式，来供mmnormalize使用，rulebase.rb：

```
rule=: %xxxdomain:word% %xxx-ip:ipv4% %xxx-resptime:number% %xxx-cputime:number% [%xxx-timestamp:char-to:]%] malibumytime 819 %xxx-version:number% "%xxx-method:word% %xxx-url:word% %xxx-protocol:char-to:"%" %xxx-retcode:number% %xxx-retsize:number% "%xxx-referer:char-to:"%" "%xxx-ua:char-to:"%" %xxx-cookie:word%
```

然后，rsyslog的配置文件中，我们可以将以上定义的字段当做属性来使用（需要以$!开头），如下：

```
$mmnormalizeRuleBase /path/to/rulebase.rb

$template t_weiyouxi_rewrite,  "%$!xxxdomain% %$!xxx-ip% %$!xxx-resptime%us - [%$!xxx-timestamp%] \"%$!xxx-method%  %$!xxx-url%  %$!xxx-protocol%\" %$!xxx-retcode% %$!xxx-retsize% \"%$!xxx-referer%\" - \"UTRS1=%$!xxx-cookie%;U_TRS2=-;SUP=-\" \"%$!xxx-ua%\" %hostname%\n"
```

​       可以看出这种方式比使用模板要简洁清楚多了。不过额外的你需要多使用一个配置文件。

# 3.4 omprog模块

如文档所表述的一样，omprog可以指定第三方的程序来处理模块，运行时，第三方的模块被当做rsyslog的子进程启动，两者通过管道通信。此时过滤规则定义的模板，就是子进程的输入格式。

```
$ActionOMProgBinary /root/rsyslog_worker_dir/prog/start.sh  #脚本可以保证第三方程序可以使用自己的启动参数

local3.*          :omprog:;t_msg
```

# 3.5额外的测试。

1、测试了imtcp和imptcp的区别，测试了两者的性能差不多，ptcp略低，这和官网说的ptcp性能更好不一致，可能我还没有压到极限，没有测出真实数据，测试过程中imtcp的cpu使用较高，大概在130%，imptcp的cpu大概在70%左右，可以预见ptcp使用多线程技术，分担了主线程的压力。

2、其他部门的同事测试tcp+da可以解决server端宕机的问题，但是如果server端因为io较高，造成阻塞，rsyslog并不能解决这个问题，syslog阻塞是个很严重的问题，如果apache直接写syslog，而syslog阻塞会导致apache无法处理请求。因此必须在server端加监控，如果io较高，就需要优化参数，或者增加server机器了。

3、使用-dn模式来测试mmnormalize模块，如过mmnormalize没有收到正确的rulebase，debug文件中会搜索‘unparser’会看到未解析的rulebase字符串，如果解析成功，可以搜索‘gennerate’，可以看到自定义的属性，在真实消息中的值。

4、rsyslog中的参数有的是全局参数，例如Createmode，有的则是对应于某一ruleset或者某一过滤规则，所以使用前，需要对参数的有效范围进行测试。

5、如果你希望client和server端都使用rsyslog，那么你不要修改两者之间的传输模板。因为rsyslog 需要根据syslog协议来解析消息，如果你使用t_msg传输，那么server端无法识别该消息。

6、同事反馈之前使用syslog-ng时消息超过设置的最大消息长度，会分成两条发送，rsyslog行为不同，超过设置的最大消息长度，超出部分会丢弃。

7、rsyslog默认会将特殊字符（\t）转换成#009 由全局配置$EscapeControlCharactersOnReceive 决定，如果自己需要根据\t处理输出时，需将该选项改为off。

8、配置文件每一行之前，只能包含空格，如果半酣其他字符，rsyslog不能识别该行。

9、使用omprog时，rsyslog通过管道将消息发送给第三方程序， 因此当消息量较大时，第三方程序要具备相应的处理能力，否则很容易造成管道阻塞，导致rsyslog阻塞。这个问题挺严重的，我们的经验是server日志转发到redis，但是开始的程序处理能力达不到，导致server端的tcp接收队列阻塞。

# 3.6 未解决的问题

1、本想测试template和mmlognorm两种方式那种更快，但是发现ab压的时候，总是会多出一些输出日志，不知道是不是压力太大，导致有retry操作，发生概率大概3%，现有测试数据，两者的性能差不多。

2、使用 -dn的debug模式，输出太多，不使用debug模式的输出又太少，希望找到其他的输出参数，失败了，估计要看代码了。