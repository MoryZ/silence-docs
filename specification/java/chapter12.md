# （十二）异常处理

**1.【强制】Java 类库中定义的可以通过预检查方式规避的`RuntimeException`异常不应该通过 catch 的方式来处理，比如`NullPointerException`，`IndexOutOfBoundsException`等等。**

无法通过预检查的异常除外，比如在解析字符串形式的数字时，可能存在数字格式错误，不得不通过 catch `NumberFormatException`来实现。

正例：`if (obj != null) {…​}`

反例：`try { obj.method(); } catch (NullPointerException e) {…​}`

-   [Sonar-1696: "NullPointerException" should not be caught](https://rules.sonarsource.com/java/RSPEC-1696)

**2.【强制】异常捕获后不要用来做流程控制，条件控制。**

异常设计的初衷是解决程序运行中的各种意外情况，且异常的处理效率比条件判断方式要低很多。

**3.【强制】catch 时请分清稳定代码和非稳定代码，稳定代码指的是无论如何不会出错的代码。对于非稳定代码的 catch 尽可能进行区分异常类型，再做对应的异常处理。**

对大段代码进行 try-catch，使程序无法根据不同的异常做出正确的应激反应，也不利于定位问题，这是一种不负责任的表现。

正例：用户注册的场景中，如果用户输入非法字符，或用户名称已存在，或用户输入密码过于简单，在程序上作出分门别类的判断，并提示给用户。

**4.【强制】异常处理的原则：**

**4.1. 捕获异常一定要处理；如果故意捕获并忽略异常，须要注释写明原因。**

方便后面的阅读者知道，此处不是漏了处理。

反例：

    try {
    } catch(Exception e) {
    }

正例：

    try {
    } catch(Exception e) {
        // continue the loop
    }

**4.2. 异常处理不能吞掉原异常，要么在日志中打印，要么在重新抛出的异常中包含原异常。**

反例：

    throw new MyException("message");

正例：

    // 记录日志后抛出新异常，向上次调用者屏蔽底层异常
    logger.error("message", ex);
    throw new MyException("message");

    // 传递底层异常
    throw new MyException("message", ex);

-   [Sonar-1166: Exception handlers should preserve the original exceptions](https://rules.sonarsource.com/java/RSPEC-1166)，其中默认包含了`InterruptedException`, `NumberFormatException`，`NoSuchMethodException`等若干例外

**4.3. 如果不想处理异常，可以不进行捕获。但最外层的业务使用者，必须处理异常，将其转化为用户可以理解的内容。**

在大部分场景下，除非很明确的知道异常应该怎么处理（如某些异常可以忽略，则可以捕获后什么都不做仅添加注释说明），应用层都不应该捕获异常，而是直接抛出到最上层，由框架统一处理，比如 Spring MVC / RPC / 定时任务 / MQ 等，禁止吞掉异常或仅仅只是打印日志，应该由框架统一进行日志打印处理。

对于 Checked Exception，可以封装成`java.lang.reflect.UndeclaredThrowableException`继续往上抛；对于`IOException`，可以使用 JDK8 后新增加的`UncheckedIOException`进行封装后继续往上抛。

**5.【强制】`finally`块的处理原则：**

**5.1. 必须对资源对象、流对象进行关闭，统一使用语法 try-with-resource，禁止使用在`finally`块中关闭`Closeable`的写法。**

正例：

    try (Writer writer = new StringWriter()) { // JDK 7
        writer.append(content);
    }

    Writer writer = new StringWriter(); // JDK 11
    try (writer) {
        writer.append(content);
    }

反例：

    Writer writer = new StringWriter();
    try {
        writer.append(content);
    } finally {
        if (writer != null) {
            writer.close();
        }
    }

**5.2. 如果处理过程中有抛出异常的可能，也要做 try-catch，否则`finally`块中抛出的异常，将代替`try`块中抛出的异常。**

反例：

    try {
        ...
        throw new TimeoutException();
    } finally {
        file.close(); // 如果file.close()抛出IOException, 将代替TimeoutException
    }

正例：

    // 在 finally 块中 try－catch
    try {
        ...
        throw new TimeoutException();
    } finally {
        IOUtils.closeQuietly(file); // 该方法中对所有异常进行了捕获
    }

-   [Sonar-1163: Exceptions should not be thrown in finally blocks](https://rules.sonarsource.com/java/RSPEC-1163)

**5.3. 不要在`finally`块中使用`return`。**

`try`块中的`return`语句执行成功后，并不马上返回，而是继续执行`finally`块中的语句，如果此处存在`return`语句，则会在此直接返回，无情丢弃掉`try`块中的返回点。

反例：

    private int x = 0;
    public int checkReturn() {
        try {
            // x 等于 1，此处不返回
            return ++x;
        } finally {
            // 返回的结果是 2
            return ++x;
        }
    }

-   [Sonar-1143: Jump statements should not occur in "finally" block](https://rules.sonarsource.com/java/RSPEC-1143)

**6.【强制】统一使用框架中的`EhisException`或其子类，也可以自定义异常类继承该类，禁止自定义异常类继承`Exception`或`RuntimeException`。**

关于是否使用受查异常的争论，详见《Clean Code》，争论已经结束，不再推荐原本初衷很好的 Checked Exception。

因为需要在抛出异常的地方，与捕获处理异常的地方之间，层层定义`throws XXX`来传递`Exception`，如果底层代码改动，将影响所有上层函数的签名，导致编译出错，对封装的破坏严重。对的处理也给上层程序员带来了额外的负担。因此其他语言都没有的设计。

**7.【强制】事务场景中，抛出异常被 catch 后，如果需要回滚，一定要注意手动回滚事务。**

**8.【推荐】在特定场景，避免每次构造异常。**

异常的构造函数需要获得整个调用栈。如果异常频繁发生，且不需要打印完整的调用栈时，可以考虑绕过异常的构造函数。

1） 如果异常的 message 不变，将异常定义为静态成员变量。

下例定义静态异常，并简单定义一层的 StackTrace。

正例：

    private static RuntimeException TIMEOUT_EXCEPTION = ExceptionUtils.setStackTrace(new RuntimeException("Timeout"),
    MyClass.class, "mymethod");

    ...

    throw TIMEOUT_EXCEPTION;

2） 如果异常的 message 会变化，则对静态的异常实例进行`clone()`再修改 message。

`Exception`默认不是`Cloneable`的。

正例：

    private static CloneableException TIMEOUT_EXCEPTION = new CloneableException("Timeout") .setStackTrace(My.class,
     "hello");

    ...

    throw TIMEOUT_EXCEPTION.clone("Timeout for 40ms");

3）自定义异常，也可以考虑重载`fillStackTrace()`为空函数，但相对没那么灵活，比如无法按场景指定一层的 StackTrace。

**9.【推荐】异常日志应包含排查问题的足够信息。**

异常信息应包含排查问题时足够的上下文信息。

捕获异常并记录异常日志的地方，同样需要记录没有包含在异常信息中，而排查问题需要的信息，比如捕获处的上下文信息。

反例：

    new TimeoutException("timeout");
    logger.error(e.getMessage(), e);

正例：

    new TimeoutException("timeout:" + eclapsedTime + ", configuration:" + configTime);
    logger.error("user[" + userId + "] expired:" + e.getMessage(), e);

-   Facebook-Contrib: Style - Method throws exception with static message string

**10.【推荐】多个异常的处理逻辑一致时，使用JDK7的语法避免重复代码。**

特别注意 JDK7 后引入了`ReflectiveOperationException`和`GeneralSecurityException`分别作为反射和安全相关异常类的父类，由于这两类异常都是 Checked Exception，故捕获的时候可直接捕获这两个父类异常。

正例：

    try {
        ...
    } catch (AException | BException | CException ex) {
        handleException(ex);
    }

    try {
        ...
    } catch (ReflectiveOperationException ex) {
        handleException(ex);
    }

反例：

    try {
        ...
    } catch (IllegalAccessException | InvocationTargetException | NoSuchMethodException ex) {
        handleException(ex);
    }

-   [Sonar-2147: Catches should be combined](https://rules.sonarsource.com/java/RSPEC-2147)
