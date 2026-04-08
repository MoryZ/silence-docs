# （六）基本类型与字符串

**1.【强制】关于基本数据类型与包装数据类型的使用标准如下：**

1.  所有的 POJO 类属性必须使用包装数据类型。

2.  RPC 方法的返回值和参数必须使用包装数据类型。

3.  局部变量尽量使用基本数据类型。

POJO 类属性没有初值是提醒使用者在需要使用时，必须自己显式地进行赋值，任何 NPE 问题，或者入库检查，都由使用者来保证。

正例：数据库的查询结果可能是`null`，因为自动拆箱，用基本数据类型接收有 NPE 风险。

反例：某业务的交易报表上显示成交总额涨跌情况，即正负 x%，x 为基本数据类型，调用的 RPC 服务，调用不成功时，返回的是默认值，页面显示为 0%，这是不合理的，应该显示成中划线-。所以包装数据类型的`null`值，能够表示额外的信息，如：远程调用失败，异常退出。

**2.【强制】数值equals比较的原则：**

**2.1. 所有包装类对象之间值的比较，全部使用`equals`方法比较。**

== 判断对象是否同一个。`Integer var = ?`在缓存区间的赋值，会复用已有对象，因此这个区间内的`Integer`使用 == 进行判断可通过，但是区间之外的所有数据，则会在堆上新产生，不会通过。因此如果用 == 来比较数值，很可能在小的测试数据中通过，而到了生产环境才出问题。

**2.2. `BigDecimal`需要使用`compareTo()`方法比较。**

因为`BigDecimal`的`equals()`还会比对精度，2.0 与 2.00 不一致。

-   Facebook-Contrib: Correctness - Method calls BigDecimal.equals()

**2.3. Atomic 系列，不能使用`equals`方法。**

因为 Atomic\* 系列没有覆写`equals`方法。

正例：

    if (counter1.get() == counter2.get()) {
        ...
    }

