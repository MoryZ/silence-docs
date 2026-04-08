# （四）方法设计

**1.【强制】相同参数类型，相同业务含义，才可以使用的可变参数，参数类型避免定义为`Object`。**

可变参数必须放置在参数列表的最后。（建议开发者尽量不用可变参数编程）。

正例：`public List<User> listUsers(String type, Long…​ ids) {…​}`

**2.【强制】外部正在调用的接口或者二方库依赖的接口，不允许修改*方法签名*，避免对接口调用方产生影响。接口过时必须加`@Deprecated`注解，并清晰地说明采用的新接口或者新服务是什么。**

**3.【强制】不能使用过时的类或方法。**

`java.net.URLDecoder`中的方法`decode(String encodeStr)`这个方法已经过时，应该使用双参数`decode(String source, String encode)`。接口提供方既然明确是过时接口，那么有义务同时提供新的接口；作为调用方来说，有义务去考证过时方法的新实现是什么。

**4.【推荐】方法的长度度量。**

方法尽量不要超过 100 行。

另外，方法长度超过 8000 个字节码时，将不会被 JIT 编译成二进制码。

-   [Sonar-107: Methods should not have too many lines](https://rules.sonarsource.com/java/RSPEC-107)，默认值改为100

-   Facebook-Contrib: Performance - This method is too long to be compiled by the JIT

**5.【推荐】方法的语句在同一个抽象层级上。**

反例：

一个方法里，前 20 行代码在进行很复杂的基本价格计算，然后调用一个折扣计算函数，再调用一个赠品计算函数。

此时可将前 20 行也封装成一个价格计算函数，使整个方法在同一抽象层级上。

**6.【推荐】为了帮助阅读及方法内联，将小概率发生的异常处理及其他极小概率进入的代码路径，封装成独立的方法。**

正例：

    if (seldomHappenCase) {
        hanldMethod();
    }

    try {
      ...
    } catch (SeldomHappenException e) {
        handleException();
    }

**7.【推荐】尽量减少重复的代码，抽取方法。**

超过 5 行以上重复的代码，都可以考虑抽取公用的方法。

**8.【推荐】方法参数最好不超过3个，最多不超过7个。**

1）如果多个参数同属于一个对象，直接传递对象。

例外：你不希望依赖整个对象，传播了类之间的依赖性。

2）将多个参数合并为一个新创建的逻辑对象。

例外：多个参数之间毫无逻辑关联。

3）将函数拆分成多个函数，让每个函数所需的参数减少。

-   [Sonar-107: Methods should not have too many parameters](https://rules.sonarsource.com/java/RSPEC-107)

**9.【推荐】下列情形，需要进行参数校验：**

1.  调用频次低的方法。

2.  执行时间开销很大的方法。此情形中，参数校验时间几乎可以忽略不计，但如果因为参数错误导致中间执行回退，或者错误，代价更大。

3.  需要极高稳定性和可用性的方法。

4.  对外提供的开放接口，不管是 RPC / API / HTTP接口。

5.  敏感权限入口。

如果使用 Apache Validate 进行校验，并附加错误提示信息时，注意不要每次校验都做一次字符串拼接。

反例：

    Validate.isTrue(length > 2, "length is " + keys.length + ", less than 2", length);

正例：

    Validate.isTrue(length > 2, "length is %d, less than 2", length);

**10.【推荐】下列情形，不需要进行参数校验：**

1.  极有可能被循环调用的方法。但在方法说明里必须注明外部参数检查。

2.  底层调用频度比较高的方法。毕竟是像纯净水过滤的最后一道，参数错误不太可能到底层才会暴露问题。比如，一般 DAO 层与 Service 层都在同一个应用中，所以 DAO 层的参数校验，可以省略。

3.  被声明成`private`，或其他只会被自己代码所调用的方法，如果能够确定在调用方已经做过检查，或者肯定不会有问题则可省略。即使忽略检查，也尽量在方法说明里注明参数的要求，比如 Spring 5 中的`@NotNull`，`@Nullable`标识。

**11.【推荐】禁用`assert`做参数校验。**

`assert`断言仅用于测试环境调试，无需在生产环境时进行的校验。因为它需要增加`-ea`启动参数才会被执行。而且校验失败会抛出一个`AssertionError`（属于`Error`，需要捕获`Throwable`）。因此在生产环境进行的校验，推荐使用 Apache Commons Lang 的 Validate。

**12.【推荐】返回值可能为`null`，尽可能使用 JDK8 的`Optional`类，或者使用`@Nullable`注解标识。**

不强制返回空集合，或者空对象。但需要添加注释充分说明什么情况下会返回`null`值。

本规约明确防止 NPE 是调用者的责任。即使被调用方法返回空集合或者空对象，对调用者来说，也并非高枕无忧，必须考虑到远程调用失败、序列化失败、运行时异常等场景返回`null`的情况。

JDK8 的`Optional`类的使用这里不展开。

**13.【推荐】不能使用有继承关系的参数类型来重载方法。**

因为方法重载的参数类型是根据编译时表面类型匹配的，不根据运行时的实际类型匹配。

反例：

    class A {
      void hello(List<String> list);
      void hello(ArrayList<String> arrayList);
    }

    List<String> arrayList = new ArrayList<>();

    // 下句调用的是hello(List<String> list)，因为 ArrayList 的定义类型是 List
    a.hello(arrayList);

**14.【推荐】不使用不稳定方法，如com.sun.\*包下的类，底层类库中internal包下的类**

`com.sun.*`、`sun.*`包下的类，或者底层类库中名称为`internal`的包下的类，都是不对外暴露的，可随时被改变的不稳定类。

-   [Sonar-1191: Classes from "sun.\*" packages should not be used](https://rules.sonarsource.com/java/RSPEC-1191)
