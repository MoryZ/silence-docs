# （十）控制语句

**1.【强制】在一个`switch`块内，每个`case`要么通过`continue` / `break` / `return`等来终止，要么注释说明程序将继续执行到哪一个`case`为止；在一个`switch`块内，都必须包含一个`default`语句并且放在最后，即使它什么代码也没有。**

注意`break`是退出`switch`语句块，而`return`是退出方法体。

-   [Sonar-131: "switch" statements should end with "default" clauses](https://rules.sonarsource.com/java/RSPEC-131)

**2.【强制】当`switch`括号内的变量类型为`String`并且此变量为外部参数时，必须先进行`null`判断。**

反例：如下代码的输出是什么？

    public class SwitchString {
        public static void main(String[] args) {
            method(null);
        }

        public static void method(String param) {
            switch (param) {
                // 肯定不是进入这里
                case "sth":
                    System.out.println("it's sth");
                    break;
                // 也不是进入这里
                case "null":
                    System.out.println("it's null");
                    break;
                // 也不是进入这里
                default:
                    System.out.println("default");
            }
        }
    }

**3.【强制】在 `if` / `else` / `for` / `while` / `do`语句中必须使用大括号。**

反例：

    if (condition) statements;

即使只有一行代码，也要采用大括号的编码方式。可在 IDE 的 Save Action 中配置自动添加。

-   [Sonar-121: Control structures should use curly braces](https://rules.sonarsource.com/java/RSPEC-121)

**4.【强制】三目运算符 condition ? 表达式 1 : 表达式 2 中，高度注意表达式 1 和 2 在类型对齐时，可能抛出因自动拆箱导致的 NPE 异常。**

以下两种场景会触发类型对齐的拆箱操作：

1.  表达式 1 或 表达式 2 的值只要有一个是原始类型。

2.  表达式 1 或 表达式 2 的值的类型不一致，会强制拆箱升级成表示范围更大的那个类型。

反例：

    Integer a = 1;
    Integer b = 2;
    Integer c = null;
    Boolean flag = false;
    // a * b 的结果是 int 类型，那么 c 会强制拆箱成 int 类型，抛出 NPE 异常
    Integer result = (flag ? a * b : c);

**5.【强制】在高并发场景中，避免使用“等于”判断作为中断或退出的条件。**

如果并发控制没有处理好，容易产生等值判断被“击穿”的情况，使用大于或小于的区间判断条件来代替。

反例：判断剩余奖品数量等于 0 时，终止发放奖品，但因为并发处理错误导致奖品数量瞬间变成了负数，这样的话，活动无法终止。

**6.【推荐】表达异常的分支时，少用 if-else 方式，多用哨兵语句式以减少嵌套层次。**

正例：

    if (condition) {
        ...
        return obj;
    }
    // 接着写else的业务逻辑代码;

如果非使用 if()…​else if()…​else…​方式表达逻辑，避免后续代码维护困难，请勿超过 3 层。

正例：超过 3 层的 if-else 的逻辑判断代码可以使用哨兵语句、策略模式、状态模式等来实现，其中哨兵语句示例如下:

    public void findBoyfriend(Man man) {
        if (man.isUgly()) {
            System.out.println("本姑娘是外貌协会的资深会员");
            return;
        }
        if (man.isPoor()) {
            System.out.println("贫贱夫妻百事哀");
            return;
        }
        if (man.isBadTemper()) {
            System.out.println("银河有多远，你就给我滚多远");
            return;
        }
        System.out.println("可以先交往一段时间看看");
    }

-   Facebook-Contrib: Style - Method buries logic to the right (indented) more than it needs to be

**7.【推荐】限定方法的嵌套层次。**

所有`if` / `else` / `for` / `while` / `try`的嵌套，当层次过多时，将引起巨大的阅读障碍，因此一般推荐嵌套层次不超过 4 层。

通过抽取方法，或哨兵语句（见6）来减少嵌套。

-   [Sonar-134: Control flow statements "if", "for", "while", "switch" and "try" should not be nested too deeply](https://rules.sonarsource.com/java/RSPEC-134)，增大为4

**8.【推荐】除常用方法(如 getXxx / isXxx)等外不要在条件判断中执行其它复杂的语句，将复杂逻辑判 断的结果赋值给一个有意义的布尔变量名，以提高可读性。**

很多`if`语句内的逻辑表达式相当复杂，与、或、取反混合运算，甚至各种方法纵深调用，理解成本非常高。如果赋值一个非常好理解的布尔变量名字，则是件令人爽心悦目的事情。

正例：

    final boolean existed = (file.open(fileName, "w") != null) && (...) || (...);
    if (existed) {
        ...
    }

反例：

    public final void acquire(long arg) {
        if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg)) {
            selfInterrupt();
        }
    }

-   [Sonar-1067: Expressions should not be too complex](https://rules.sonarsource.com/java/RSPEC-1067)，增大为4

**9.【推荐】简单逻辑，善用三目运算符，减少 if-else 语句的编写。**

    s !=``NULL``? s : "";

**10.【推荐】不要在其它表达式（尤其是条件表达式）中，插入赋值语句。**

赋值点类似于人体的穴位，对于代码的理解至关重要，所以赋值语句需要清晰地单独成为一行。

反例：

    public Lock getLock(boolean fair) {
        // 算术表达式中出现赋值操作，容易忽略 count 值已经被改变
        threshold = (count = Integer.MAX_VALUE) - 1;
        // 条件表达式中出现赋值操作，容易误认为是 sync == fair
        return (sync = fair) ? new FairSync() : new NonfairSync();
    }

**11.【推荐】循环体中的语句要考量性能，以下操作尽量移至循环体外处理，如定义对象、变量、获取数据 库连接，进行不必要的 try-catch 操作（这个 try-catch 是否可以移至循环体外）。**

**12.【推荐】避免采用取反逻辑运算符。**

取反逻辑不利于快速理解，并且取反逻辑写法一般都存在对应的正向逻辑写法。

正例：使用 if(x &lt; 628) 来表达 x 小于 628。

反例：使用 if(!(x &gt;= 628)) 来表达 x 小于 628。

-   [Sonar-1940: Boolean checks should not be inverted](https://rules.sonarsource.com/java/RSPEC-1940)

**13.【推荐】表达式中，能造成短路概率较大的逻辑尽量放前面，使得后面的判断可以免于执行。**

    if (maybeTrue() || maybeFalse()) { ... }

    if (maybeFalse() && maybeTrue()) { ... }

**14.【推荐】能用`while`循环实现的代码，就不用`do-while`循环。**

`while`语句能在循环开始的时候就看到循环条件，便于帮助理解循环内的代码；

`do-while`语句要在循环最后才看到循环条件，不利于代码维护，代码逻辑容易出错。

**15.【推荐】公开接口需要进行入参保护，尤其是批量操作的接口。**

反例：某业务系统，提供一个用户批量查询的接口，API 文档上有说最多查多少个，但接口实现上没做任何保护，导致调用方传了一个 1000 的用户 id 数组过来后，查询信息后，内存爆了。