-   [Sonar-2204: ".equals()" should not be used to test the values of "Atomic" classes](https://rules.sonarsource.com/java/RSPEC-2204)

**2.4. `double`及`float`的比较，要特殊处理。**

因为精度问题，浮点数间的`equals`非常不可靠。

    // RIGHT
    float f1 = 0.15F;
    float f2 = 0.45F / 3; // 实际等于0.14999999

    // WRONG
    if (f1 == f2) {...}
    if (Double.compare(f1, f2) == 0) {...}

    // RIGHT
    static final float EPSILON = 0.00001f;
    if (Math.abs(f1 - f2) < EPSILON) {...}

-   [Sonar-1244: Floating point numbers should not be tested for equality](https://rules.sonarsource.com/java/RSPEC-1244)

**3.【强制】数字类型的计算原则：**

**3.1. 数字运算表达式，因为先进行等式右边的运算，再赋值给等式左边的变量，所以等式两边的类型要一致。**

例子1：`int`与`int`相除后，哪怕被赋值给`float`或`double`，结果仍然是四舍五入取整的`int`。

需要强制将除数或被除数转换为`float`或`double`。

    double d = 24 / 7;  // 结果是3.0
    double d =  (double) 24 / 7; // 结果是正确的3.42857

例子2： `int`与`int`相乘，哪怕被赋值给`long`，仍然会溢出。

需要强制将乘数的一方转换为`long`。

    long l = Integer.MAX_VALUE * 2; // 结果是溢出的－2
    long l = Integer.MAX_VALUE * 2L; // 结果是正确的4294967294

另外，int的最大值约21亿，留意可能溢出的情况。

-   [Sonar-2184: Math operands should be cast before assignment](https://rules.sonarsource.com/java/RSPEC-2184)

**3.2. 数字取模的结果不一定是正数，负数取模的结果仍然负数。**

取模做数组下标时，如果不处理负数的情况，很容易`ArrayIndexOutOfBoundException`。

另外，`Integer.MIN_VALUE`取绝对值也仍然是负数。

    -4 % 3  = -1;
    Math.abs(Integer.MIN_VALUE) = -2147483648;

-   Findbugs: Style - Remainder of hashCode could be negative

**3.3. 浮点数之间的等值判断，基本数据类型不能使用 == 进行比较，包装数据类型不能使用 `equals`进行判断。**

浮点数采用“尾数+阶码”的编码方式，类似于科学计数法的“有效数字+指数”的表示方式。二进制无法精确表示大部分的十进制小数。

反例：

    float a = 1.0F - 0.9F;
    float b = 0.9F - 0.8F;
    if (a == b) {
        // 预期进入此代码块，执行其它业务逻辑
        // 但事实上 a == b 的结果为 false
    }
    Float x = Float.valueOf(a);
    Float y = Float.valueOf(b);
    if (x.equals(y)) {
        // 预期进入此代码块，执行其它业务逻辑
        // 但事实上 equals 的结果为 false
    }

\(1\) 指定一个误差范围，两个浮点数的差值在此范围之内，则认为是相等的。

正例：

    float a = 1.0F - 0.9F;
    float b = 0.9F - 0.8F;
    float diff = 1e-6F;
    if (Math.abs(a - b) < diff) {
        System.out.println("true");
    }

\(2\) 使用`BigDecimal`来定义值，再进行浮点数的运算操作。

正例：

    BigDecimal a = new BigDecimal("1.0");
    BigDecimal b = new BigDecimal("0.9");
    BigDecimal c = new BigDecimal("0.8");
    BigDecimal x = a.subtract(b);
    BigDecimal y = b.subtract(c);
    if (x.compareTo(y) == 0) {
        System.out.println("true");
    }

在预知小数精度的情况下，将浮点运算放大为整数计数，比如货币以“分”而不是以“元”计算，参考第 5 条。

-   [Sonar-2164: Math should not be performed on floats](https://rules.sonarsource.com/java/RSPEC-2164)

**4.【强制】禁止使用构造方法`BigDecimal(double)`的方式把`double`值转化为`BigDecimal`对象。**

`BigDecimal(double)`存在精度损失风险，在精确计算或值比较的场景中可能会导致业务逻辑异常。如：`BigDecimal g = new BigDecimal(0.1F)；`实际的存储值为：0.100000001490116119384765625。

正例：优先推荐入参为`String`的构造方法，或使用`BigDecimal`的`valueOf`方法，此方法内部其实执行了`Double`的`toString`，而`Double`的`toString`按`double`的实际能表达的精度对尾数进行了截断。

    BigDecimal recommend1 = new BigDecimal("0.1");
    BigDecimal recommend2 = BigDecimal.valueOf(0.1);

**5.【强制】任何货币金额，均以*最小货币单位*且为整型类型进行存储。**

**6. 字符串拼接的原则：**

**6.1.【强制】当字符串拼接不在一个命令行内写完，而是存在多次拼接时（比如循环），使用`StringBuilder.append()`。**

    String s = "hello" + str1 +  str2; // 基本 OK ，除非初始长度有问题，见第 6.4 点

    // WRONG
    String s = "hello";
    if (condition) {
        s += str1;
    }

    // WRONG
    String str = "start";
    for (int i = 0; i < 100; i++) {
        str = str + "hello";
    }

反编译出的字节码文件显示，每条用`+`进行字符拼接的语句，都会`new`出一个`StringBuilder`对象，然后进行`append`操作，最后通过`toString`方法返回`String`对象。所以上面两个错误例子，会重复构造`StringBuilder`，重复`toString()`造成资源浪费。

-   [Sonar-1643: Strings should not be concatenated using '+' in a loop](https://rules.sonarsource.com/java/RSPEC-1643)

**6.2.【强制】字符串拼接对象时，不要显式调用对象的`toString()`方法。**

如上，`+`实际是`StringBuilder`，本身会调用对象的`toString()`，且能很好的处理`null`的情况，当然有时候也要注意输出`null`字符串也不一定是预期希望得到的结果，还是要显式的处理`null`值。

反例：

    str = "result:" + myObject.toString();  // myObject 为``NULL``时，抛 NPE

正例：

    str = "result:" + myObject;  // myObject 为``NULL``时，输出 result:null

**6.3.【强制】使用`StringBuilder`，而不是有所有方法都有同步修饰符的`StringBuffer`。**

因为内联不成功，逃逸分析并不能抹除`StringBuffer`上的同步修饰符。

-   [Sonar-1149: Synchronized classes Vector, Hashtable, Stack and StringBuffer should not be used](https://rules.sonarsource.com/java/RSPEC-1149)

**6.4.【推荐】当拼接后字符串的长度远大于 16 时，指定`StringBuilder`的大概长度，避免容量不足时的成倍扩展。**

**6.5.【推荐】如果字符串长度很大且频繁拼接，可考虑`ThreadLocal`重用`StringBuilder`对象**

参考`BigDecimal`的`toString()`实现。

**7.【强制】字符操作时，优先使用字符参数，而不是字符串，能提升性能。**

反例：

    str.indexOf("e");

    stringBuilder.append('a');
    str.indexOf('e');
    str.replace('m', 'z');

其他像`split`等方法，若在JDK `String`中未提供针对字符参数的方法，可考虑使用 Apache Commons 的`StringUtils`。

-   [Sonar-3027: String function use should be optimized for single characters](https://rules.sonarsource.com/java/RSPEC-3027)

**8.【强制】利用好正则表达式的预编译功能，可以有效加快正则匹配速度。**

正例：

    // 在某个地方预先编译 Pattern ，比如类的静态变量
    private static Pattern pattern = Pattern.compile("[a-zA-z]");
    ...
    // 真正使用 Pattern 的地方
    result = pattern.matcher("abc").matches();

反例：

    // 直接使用 String 的 matches() 方法
    result = "abc".matches("[a-zA-z]");

    // 每次重新构造 Pattern
    Pattern pattern = Pattern.compile("[a-zA-z]");
    result = pattern.matcher("abc").matches();

**9.【强制】禁止使用`+`将`int`等数字类型转换成`String`类型。**

-   如果是基本类型，使用`Integer.toString()`静态方法。

-   如果是`Object`类型转`String`，有以下三种方法：

    -   直接使用对象上的`toString()`方法（非静态方法），注意该方法可能产生 NPE 异常。

    -   使用`String.valueOf()`方法，注意如果是`null`该方法会返回`null`字符串，虽然不会抛出 NPE 异常，但可能会产生预期外的后果。

    -   如果确定该对象就是`String`对象，直接使用`(String) obj`即可，当 obj 为`null`时也不会抛出 NPE 异常，若该对象实例不是`String`时，则会抛出`ClassCastException`。

反例：

    int i = 0;
    String str = "" + i;

    Integer integer = 1;
    str = "" + integer;

正例：

    int i = 0;
    String str = Integer.toString(i);
    str = String.valueOf(i); // 内部调用的也是 Integer.toString()

    Integer integer = 1;
    str = integer.toString();
    str = String.valueOf(integer);

    Object obj = List.of(1);
    str = obj.toString();
    str = String.valueOf(obj);

    String s = null;
    str = (String) s;

**10.【推荐】原子数据类型与包装类型的转换原则：**

**10.1. 自动转换（AutoBoxing）有一定成本，调用者与被调用函数间尽量使用同一类型，减少默认转换。**

反例：

    // sum 类型为 Long，i 类型为 long，每次相加都需要 AutoBoxing。
    Long sum = 0L;

    for (long i = 0; i < 10000; i++) {
        sum + =i;
    }

正例：

    // 准确使用API返回正确的类型
    Integer i = Integer.valueOf(str);
    int i = Integer.parseInt(str);

-   [Sonar-2153: Boxing and unboxing should not be immediately reversed](https://rules.sonarsource.com/java/RSPEC-2153)

**10.2. 自动拆箱有可能产生NPE，要注意处理。**

反例：

    // 如果intObject为null，产生NPE
    int i = intObject;
