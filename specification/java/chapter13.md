# （十三）日志规约

**1.【强制】应用中不可直接使用日志库（Log4j、Logback）中的API，而应依赖使用日志框架(SLF4J、JCL—Jakarta Commons Logging)中的 API，使用门面模式的日志框架，有利于维护和各个类的日志处理方式统一。**

    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;

    private static final Logger logger = LoggerFactory.getLogger(Foo.class);

**2.【强制】应用中的扩展日志（如打点、临时监控、访问日志等）命名方式：**

appName\_logType\_logName.log。

-   logType：日志类型，如 stats / monitor / access 等

-   logName：日志描述。

-   这种命名的好处：通过文件名就可知道日志文件属于什么应用，什么类型，什么目的，也有利于归类查找。

**3.【强制】对不确定会否输出的日志，采用占位符或条件判断的方式。**

正例：

    logger.debug("Processing trade with id: {} symbol : {} ", id, symbol);

但如果`symbol.getMessage()`本身是个消耗较大的动作，占位符在此时并没有帮助，须要改为条件判断方式来完全避免它的执行。

反例：

    logger.debug("Processing trade with id: {} symbol : {} ", id, symbol.getMessage());

正例：

    if (logger.isDebugEnabled()) {
        logger.debug("Processing trade with id: " + id + " symbol: " + symbol.getMessage());
    }

**4.【强制】对确定输出，而且频繁输出的日志，采用直接拼装字符串的方式。**

如果这是一条 WARN，ERROR 级别的日志，或者确定输出的 INFO 级别的业务日志，直接用字符串拼接，要比使用占位符替换更加高效。

Slf4j 的占位符并没有魔法，每次输出日志都要进行占位符的查找，字符串的切割与重新拼接。

正例：

    // RIGHT
    logger.info("I am a business log with id: " + id + " symbol: " + symbol);

    // RIGHT
    logger.warn("Processing trade with id: " + id + " symbol: " + symbol);

**5.【强制】尽量使用异步日志。**

低延时的应用，使用异步输出的形式（以AsyncAppender串接真正的Appender），可减少IO造成的停顿。

需要正确配置异步队列长度及队列满的行为，是丢弃还是等待可用，业务上允许丢弃的尽量选丢弃。

**6.【强制】避免重复打印日志，浪费磁盘空间，务必在日志配置文件中设置`additivity=false`。**

正例：`<logger name="com.taobao.dubbo.config" additivity="false">`

**7.【强制】禁止使用`System.out`或`System.err`输出或使用`e.printStackTrace()`打印异常堆栈。**

标准日志输出与标准错误输出文件每次 JBoss 重启时才滚动，如果大量输出送往这两个文件，容易造成文件大小超过操作系统大小限制。

例外: 应用启动和关闭时，担心日志框架还未初始化或已关闭。

-   [Sonar-106: Standard outputs should not be used directly to log anything](https://rules.sonarsource.com/java/RSPEC-106)

-   [Sonar-1148: Throwable.printStackTrace(…​) should not be called](https://rules.sonarsource.com/java/RSPEC-1148)

**8.【强制】日志打印时禁止直接用 JSON 工具将对象转换成`String`。**

如果对象里某些 get 方法被覆写，存在抛出异常的情况，则可能会因为打印日志而影响正常业务流程的执行。

正例：打印日志时仅打印出业务相关属性值或者调用其对象的`toString()`方法。

**9.【强制】谨慎地记录日志。生产环境禁止输出 debug 日志；有选择地输出 info 日志；如果使用 warn 来记录刚上线时的业务行为信息，一定要注意日志输出量的问题，避免把服务器磁盘撑爆，并记得及时删除这些观察日志。**

大量地输出无效日志，不利于系统性能，也不利于快速定位错误点。

记录日志时请思考：这些日志真的有人看吗？看到这条日志你能做什么？能不能给问题排查带来好处？

**10.【强制】可以使用 warn 日志级别来记录用户输入参数错误的情况，避免用户投诉时，无所适从。如非必要，请不要在此场景打出 error 级别，避免频繁报警。**

注意日志输出的级别，error 级别只记录系统逻辑出错、异常或者重要的错误信息。

**11.【推荐】禁止配置日志框架输出日志打印处的类名，方法名及行号的信息。**

日志框架在每次打印时，通过主动获得当前线程的 StackTrace 来获取上述信息的消耗非常大，尽量通过 Logger 名本身给出足够信息。

**12.【推荐】尽量用英文来描述日志错误信息，如果日志中的错误信息用英文描述不清楚的话使用中文描述即可，否则容易产生歧义。**

国际化团队或海外部署的服务器由于字符集问题，使用全英文来注释和描述日志错误信息。

**13.【推荐】为了保护用户隐私，日志文件中的用户敏感信息需要进行脱敏处理。**
