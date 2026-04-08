# （一）命名风格

**1.【强制】所有编程相关的命名均不能以下划线或美元符号开始，也不能以下划线或美元符号结束。**

反例：\_name / \_\_name / $Object / name\_ / name$ / Object$

例外：对于 lambda 表达式的参数，如果该参数名字已经被定义使用了，可使用下划线开头以避免编译错误（此规则参考 Spring 中的处理方式）。

    Map<String, Integer> msi = new HashMap<>();
    String name = computeSomeKey();
    msi.computeIfAbsent(name, _name -> _name.trim());

**2.【强制】所有编程相关的命名严禁使用拼音、拼音缩写或拼音与英文混合的方式，除非中国式业务词汇没有通用易懂的英文对应，更不允许直接使用中文的方式。**

正确的英文拼写和语法可以让阅读者易于理解，避免歧义。

正例：ali / alibaba / taobao / kaikeba / aliyun / youku / fapiao / hangzhou 等国际通用的名称，可视同英文。

反例： DaZhePromotion【打折】/ getPingFenByName()【评分】/ String fw【福娃】/ int 变量名 = 3

**3.【强制】类名使用UpperCamelCase风格，以下情形例外： DO / PO / DTO / BO / VO / UID 等。**

TCP、UDP、XML、SSL等缩写也遵循驼峰形式。

正例：ForceCode / UserDO / HtmlDTO / XmlService / TcpUdpDeal / TaPromotion

反例：forcecode / UserDo / HTMLDto / XMLService / TCPUDPDeal / TAPromotion

