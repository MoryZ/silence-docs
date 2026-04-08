# （三）代码格式

**1.【强制】使用内部统一的代码格式模板，基于 IDE 自动的格式化。**

1.  IDE 的默认代码格式模板，能简化绝大部分关于格式规范（如空格，括号）的描述。

2.  统一的模板，在接手旧项目时先进行一次全面格式化，这样可以避免不同开发者之间因为格式不统一产生代码合并冲突，或者代码变更日志中因为格式不同引起的变更，掩盖了真正的逻辑变更。

3.  设定统一的行宽，建议 120。

4.  设定 4 个空格缩进，禁止使用 Tab 字符，基于 IDE 自动转换。

**2.【强制】IDE 的 text file encoding 设置为UTF-8; IDE 中文件的换行符使用 Unix 格式，不要使用 Windows 格式。**

**3.【强制】注释的双斜线与注释内容之间有且仅有一个空格。**

正例：

    // 这是示例注释，请注意在双斜线之后有一个空格
    String commentString = new String("demo");

**4.【强制】没有必要增加若干空格来使变量的赋值等号与上一行对应位置的等号对齐。**

正例：

    int one = 1;
    long two = 2L;
    float three = 3F;
    StringBuilder builder = new StringBuilder();

增加`builder`这个变量，如果需要对齐，则需要给`one`、`two`、`three`都增加几个空格，在变量比较多的情况下，是非常累赘的事情。

**5.【强制】Web Controller / Service / Repository / DAO 类内的方法，按照 HTTP Method "GET / POST / PUT / PATCH / DELETE" 的顺序定义。**

在方法较多时可帮助代码阅读者快速定位到想要查找的方法，并且可以统一代码风格。

正例：

    public User findById() {...}
    public List<User> findAll() {...}
    public ResponseEntity<Void> create() {...}
    public void update() {...}
    public void delete() {...}

**6.【推荐】用小括号来限定运算优先级。**

我们没有理由假设读者能记住整个 Java 运算符优先级表。除非作者和 Reviewer 都认为去掉小括号也不会使代码被误解，甚至更易于阅读。

正例：

    if ((a == b) && (c == d))

-   [Sonar-1068: Limited dependence should be placed on operator precedence rules in expressions](https://rules.sonarsource.com/java/RSPEC-1068)，修改了三目运算符`foo !=`NULL`? foo : ""`不需要加括号。

**7.【推荐】类内方法定义的顺序，不要“总是在类的最后添加新方法”。**

一个类就是一篇文章，想象一个阅读者的存在，合理安排方法的布局。

1）顺序依次是：构造函数 &gt; (公有方法&gt;私有方法&gt;保护方法) &gt; getter/setter方法。

如果公有方法可以分成几组，私有方法也紧跟公有方法的分组。私有方法应该在第一次被用到的公有方法后声明。

2）当一个类有多个构造方法，或者多个同名的重载方法，这些方法应该放置在一起。其中参数较多的方法在后面。

正例：

    public Foo(int a) {...}
    public Foo(int a, String b) {...}

    public void foo(int a) {...}
    public void foo(int a, String b) {...}

3）作为调用者的方法，尽量放在被调用的方法前面。

正例：

    public void foo() {
        bar();
    }

    public void bar() {...}

**8.【推荐】通过空行进行逻辑分段。**

一段代码也是一段文章，需要合理的分段而不是一口气读到尾。

不同组的变量之间，不同业务逻辑的代码行之间，插入一个空行，起逻辑分段的作用。

而联系紧密的变量之间、语句之间，则尽量不要插入空行。

正例：

    int width;
    int height;

    String name;

**9.【推荐】避免IDE格式化。**

对于一些特殊场景（如使用大量的字符串拼接成一段文字，或者想把大量的枚举值排成一列），为了避免 IDE 自动格式化，土办法是把注释符号 // 加在每一行的末尾，但这有视觉的干扰，可以使用`@formatter:off`和`@formatter:on`来包装这段代码，让 IDE 跳过它。

正例：

    // @formatter:off
    ...
    // @formatter:on
