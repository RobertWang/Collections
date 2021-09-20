> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7008744306823397384) const today = new Date(); const dd = String(today.getDate()).padStart(2, '0'); // 日 const mm = String(today.getMonth() + 1).padStart(2, '0'); // 月 const yyyy = today.getFullYear(); // 年 const curDate = `${yyyy}-${mm}-${dd}` console.log(curDate) // 输出: 2021-09-17 复制代码

在 day.js 中我们只需这样，当然不止这样，还支持很多功能。

```
import dayjs from "dayjs";

const curDate = dayjs().format('YYYY-MM-DD');

console.log(curDate)
// 输出: 2021-09-17
复制代码
```

Day.js 例子
---------

现在我们来看一些实用、有趣的例子，与原生 API 相比，它更加简单，而且可读性更强。

#### 1. 获取两个日期相差的天数

[查看文档](https://link.juejin.cn?target=https%3A%2F%2Fday.js.org%2Fdocs%2Fen%2Fdisplay%2Fdifference "https://day.js.org/docs/en/display/difference")

```
import dayjs from "dayjs";

// 第二个参数指定为'day'代表以日为颗粒度
dayjs(new Date(2021, 10, 1)).diff(new Date(2021, 9, 17), "day"); 
// 输出: 15
复制代码
```

#### 2. 检查日期是否合法

[查看文档](https://link.juejin.cn?target=https%3A%2F%2Fday.js.org%2Fdocs%2Fen%2Fparse%2Fis-valid "https://day.js.org/docs/en/parse/is-valid")

```
import dayjs from "dayjs";

dayjs("20").isValid(); 
// 输出:  false
dayjs("2021-09-17").isValid(); 
// 输出:  true
复制代码
```

#### 3. 获取输入日期月份的天数

[查看文档](https://link.juejin.cn?target=https%3A%2F%2Fday.js.org%2Fdocs%2Fen%2Fdisplay%2Fdays-in-month "https://day.js.org/docs/en/display/days-in-month")

```
import dayjs from "dayjs";

dayjs("2021-09-13").daysInMonth() 
// 输出: 30
复制代码
```

#### 4. 添加日、月、年、时、分、秒

[查看文档](https://link.juejin.cn?target=https%3A%2F%2Fday.js.org%2Fdocs%2Fen%2Fmanipulate%2Fadd "https://day.js.org/docs/en/manipulate/add")

```
import dayjs from "dayjs";

dayjs("2021-09-17 08:10:00").add(20, "minute").format('YYYY-MM-DD HH:mm:ss') 
// 输出: 2021-09-17 08:30:00
复制代码
```

#### 5. 减去日、月、年、时、分、秒

[查看文档](https://link.juejin.cn?target=https%3A%2F%2Fday.js.org%2Fdocs%2Fen%2Fmanipulate%2Fsubtract "https://day.js.org/docs/en/manipulate/subtract")

```
import dayjs from "dayjs";

dayjs("2021-09-17 08:10:00").subtract(20, "minute").format('YYYY-MM-DD HH:mm:ss')
// 输出: 2021-09-17 07:50:00
复制代码
```

使用插件来扩展功能
---------

#### 1. RelativeTime

[查看文档](https://link.juejin.cn?target=https%3A%2F%2Fday.js.org%2Fdocs%2Fen%2Fplugin%2Frelative-time "https://day.js.org/docs/en/plugin/relative-time")

获取指定时间到现在的时间差。

```
import dayjs from "dayjs";
import relativeTime from "dayjs/plugin/relativeTime";

dayjs.extend(relativeTime);

dayjs("2021-09-16 13:28:55").fromNow();
// 输出: 9 hours ago
复制代码
```

下面是所有的输出表

<table><thead><tr><th>Range</th><th>Key</th><th>Sample Output</th></tr></thead><tbody><tr><td>0 to 44 秒</td><td>s</td><td>a few seconds ago</td></tr><tr><td>45 to 89 秒</td><td>m</td><td>a minute ago</td></tr><tr><td>90 秒 to 44 分钟</td><td>mm</td><td>2 minutes ago ... 44 minutes ago</td></tr><tr><td>45 to 89 分钟</td><td>h</td><td>an hour ago</td></tr><tr><td>90 分钟 to 21 小时</td><td>hh</td><td>2 hours ago ... 21 hours ago</td></tr><tr><td>22 to 35 小时</td><td>d</td><td>a day ago</td></tr><tr><td>36 小时 to 25 天</td><td>dd</td><td>2 days ago ... 25 days ago</td></tr><tr><td>26 to 45 天</td><td>M</td><td>a month ago</td></tr><tr><td>46 天 to 10 月</td><td>MM</td><td>2 months ago ... 10 months ago</td></tr><tr><td>11 月 to 17 月</td><td>y</td><td>a year ago</td></tr><tr><td>18 月 +</td><td>yy</td><td>2 years ago ... 20 years ago</td></tr></tbody></table>

#### 2. WeekOfYear

[查看文档](https://link.juejin.cn?target=https%3A%2F%2Fday.js.org%2Fdocs%2Fen%2Fplugin%2Fweek-of-year "https://day.js.org/docs/en/plugin/week-of-year")

获取指定日期是当年的第几周

```
import dayjs from "dayjs";
import weekOfYear from "dayjs/plugin/weekOfYear";

dayjs.extend(weekOfYear);

dayjs("2021-09-13 14:00:00").week(); 
// 输出: 38
复制代码
```

#### 3. IsSameOrAfter

[查看文档](https://link.juejin.cn?target=https%3A%2F%2Fday.js.org%2Fdocs%2Fen%2Fplugin%2Fis-same-or-after "https://day.js.org/docs/en/plugin/is-same-or-after")

检查一个日期是否等于或者大于一个日期

```
import dayjs from "dayjs";
import isSameOrAfter from "dayjs/plugin/isSameOrAfter";

dayjs.extend(isSameOrAfter);

dayjs("2021-09-17").isSameOrAfter("2021-09-16"); 
// 输出: true
复制代码
```

#### 4. MinMax

[查看文档](https://link.juejin.cn?target=https%3A%2F%2Fday.js.org%2Fdocs%2Fen%2Fplugin%2Fmin-max "https://day.js.org/docs/en/plugin/min-max")

获取数组中最大的日期，或者最小的日期

```
import dayjs from "dayjs";
import minMax from "dayjs/plugin/minMax";

dayjs.extend(minMax)

const maxDate = dayjs.max([
    dayjs("2021-09-13"), 
    dayjs("2021-09-16"), 
    dayjs("2021-09-20")
])

const minDate = dayjs.min([
    dayjs("2021-09-13"), 
    dayjs("2021-09-16"), 
    dayjs("2021-09-20")
])

maxDate.format('YYYY-MM-DD HH:mm:ss') 
// 输出: 2021-09-20 00:00:00
minDate.format('YYYY-MM-DD HH:mm:ss') 
// 输出: 2021-09-13 00:00:00
复制代码
```

#### 5. IsBetween

[查看文档](https://link.juejin.cn?target=https%3A%2F%2Fday.js.org%2Fdocs%2Fen%2Fplugin%2Fadvanced-format "https://day.js.org/docs/en/plugin/advanced-format")

检查指定日期是否在指定的日期范围内

```
import dayjs from "dayjs";
import isBetween from "dayjs/plugin/isBetween";

dayjs.extend(isBetween);

// 使用日为颗粒度进行比较
dayjs("2010-10-21").isBetween(dayjs("2010-10-20"), dayjs("2010-10-25"), "day");
// 输出: true

// 使用年为颗粒度进行比较
dayjs("2010-10-21").isBetween(dayjs("2010-10-20"), dayjs("2010-10-25"), "year");
// 输出: false
复制代码
```

唠叨
--

如果本文对你有用，您的点赞、评论、关注是对我最大的鼓励 O(∩_∩)O👍👍👍