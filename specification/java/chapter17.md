# （十七）其他规约

**1.【强制】使用 MapStruct 进行属性的 copy，避免用 Apache Beanutils / Spring BeanUtils。**

MapStruct 在编译时动态生成代码，无反射调用开销，而 Apache BeanUtils / Spring BeanUtils 等均通过反射实现属性复制，性能较差。

**2.【强制】注意`Math.random()`这个方法返回是`double`类型，注意取值的范围 0 ≤ x &lt; 1 (能够取到*零*值，注意除零异常)，如果想获取整数类型的随机数，不要将 x 放大 10 的若干倍然后取整，直接使用`Random`对象的`nextInt`或者`nextLong`方法。**

**3.【强制】枚举 enum(括号内)的属性字段必须是私有且不可变。**

**4.【强制】变量声明尽量靠近使用的分支。**

不要在一个代码块的开头把局部变量一次性都声明了（这是c语言的做法），而是在第一次需要使用它时才声明。

否则如果方法已经退出或进入其他分支，就白白初始化了变量。

反例：

    Foo foo = new Foo();

    if (ok){
        return;
    }

    foo.bar();

**5.【强制】不要像C那样一行里做多件事情。**

反例：

    fooBar.fChar = barFoo.lchar = 'c';
    argv++; argc--;
    int level, size;

-   [Sonar-1659: Multiple variables should not be declared on the same line](https://rules.sonarsource.com/java/RSPEC-1659)

**6.【强制】不要为了性能而使用JNI本地方法。**

Java 在 JIT 后并不比 C 代码慢，JNI 方法因为要反复跨越 JNI 与 Java 的边界反而有额外的性能损耗。

因此 JNI 方法仅建议用于调用 JDK 所没有包括的，对特定操作系统的系统调用。

**7.【推荐】正确使用反射，减少性能损耗。**

获取`Method` / `Field`对象的性能消耗较大, 而如果对`Method`与`Field`对象进行缓存再反复调用，则并不会比直接调用类的方法与成员变量慢（前15次使用`NativeAccessor`，第15次后会生成`GeneratedAccessorXXX`，bytecode 为直接调用实际方法）。

    // 用于对同一个方法多次调用
    private Method method = ....

    public void foo(){
        method.invoke(obj, args);
    }

    // 用于仅会对同一个方法单次调用
    ReflectionUtils.invoke(obj, methodName, args);

**8.【推荐】及时清理不再使用的代码段或配置信息。**

对于垃圾代码或过时配置，坚决清理干净，避免程序过度臃肿，代码冗余。

正例：对于暂时被注释掉，后续可能恢复使用的代码片断，在注释代码上方，统一规定使用三个斜杠（///）来说明注释掉代码的理由：

    public static void hello() {
        /// 业务方通知活动暂停
        // Business business = new Business();
        // business.active();
        System.out.println("it's finished");
    }
