# 在Grafana中使用PostgreSQL

Grafana附带了一个内置的PostgreSQL数据库插件,你可以通过可视化的方式查询PostgreSQL数据库中的数据

## 添加数据源

1. 单击顶部标题中的Grafana图标打开侧边栏
2. 在*Configuration*图标的侧边菜单栏下方找到*DataSources*
3. 单击*+Add data source* 顶部标题中的按钮
4. 从下拉栏的类型中选择PostgreSQL

## 数据源选项

| 名称         | 描述                                                         |
| :----------- | :----------------------------------------------------------- |
| Name         | 数据源的名称                                                 |
| Default      | 默认数据源                                                   |
| Host         | PostgreSQL的IP地址主机明和端口                               |
| Database     | PostgreSQL数据库的名称                                       |
| User         | PostgreSQL用户名                                             |
| Passwd       | PostgreSQL用户密码                                           |
| SSL Mode     | 是否启用SSL链接服务器                                        |
| Max open     | 打开数据库链接的最大数 默认是无限的                          |
| Max idle     | 空闲池中最大的连接数默认时2                                  |
| Max lifetime | 可以重新链接的最大时间 默认4小时                             |
| Version      | 版本                                                         |
| TimescaleDB  | TimescaleDB是一个构建为PostgreSQL扩展的时间序列数据库。如果启用，Grafana将`time_bucket`在`$__timeGroup`宏中使用并在查询构建器中显示TimescaleDB特定的聚合函数 |

## 最小间隔时间

