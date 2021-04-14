# Cron 表达式

cron 表达式是以下格式的字符串：

```bash
<seconds> <minutes> <hours> <day_of_month> <month> <day_of_week> [year]
```

Elasticsearch 使用来自 [Quartz 任务调度器](https://quartz-scheduler.org/)的 cron 解析器。有关编写 Quartz cron 表达式的更多信息，参阅 [Quartz CronTrigger 教程](http://www.quartz-scheduler.org/documentation/quartz-2.3.0/tutorials/crontrigger.html)。

?> 你可以使用 [elasticsearch-croneval](/command_line_tools/elasticsearch-croneval) 命令行工具校验你的 cron 表达式。

## Cron 表达式元素

除了 `year`（年），所有元素都是必需的。有关允许的特殊字符的信息，参阅 [Cron 特殊字符](/rest_apis/api_convention/cron_expressions?id=Cron-特殊字符)。

- `<seconds>`
（必需的） 有效值： `0`-`59` 以及特殊字符 `,` `-` `*` `/`
- `<minutes>`
（必需的） 有效值： `0`-`59` 以及特殊字符 `,` `-` `*` `/`
- `<hours>`
（必需的） 有效值： `0`-`23` 以及特殊字符 `,` `-` `*` `/`
- `<day_of_month>`
（必需的） 有效值： `1`-`31` 以及特殊字符 `,` `-` `*` `/` `?` `L` `W`
- `<month>`
（必需的） 有效值： `1`-`12`， `JAN`-`DEC`， `jan`-`dec`， 以及特殊字符 `,` `-` `*` `/`
- `<day_of_week>`
（必需的） 有效值： `1`-`7`， `SUN`-`SAT`， `sun`-`sat`， 以及特殊字符 `,` `-` `*` `/` `?` `L` `#`
- `<year>`
（可选的） 有效值： `1970`-`2099` 以及特殊字符 `,` `-` `*` `/`

## Cron 特殊字符

- `*`

为字段选择所有可能的值。例如，在 `hours`（小时）字段中 `*` 表示“每个小时”。

- `?`

没有特定值。当你不在意值是什么时使用。例如，如果你希望计划在一个月里的特定天触发，但不关心是一周中的哪一天，你可以在 `day_of_week` 字段中指定 `?`。

- `-`

范围值（包含）。用于分隔最小值和最大值。例如，如果你希望计划在上午 9:00 到下午 5:00 之间每个小时触发，你可以在 `hours` 字段指定 `9-17`。

- `,`

多个值。用于字段中分隔多个值。例如，如果你希望计划在每周二和每周四触发，你可以在 `day_of_week` 字段指定 `TUE,THU`。

- `/`

增量。用于当指定时间增量时分隔值。第一个值表示起点，第二个值表示间隔值。例如，如果你希望计划在一个小时里每 20 分钟触发一次，你可以在 `minutes` 字段指定 `0/20`。类似的，在 `day_of_month` 字段指定 `1/5`，将从每月的第一天开始每 5 天执行一次。

- `L`

最后。在 `day_of_month` 字段中使用，意味着这个月的最后一天——1 月时是 31 日，不闰年时 2 月时是 28 日，4 月时是 30 日，等等。在 `day_of_week` 字段中单独使用以替代 `7` 或 `SAT`，或者在一周中的某一特定日期后，选择该类型在月中最后一天。例如，`6L` 代表一个月的最后一个周五。你可以在 `day_of_month` 字段指定 `LW`，用于指定该月的最后一个工作日。当指定值的列表或范围时，避免使用 `L` 选项，因为结果可能不是你期待的结果。

- `W`

工作日。用于指定靠近指定日期的工作日（星期一到星期五）。例如，如果你在 `day_of_month` 字段指定 `15W`，且 15 号是星期六，计划将在 14 号触发。如果 15 号是星期天，计划将在 16 号星期一触发。如果 15 号是星期二，计划会在 15 号星期二触发。然而，如果你为 `day_of_month` 指定 `1W`，且 1 号是星期六，计划会在 3 号星期一触发——它不会跳过月的范围。你可以在 `day_of_month` 字段指定 `LW`，用于指定一个月的最后一个工作日。当 `day_of_month` 是单独的一天，你才能使用 `W` 选项——当指定的是一个范围或列表天数时，它是无效的。

- `#`

一个月里的第XXX天。在 `day_of_week` 字段用于指定一个月的第XXX天。例如，如果你指定 `6#1`，计划会在一个月的第一个星期五触发。注意你指定 `3#5`，且在某一个月里没有 5 个星期二，计划在那个月不会触发。

## 例子

### 设置每日触发器

- `0 5 9 * * ?`

在每天上午 UTC 9:05 触发。

- `0 5 9 * * ? 2020`

在 2020 年的每天上午 UTC 9:05 触发。

### 限制触发器为天数或时间范围

- `0 5 9 ? * MON-FRI`

在星期一到星期五的每天上午 UTC 9：05 触发。

- `0 0-5 9 * * ?`

在每天上午 9:00 UTC 到 9:05 UTC 的每分钟触发。

### 设置周期触发器

- `0 0/15 9 * * ?`

在每天上午 9:00 UTC 到 9:45 UTC 每 15 分钟触发。

- `0 5 9 1/3 * ?`

从一个月的第一天开始，每 3 天在上午 9:05 UTC 触发。

### 设置特定日期触发的计划

- `0 1 4 1 4 ?`

在 4 月 1 日上午 4:01 UTC 触发。

- `0 0,30 9 ? 4 WED`

在 4 月的每个星期三的上午 9:00 UTC 和 9:30 UTC 触发。

- `0 5 9 15 * ?`

在每个月 15 号的上午 9:05 UTC 触发。

- `0 5 9 15W * ?`

在每个月的 15 号最近的工作日的上午 9:05 UTC 触发。

- `0 5 9 ? * 6#1`

在每个月的第一个星期一的上午 9:05 UTC 触发。

### 使用 last（最后）设置触发器

- `0 5 9 L * ?`

在每个月的最后一天的上午 9:05 UTC 触发。

- `0 5 9 ? * 2L`

在每个月的最后一个星期一的上午 9：05 UTC 触发。

- `0 5 9 LW * ?`

在每个月的最后一个工作日的上午 9:05 UTC 触发。

> [原文链接](https://www.elastic.co/guide/en/elasticsearch/reference/current/cron-expressions.html)