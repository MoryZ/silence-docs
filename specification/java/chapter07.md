# （七）日期时间

**1.【强制】除需要兼容老代码或老 API，否则禁止使用老的`Date`、`Calendar`、`SimpleDateFormat`等日期类和 API，统一使用 JDK8 引入的`java.time`包下的类和 API 。**

可以使用`Instant`代替`Date`，`ZonedDateTime`代替`Calendar`，`DateTimeFormatter`代替`SimpleDateFormat`等，官方给出的解释：simple beautiful strong immutable thread-safe。

正例：

    Instant now = Instant.now();

反例：

    Date now = new Date();

**2.【强制】日期格式化时，传入 pattern 中表示年份统一使用小写的 y。**

日期格式化时，`yyyy`表示当天所在的年，而大写的`YYYY`代表是 week in which year（JDK7 之后引入的概念），意思是当天所在的周属于的年份，一周从周日开始，周六结束，只要本周跨年，返回的`YYYY`就是下一年。

正例：表示日期和时间的格式如下所示：

    DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");

反例：某程序员因使用`YYYY/MM/dd`进行日期格式化，2017/12/31 执行结果为 2018/12/31，造成线上故障。

**3.【强制】在日期格式中分清楚大写的 M 和小写的 m，大写的 H 和小写的 h 分别指代的意义。**

日期格式中的这两对字母表意如下：

1.  表示月份是大写的 M

2.  表示分钟则是小写的 m

3.  24 小时制的是大写的 H

4.  12 小时制的则是小写的 h

**4.【强制】获取当前毫秒数：`System.currentTimeMillis()`，而不是`Instant.now().toEpochMilli()`。**

获取纳秒级时间，则使用`System.nanoTime`方法，请注意，该方法返回的并不是当前时间的纳秒数，而是机器从启动到现在流逝的纳秒数。

如果希望在测试用例中控制当前时间的值，则使用 JDK8 中的`Clock`，在测试和生产环境中使用不同的实现。

**5.【强制】禁止在程序中写死一年为 365 天，避免在公历闰年时出现日期转换错误或程序逻辑错误。**

正例：

    int daysOfYear = Year.of(2023).length();

反例：

    // 第一种情况：在闰年 366 天时，出现数组越界异常
    int[] dayArray = new int[365];
    // 第二种情况： 一年有效期的会员制，2020 年 1 月 26 日注册，硬编码 365 返回的却是 2021 年 1 月 25 日
    Calendar calendar = Calendar.getInstance();
    calendar.set(2020, 1, 26);
    calendar.add(Calendar.DATE, 365);

**6.【推荐】避免公历闰年 2 月问题。闰年的 2 月份有 29 天，一年后的那一天不可能是 2 月 29 日。**
