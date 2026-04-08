# （五）类设计

**1.【强制】所有的覆写方法，必须加`@Override`注解。**

`getObject()`与`get0bject()`的问题。一个是字母的`O`，一个是数字的`0`，加`@Override`可以准确判断是否覆盖成功。另外，如果在抽象类中对方法签名进行修改，其实现类会马上编译报错。最后，在 IDE 的 Save Action 中配置自动添加`@Override`注解，如果无意间错误同名覆写了父类方法也能被发现。

-   [Sonar-1161: "@Override" should be used on overriding and implementing methods](https://rules.sonarsource.com/java/RSPEC-1161)

**2.【强制】定义数据对象 DO 时，属性类型要与数据库字段类型相匹配。**

正例：数据库字段的`bigint`必须与类属性的`Long`类型相对应。

反例：某业务的数据库表`id`字段定义类型为`bigint unsigned`，实际类对象属性为`Integer`，随着`id`越来越大，超过`Integer`的表示范围而溢出成为负数，此时数据库`id`不支持存入负数抛出异常产生线上故障。

**3.【强制】定义 DO / Query / Command / View 等 POJO 类时，不要设定任何属性*默认值*。**

反例：某业务的 DO 的`createTime`默认值为`Instant.now()`； 但是这个属性在数据提取时并没有置入具体值，在更新其它字段时又附带更新了此字段，导致创建时间被修改成当前时间。

**4.【强制】禁止在 POJO 类中，同时存在对应属性`xxx`的`isXxx()`和`getXxx()`方法。**

框架在调用属性 xxx 的提取方法时，并不能确定哪个方法一定是被优先调用到，神坑之一。

**5.【强制】`Object`的`equals`方法容易抛空指针异常，应使用常量或确定有值的对象来调用`equals`。**

正例：`"test".equals(param);`

反例：`param.equals("test");`

推荐使用 JDK7 引入的工具类`java.util.Objects#equals(Object a, Object b)`。

-   [Sonar-1132: Strings literals should be placed on the left side when checking for equality](https://rules.sonarsource.com/java/RSPEC-1132)

**6.【强制】`hashCode`和`equals`方法的处理，遵循如下规则：**

1.  只要重写`equals`，就必须重写`hashCode`。而且选取相同的属性进行运算。

2.  只选取真正能决定对象是否一致的属性，而不是所有属性，可以改善性能。

3.  对不可变对象，可以缓存`hashCode`值改善性能（比如`String`就是例子）。

4.  类的属性增加时，及时重新生成`toString`，`hashCode`和`equals`方法。

    -   [Sonar-1206: "equals(Object obj)" and "hashCode()" should be overridden in pairs](https://rules.sonarsource.com/java/RSPEC-1206)

**7.【强制】POJO类必须覆写`toString`方法。如果继承了另一个 POJO 类，注意在前面加一下 `super.toString()`。**

在方法执行抛出异常时，可以直接调用 POJO 的`toString()`方法打印其属性值，便于排查问题。

**8.【强制】使用 IDE 生成`toString`，`hashCode`和`equals`方法。**

使用 IDE 生成而不是手写，能保证`toString`有统一的格式，`equals`和`hashCode`则避免不正确的`null`值处理。子类生成`toString()`时，还需要勾选父类的属性。

**9.【强制】除了保持兼容性的情况，总是移除无用属性、方法与参数。**

特别是`private`的属性、方法、内部类，`private`方法上的参数，一旦无用立刻移除。信任代码版本管理系统。

-   [Sonar-3985: Unused "private" classes should be removed](https://rules.sonarsource.com/java/RSPEC-3985)

-   [Sonar-1068: Unused "private" fields should be removed](https://rules.sonarsource.com/java/RSPEC-1068)

-   [Sonar-1144: Unused "private" methods should be removed](https://rules.sonarsource.com/java/RSPEC-1144)

-   [Sonar-1481: Unused local variables should be removed](https://rules.sonarsource.com/java/RSPEC-1481)

-   [Sonar-1172: Unused method parameters should be removed](https://rules.sonarsource.com/java/RSPEC-1172)

**10.【强制】工具类需要设置为`final`，并且将其构造器设置为`private`使其不能被实例化。**

正例：

    public final class StringUtils {
        private StringUtils() {
            throw new AssertionError();
        }
    }

构造器中抛出`AssertionError`防止反射实例化工具类。

**11. 静态方法访问的原则：**

**11.1.【强制】避免通过一个类的对象引用访问此类的静态变量或静态方法，无谓增加编译器解析成本，直接用*类名*来访问即可。**

-   [Sonar-2209: "static" members should be accessed statically](https://rules.sonarsource.com/java/RSPEC-2209)

-   [Sonar-2440: Classes with only "static" methods should not be instantiated](https://rules.sonarsource.com/java/RSPEC-2440)

**11.2.【推荐】除测试用例，不要使用static import 静态方法。**

静态导入后忽略掉的类名，给阅读者造成障碍。

例外：测试环境中的assert语句，大家都太熟悉了。

-   [Sonar-3030: Classes should not have too many "static" imports](https://rules.sonarsource.com/java/RSPEC-3030)

**11.3.【推荐】尽量避免在非静态方法中修改静态成员变量的值。**

反例：

    public void foo() {
        ClassA.staticFiled = 1;
    }

-   [Sonar-2696: Instance methods should not write to "static" fields](https://rules.sonarsource.com/java/RSPEC-2696)

-   [Sonar-3010: Static fields should not be updated in constructors](https://rules.sonarsource.com/java/RSPEC-3010)

**12.【推荐】类成员与方法访问控制从严：**

1.  如果不允许外部直接通过`new`来创建对象，那么构造方法必须是`private`。

2.  工具类不允许有`public`或`default`构造方法。

3.  类非`static`成员变量并且与子类共享，必须是`protected`。

4.  类非`static`成员变量并且仅在本类使用，必须是`private`。

5.  类`static`成员变量如果仅在本类使用，必须是`private`。

6.  若是`static`成员变量，考虑是否为`final`。

7.  类成员方法只供类内部调用，必须是`private`。

8.  类成员方法只对继承类公开，那么限制为`protected`。

任何类、方法、参数、变量，严控访问范围。过于宽泛的访问范围，不利于模块解耦。思考：如果是一个`private`的方法，想删除就删除，可是一个`public`的`service`方法，或者一个`public`的成员变量，删除一下，不得手心冒点汗吗？

例外：为了单元测试，有时也可能将访问范围扩大，此时需要加上 JavaDoc 说明。

**13.【推荐】减少类之间的依赖。**

比如如果 A 类只依赖 B 类的某个属性，在构造函数和方法参数中，只传入该属性。让阅读者知道，A 类只依赖了 B 类的这个属性，而不依赖其他属性，也不会调用 B 类的任何方法。

正例：

    a.foo(b.bar);

反例：

    a.foo(b);

**14.【推荐】定义变量与方法参数时，尽量使用接口而不是具体类。**

使用接口可以保持一定的灵活性，也能向读者更清晰的表达你的需求：变量和参数只是要求是一个`Map`，而不是特定要求是一个`HashMap`。

例外：如果变量和参数要求某种特殊类型的特性，则需要清晰定义该参数类型，同样是为了向读者表达你的需求。

**15.【推荐】类的长度度量。**

类尽量不要超过300行。

对过大的类进行分拆时，可考虑其内聚性，即类的属性与类的方法的关联程度，如果有些属性没有被大部分的方法使用，其内聚性是低的。

**16.【推荐】构造函数如果有很多参数，且有多种参数组合时，建议使用 Builder 模式。**

正例：

    Executor executor = new ThreadPoolBuilder().coreThread(10).queueLenth(100).build();

即使仍然使用构造函数，也建议使用 chain constructor 模式，逐层加入默认值传递调用，仅在参数最多的构造函数里实现构造逻辑。

正例：

    public A() {
        A(DEFAULT_TIMEOUT);
    }

    public A(int timeout) {
        ...
    }

**17.【推荐】构造函数要简单，尤其是存在继承关系的时候。**

可以将复杂逻辑，尤其是业务逻辑，抽取到独立函数，如`init()`，`start()`，让使用者显式调用。

正例：

    Foo foo = new Foo();
    foo.init();

**18.【推荐】内部类的定义原则。**

当一个类与另一个类关联非常紧密，处于从属的关系，特别是只有该类会访问它时，可定义成私有内部类以提高封装性。

另外，内部类也常用作回调函数类，在 JDK8 下建议写成 Lambda 。

内部类分匿名内部类，内部类，静态内部类三种。

1.  匿名内部类 与 内部类，按需使用：

    在性能上没有区别；当内部类会被多个地方调用，或匿名内部类的长度太长，已影响对调用它的方法的阅读时，定义有名字的内部类。

2.  静态内部类 与 内部类，优先使用静态内部类：

    1.  非静态内部类持有外部类的引用，能访问外类的实例方法与属性。构造时多传入一个引用对性能没有太大影响，更关键的是向阅读者传递自己的意图，内部类是否会访问外部类。

    2.  非静态内部类里不能定义static的属性与方法。

-   [Sonar-2694: Inner classes which do not reference their owning classes should be "static"](https://rules.sonarsource.com/java/RSPEC-2694)

-   [Sonar-1604: Anonymous inner classes containing only one method should become lambdas](https://rules.sonarsource.com/java/RSPEC-1604)

**19.【推荐】setter 方法中，参数名称与类成员变量名称一致，this.成员名=参数名。在 getter / setter 方法中，不要增加业务逻辑，增加排查问题的难度。**

反例：

    public Integer getData() {
        if (condition) {
            return this.data + 100;
        } else {
            return this.data - 100;
        }
    }

**20.【推荐】`final`可以声明类、成员变量、方法、以及本地变量，下列情况使用`final`关键字：**

1.  不允许被继承的类，如：`String`类。

2.  不允许修改引用的域对象，如： POJO 类的域变量。

3.  不允许被覆写的方法，如： POJO 类的 setter 方法。

4.  不允许运行过程中重新赋值的局部变量。

5.  避免上下文重复使用一个变量，使用`final`关键字可以强制重新定义一个变量，方便更好地进行重构

**21.【推荐】得墨忒耳法则，不要和陌生人说话。**

以下调用，一是导致了对A对象的内部结构(B,C)的紧耦合，二是连串的调用很容易产生NPE，因此链式调用尽量不要过长。

反例：

    obj.getA().getB().getC().hello();