-   [Sonar-101: Class names should comply with a naming convention](https://rules.sonarsource.com/java/RSPEC-101)

-   [Sonar-114: Interface names should comply with a naming convention](https://rules.sonarsource.com/java/RSPEC-114)

**4.【强制】方法名、参数名、成员变量、局部变量都统一使用lowerCamelCase风格。**

正例： localValue / getHttpMessage() / inputUserId

-   [Sonar-100: Method names should comply with a naming convention](https://rules.sonarsource.com/java/RSPEC-100)

-   [Sonar-116: Field names should comply with a naming convention](https://rules.sonarsource.com/java/RSPEC-116)

-   [Sonar-117: Local variable and method parameter names should comply with a naming convention](https://rules.sonarsource.com/java/RSPEC-117)

**5.【强制】常量名应该全部大写，单词间用下划线隔开。力求语义表达完整清楚，不要嫌名字长。**

正例： MAX\_STOCK\_COUNT / CACHED\_EXPIRATION

反例： MAX\_COUNT / EXPIRATION

例外：当一个`static final`字段不是一个真正常量，比如不是基本类型时，不需要使用大写命名。

    private static final Logger logger = Logger.getLogger(MyClass.class);

常量以`private static final`开头，统一风格。

正例：

    private static final Logger logger = Logger.getLogger(MyClass.class);

反例：

    private final static Logger logger = Logger.getLogger(MyClass.class);

-   [Sonar-115: Constant names should comply with a naming convention](https://rules.sonarsource.com/java/RSPEC-115)

-   [Sonar-308: Static non-final field names should comply with a naming convention](https://rules.sonarsource.com/java/RSPEC-308)

**6.【强制】枚举成员名称需要全大写，单词间用下划线隔开。**

正例：枚举名字为 ProcessStatus 的成员名称 SUCCESS / UNKNOWN\_REASON。

枚举其实就是特殊的常量类，且构造方法被默认强制是私有。

**7.【强制】抽象类命名如无特殊情况使用 Abstract 或 Base 开头；异常类命名使用 Exception 结尾；测试类命名以它要测试的类名开始，以 Test 结尾。**

正例：DealStatusEnum / AbstractView / BaseView / TimeoutException / UserServiceTest

-   [Sonar-2166: Classes named like "Exception" should extend "Exception" or a subclass](https://rules.sonarsource.com/java/RSPEC-2166)

-   [Sonar-3577: Test classes should comply with a naming convention](https://rules.sonarsource.com/java/RSPEC-3577)

**8.【强制】类型与中括号紧挨相连来定义数组。**

正例：定义整形数组`int[] arrayDemo`。

反例：在 main 参数中，使用`String args[]`来定义。

**9.【强制】POJO 类中的任何布尔类型的变量，都不要加 is 前缀，否则部分框架解析会引起序列化错误。**

数据库规约中的建表约定第 1 条，表达是与否的变量采用`is_xxx`的命名方式，所以需要在`@Column`注解或`<resultMap>`设置从`is_xxx`到`xxx`的映射关系。

反例：定义为布尔类型`Boolean isDeleted`的字段，它的 getter 方法也是`isDeleted()`，部分框架在反向解析时，“误以为” 对应的字段名称是`deleted`，导致字段获取不到，得到意料之外的结果或抛出异常。

**10.【强制】包名全部小写。点分隔符之间尽量只有一个英语单词，无特殊情况不要超过两个单词，即使有多个单词也不使用下划线或大小写分隔。包名统一使用*单数*形式，但是类名如果有复数含义，可以使用复数形式。**

正例： com.amazon.member / com.amazon.member.util / com.amazon.benefits / com.amazon.messagesource，类名为MessageSourceUtils（此规则参考 Spring 的框架结构）

反例： com.amazon.message\_source / com.amazon.messageSource / com.amazon.utils

-   [Sonar-120: Package names should comply with a naming convention](https://rules.sonarsource.com/java/RSPEC-120)

**11.【强制】避免在子父类的成员变量之间、或者不同代码块的局部变量之间采用完全相同的命名,使可理解性降低。**

子类、父类成员变量名相同，即使是`public`也是能够通过编译，而局部变量在同一方法内的不同代码块中同名也是合法的，但是要避免使用。对于非 setter / getter 的参数名称也要避免与成员变量名称相同。

反例：

    public class ConfusingName {
        protected int stock;
        protected String alibaba;
        // 非 setter/getter 的参数名称，不允许与本类成员变量同名
        public void access(String alibaba) {
            if (condition) {
                final int money = 666;
                // ...
            }
            for (int i = 0; i < 10; i++) {
                // 在同一方法体中，不允许与其它代码块中的 money 命名相同
                final int money = 15978;
                // ...
            }
        }
    }
    class Son extends ConfusingName {
        // 不允许与父类的成员变量名称相同
        private int stock;
    }

-   [Sonar-2387: Child class fields should not shadow parent class fields](https://rules.sonarsource.com/java#RSPEC-2387)

-   [Sonar-1117: Local variables should not shadow class fields](https://rules.sonarsource.com/java/RSPEC-1117)

**12.【强制】杜绝完全不规范的英文缩写，避免望文不知义。**

反例： AbstractClass “缩写”成 AbsClass；condition “缩写”成 condi；message “缩写”成 msg，此类随意缩写严重降低了代码的可读性。

**13.【强制】接口类中的方法和属性不要加任何修饰符号（public 也不要加），保持代码的简洁性，并加上有效的 Javadoc 注释。尽量不要在接口里定义变量，如果一定要定义变量，确定与接口方法相关，并且是整个应用的基础常量。**

正例：接口方法签名`void commit()`;

正例：接口基础常量`String COMPANY = "amazon"`;

反例：接口方法定义`public abstract void f()`;

JDK8 中接口允许有默认实现，那么这个 default 方法，是对所有实现类都有价值的默认实现。

**14.【强制】Controller / Service / Repository / DAO 层方法命名规约。**

1.  根据某个或多个条件查询对象的方法用`find`做前缀，若需要根据条件查询后跟`ByXXX`，如`findByUsername`，多个条件用`And`连接，如`findByAgeAndGender`。

2.  获取统计值的方法用`count`做前缀，条件同`find`方法，如`countByAge`。

3.  插入的方法用`insert` / `create`做前缀。

4.  修改的方法用`update`做前缀，若需要根据条件更新后跟`ByXXX`，规则同查询。

5.  删除的方法用`delete` / `remove`做前缀，若需要根据条件后跟`ByXXX`，规则同查询。

**15.【强制】领域模型命名规约。**

1.  数据对象：xxx，xxx 即为数据表名。

2.  数据查询对象：xxxQuery，xxx 为业务领域相关的名称。

3.  数据变更对象：xxxCommand，xxx 为业务领域相关的名称。

4.  展示对象：xxxView，xxx 一般为业务领域及网页相关名称。

5.  POJO 是 Query / Command / View 的统称，禁止命名成 xxxPOJO。

**16.【推荐】为了达到代码自解释的目标，任何自定义编程元素在命名时，使用完整的单词组合来表达。**

正例：在 JDK 中，对某个对象引用的`volatile`字段进行原子更新的类名为 `AtomicReferenceFieldUpdater`。

反例： 常见的方法内变量为`int a;`的定义方式。

**17.【推荐】命名的好坏，在于其“模糊度”。**

1.  对于集合类的变量，应尽量用英语复数的形式命名，避免使用`xxxList`的形式，如`userList`，推荐用`users`，以防止某些情况下类型从`List`变为其他集合类型（如`Set`）的情况。

2.  如果上下文很清晰，局部变量可以使用`list`这种简略命名，否则应该使用`users`这种更清晰的命名。

3.  禁止 a1, a2, a3 这种带编号的命名方式。

4.  方法的参数名叫`books`，方法里的局部变量名叫`theBooks`也是很没诚意的。

5.  如果一个应用里同时存在`Account`、`AccountInfo`、`AccountData`类，或者一个类里同时有`getAccountInfo()`、`getAccountData()`、`save()`、`store()`的函数，阅读者将非常困惑。

6.  禁止`callerId`与`calleeId`，`mydearfriendswitha`与`mydearfriendswithb`这种拼写极度接近、考验阅读者眼力的写法。

**18.【推荐】如果模块、接口、类、方法使用了设计模式，在命名时要体现出具体模式，有利于阅读者快速理解设计思想。**

正例：OrderFactory / LoginProxy / ResourceObserver

**19.【推荐】实现类的类名应体现具体含义，如果是默认实现用`Default`开头，避免使用`Impl`等无意义的后缀。**

正例：`UserResource`实现`UserService`接口。

正例：`AbstractTranslator`实现`Translatable`接口。

反例：`UserServiceImpl`实现`UserService`接口。