[$__interval](https://grafana.com/docs/reference/templating/#the-interval-variable),[$__interval_ms](https://grafana.com/docs/reference/templating/#the-interval-ms-variable) 变量的下限值，建议设置的写入频率值1m（每分钟写入数据一次） 。还可以在“数据源选项”下的仪表板面板中重写此配置选项。注意，这个值必须是有效的时间标识符，例如1m(1分钟)或30s(30秒)。支持以下时间标识符:

| 描述 | 标识符 |
| ---- | ------ |
| y    | 年     |
| m    | 月     |
| w    | 周     |
| d    | 日     |
| h    | 小时   |
| m    | 分     |
| s    | 秒     |
| ms   | 毫秒   |

## 数据库用户权限

因为 Grafana不验证查询是否安全。查询可以包含任何SQL语句。为了防止类似`DELETE FROM user`和`DROP TABLE user`的情况发生.数据库用户应仅授于查询指定数据库权限或表选择的权限在添加数据源时。

```sql
CREATE USER grafanareader WITH PASSWORD 'password';
 GRANT USAGE ON SCHEMA schema TO grafanareader;
 GRANT SELECT ON schema.table TO grafanareader;
```

## 查询编辑器

> 仅适用于Grafana v5.3 +

![](https://grafana.com/docs/img/docs/v53/postgres_query.gif)

在Graph或Singlestat的编辑面板找到metrics选项卡中PostgreSQL查询编辑器。单击面板标题进入编辑模式，然后编辑。

在面板编辑模式下 单击 generate SQL链接，将展现执行的SQL语句

###  选择 表, 时间列和指标列(FROM)

当您第一次进入编辑模式或添加新的查询时，Grafana将用第一个表来预填充查询生成器，该表具有一个时间戳和一个数列.

在FROM字段中，Grafana建议将数据库用户表位于`search_path`。如果要选择的表或视图不在`search_path`中可以手动输入完全限定名称（schema.table）`public.metrics`。

time cloumn 是指包含时间值的列的名称。Metric column字段的值是可选的。如果选择了值，则Metric column字段将用作级名称。

建议Metric column 只能包含这几种数据类型（char，varchar，text）。如果要使用具有不同数据类型的列作为标准Metric column，则可以使用强制转换：`ip::text`。您还可以在公制列字段中输入任意SQL表达式，以评估文本数据类型`hostname || ' ' || container_name`。

### 列，窗口和聚合函数（SELECT）

在该`SELECT`行中，您可以指定要使用的列和函数。在`column`中，您可以编写任意表达式代替colum:

`column1 * column2 / column3`。

查询编辑器中的可用功能取决于您在配置数据源时选择的PostgreSQL版本。如果使用聚合函数，则需要对结果集进行分组。如果添加聚合函数，编辑器将自动添加`GROUP BY time`。

编辑器简化查询的这一部分.

例如：

![](https://grafana.com/docs/img/docs/v53/postgres_select_editor.png)

以上将生成以下PostgreSQL `SELECT`子句

```sql
avg(tx_bytes) OVER (ORDER BY "time" ROWS 5 PRECEDING) AS "tx_bytes"
```

单击加号按钮并从菜单中选择来添加更多列值。多个列值将在图表面板中绘制单独的级数

### 过滤数据（WHERE）

单击`WHERE`环境右侧的加号图标,添加过滤器。您可以通过单击过滤器并选择来`Remove`过滤器。当前所选时间范围的过滤器会自动添加到新查询中。

### 分组

按时间或其他列进行分组,单击分组行末尾的加号图标。建议下拉列表仅显示当前所选表格的文本列，也可以手动输入任何列。您可以通过单击该项目然后选择来删除该组`Remove`。

如果添加任何分组，则所有选定的列都需要应用聚合函数。添加分组时，查询构建器将自动将聚合函数添加到所有列,而不需要额外使用聚合函数。

### 差距填补

当您按时间分组时，Grafana可以填写缺失值。time函数接受两个参数。第一个参数是您要分组的时间窗口，第二个参数是您希望Grafana用其填充缺失项目的值。

## 文本编辑器模式（RAW）

您可以通过单击hamburger图标并选择`Switch editor mode`或单击`Edit SQL`查询下方来切换到原始查询编辑器模式

> 如果您使用原始查询编辑器，请确保您的查询至少已具有`ORDER BY time`返回时间范围的过滤器

## 指令宏

可以在查询中使用宏来简化语法并允许动态部分。

| 指令                                                  | 描述                                                         |
| ----------------------------------------------------- | ------------------------------------------------------------ |
| *$__time(dateColumn)*                                 | 列的表达式将会被代替并重命名为`time`。例如，**dateColumn as time** |
| *$__timeSec(dateColumn)*                              | 列的表达式将会被代替并重命名为`time`,并将值转换为unix时间戳。例如，**extract(epoch from dateColumn) as time** |
| *$ __ timeFilter（dateColumn）*                       | 将使用指定的列名称替换为时间范围过滤器。例如，**dateColumn BETWEEN'2017-04-21T05：01：17Z'AND'2017-04-21T05：06：17Z'** |
| *$__timeFrom()*                                       | 被当前活动开始时间所取代。例如，**'2017-04-21T05：01：17Z'** |
| *$__timeTo()*                                         | 被当前活动结束时间所取代。例如，**'2017-04-21T05：06：17Z'** |
| *$__timeGroup(dateColumn,‘5m’)*                       | 被GROUP BY中可用的表达式替换。例如**\*(extract(epoc from dateColumn/300):: bigint300** |
| *$__timeGroup(dateColumn,‘5m’, 0)*                    | 与上面相同，但填充参数的缺失点将会被Grafana添加为0           |
| *$__timeGroup(dateColumn,‘5m’, NULL)*                 | 与上面相同，但NULL将用作缺失点的值。                         |
| *$__timeGroup(dateColumn,‘5m’, previous)*             | 与上面相同，但该系列中的先前值将用作填充值。如果还没有设置任何值，将使用NULL（仅在Grafana 5.3+中可用）。 |
| *$__timeGroupAlias(dateColumn,‘5m’)*                  | 替换为与$ __ timeGroup相同的表达式，还添加了列别名（仅在Grafana 5.3+中可用）。 |
| *$__unixEpochFilter(dateColumn)*                      | 使用指定列名称的时间范围过滤器替换，时间表示为unix时间戳。例如，**dateColumn> = 1494410783 AND dateColumn <= 1494497183** |
| *$__unixEpochFrom()*                                  | 被当前活动开始时间取代为unix时间戳。例如，**1494410783**     |
| *$__unixEpochTo()*                                    | 当前活动结束时间取代为unix时间戳。例如，**1494410783**       |
| *$__unixEpochNanoFilter(dateColumn)*                  | 将纳秒时间戳作为时间替换为列名称的时间。例如，**dateColumn> = 1494410783152415214 AND dateColumn <= 1494497183142514872** |
| *$__unixEpochNanoFrom()*                              | 将活动开始时间替换为纳秒时间戳。例如，**1494410783152415214** |
| *$__unixEpochNanoTo()*                                | 将活动结束时间替换为unix时间戳。例如，**1494497183142514872** |
| *$__unixEpochGroup(dateColumn,‘5m’, [fillmode])*      | 与$ __ timeGroup相同，但存储为unix时间戳（仅在Grafana 5.3+中可用）。 |
| *$__unixEpochGroupAlias(dateColumn,‘5m’, [fillmode])* | 与上面相同，还添加了列别名（仅在Grafana 5.3+中可用）。       |

我们计划添加更多宏。如果您对想要查看的宏有什么建议，请在我们的GitHub仓库中[反馈](https://github.com/grafana/grafana)

## 表查询

如果`Format as`查询选项设置为`Table`那么您基本上可以执行任何类型的SQL查询。表格面板将自动显示查询返回的任何列和行的结果。

查询编辑器带有示例查询:

![](https://grafana.com/docs/img/docs/v46/postgres_table_query.png)

查询:

```sql
SELECT
  title as "Title",
  "user".login as "Created By",
  dashboard.created as "Created On"
FROM dashboard
INNER JOIN "user" on "user".id = dashboard.created_by
WHERE $__timeFilter(dashboard.created)
```



您可以使用常规选择语法`as`SQL来控制table panel列的名称

生成的表格面板：

![](https://grafana.com/docs/img/docs/v46/postgres_table.png)

## 时间序列查询

如果在面板使用中`Format as`设置为`Time series`，无论是SQL datetime或Unix时间类型,查询必须返回一个`time`名为列, 除了`time`和`metric`之外的任何列都被视为列值。您可以返回名为`metric`的列，该列用作列的度量标准名称。如果返回多个列值和一个名为metric的列，则此列用作系列名称的前缀（仅在Grafana 5.3+中可用）

时间序列查询的结果集需要按时间排序。

**`metric`列示例：**

```sql
SELECT
  $__timeGroup("time_date_time",'5m'),
  min("value_double"),
  'min' as metric
FROM test_data
WHERE $__timeFilter("time_date_time")
GROUP BY time
ORDER BY time
```

**使用$ __ timeGroup宏中的fill参数将null值转换为零的示例：**

```sql
SELECT
  $__timeGroup("createdAt",'5m',0),
  sum(value) as value,
  measurement
FROM test_data
WHERE
  $__timeFilter("createdAt")
GROUP BY time, measurement
ORDER BY time
```

**多列示例：**

```sql
SELECT
  $__timeGroup("time_date_time",'5m'),
  min("value_double") as "min_value",
  max("value_double") as "max_value"
FROM test_data
WHERE $__timeFilter("time_date_time")
GROUP BY time
ORDER BY time
```



## 模板

您可以在metric标准查询中使用变量代替硬编码服务器，应用程序和传感器名称等内容。变量显示为仪表板顶部的下拉选择框。这些下拉菜单可以轻松更改仪表板中显示的数据。

查看[模板](https://grafana.com/docs/reference/templating/)文档，了解模板功能和不同类型的模板变量。

## 查询变量

如果添加该类型的模板变量，则`Query`可以编写PostgreSQL查询，该查询可以返回显示为下拉选择框的测量名称，键名或键值等内容。

例如:

如果在模板变量查询设置中指定类似的查询，则可以在表中包含hostname列的所有值

``` sql
SELECT hostname FROM host
```

查询可以返回多个列，Grafana将自动从中创建列表。例如，下面的查询将返回一个包含from `hostname`和的值的列表`hostname2`

```sql
SELECT host.hostname, other_host.hostname2 FROM host JOIN other_host ON host.city = other_host.city
```

要`$__timeFilter(column)`在查询中使用与时间范围相关的宏，需要将模板变量的刷新模式设置为“ *开时间范围更改”*。

```sql
SELECT event_name FROM event_log WHERE $__timeFilter(time_column)
```

另一个选项是可以创建键/值变量的查询。该查询应返回两个名为`__text`和的列`__value`。`__text`列值应该是唯一的（如果不是唯一的，则第一个值会被录用）。下拉列表中的选项将包含一个文本和值，允许您将`友好名称作`为文本，将id作为值。`hostname`作为文本和`id`值的示例查询：

```sql
SELECT hostname AS __text, id AS __value FROM host
```

您还可以创建嵌套变量。使用名为变量的变量`region`，您可以让hosts变量仅显示当前所选区域中的主机，并使用这样的查询（如果`region`是多值变量，则使用`IN`比较运算符而不是`=`匹配多个值）：

```sql
SELECT hostname FROM host  WHERE region IN($region)
```

## 在查询中使用变量

从Grafana 4.3.0到4.6.0，模板变量总是自动引用。如果您的模板变量是字符串，请不要将它们用where子句中的引号括起来。

从Grafana 4.7.0开始，模板变量值仅在模板变量为a时引用`multi-value`。

如果变量是多值变量，则使用`IN`比较运算符而不是`=`匹配多个值。

有两种语法：

`$<varname>`名为的模板变量的示例`hostname`：

```sql
SELECT
  atimestamp as time,
  aint as value
FROM table
WHERE $__timeFilter(atimestamp) and hostname in($hostname)
ORDER BY atimestamp ASC
```

`[[varname]]`名为的模板变量的示例`hostname`：

```sql
SELECT
  atimestamp as time,
  aint as value
FROM table
WHERE $__timeFilter(atimestamp) and hostname in([[hostname]])
ORDER BY atimestamp ASC
```

### 禁用多值变量的引用

Grafana会自动为多值变量创建带引号的逗号分隔字符串。例如：如果`server01`与`server02`被选择那么它将被格式化为：`'server01', 'server02'`。要禁用引用，请对变量使用csv格式化选项：

```sql
${servers:csv}
```

在[Variables](https://grafana.com/docs/reference/templating/#advanced-formatting-options)文档中阅读有关变量格式选项的更多信息。

## 注释

[注释](https://grafana.com/docs/reference/annotations/)允许您在图表上叠加丰富的事件信息。您可以通过仪表板菜单/注释视图添加注释查询。

**使用带有纪元值的时间列的示例查询**

```sql
SELECT
  epoch_time as time,
  metric1 as text,
  concat_ws(', ', metric1::text, metric2::text) as tags
FROM
  public.test_data
WHERE
  $__unixEpochFilter(epoch_time)
```

**使用本机sql日期/时间数据类型的时间列的示例查询：**

```sql
SELECT
  native_date_time as time,
  metric1 as text,
  concat_ws(', ', metric1::text, metric2::text) as tags
FROM
  public.test_data
WHERE
  $__timeFilter(native_date_time)
```

| 名称 | 描述                                                         |
| :--- | :----------------------------------------------------------- |
| time | 日期/时间字段的名称。可以是具有本机sql日期/时间数据类型或纪元值的列。 |
| text | 事件描述字段。                                               |
| tags | 用于事件标记的可选字段名称，以逗号分隔的字符串形式。         |

## 警报

时间序列查询应该在警报条件下工作。警报规则条件中尚不支持表格式查询

现在可以使用Grafana的配置系统使用配置文件配置数据源。您可以在[配置文档页面](https://grafana.com/docs/administration/provisioning/#datasources)上阅读有关其工作原理以及可以为数据源设置的所有设置的更多信息

以下是此数据源的一些配置示例。

```yaml
apiVersion: 1

datasources:
  - name: Postgres
    type: postgres
    url: localhost:5432
    database: grafana
    user: grafana
    secureJsonData:
      password: "Password!"
    jsonData:
      sslmode: "disable" # disable/require/verify-ca/verify-full
      maxOpenConns: 0         # Grafana v5.4+
      maxIdleConns: 2         # Grafana v5.4+
      connMaxLifetime: 14400  # Grafana v5.4+
      postgresVersion: 903 # 903=9.3, 904=9.4, 905=9.5, 906=9.6, 1000=10
      timescaledb: false
```