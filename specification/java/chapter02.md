# （二）常量定义

**1.【强制】不允许任何魔法值（即未经预先定义的常量）直接出现在代码中。**

反例：

    // 开发者 A 定义了缓存的 key。
    String key = "Id#taobao_" + tradeId;
    cache.put(key, value);
    // 开发者 B 使用缓存时直接复制少了下划线，即 key 是"Id#taobao" + tradeId，导致出现故障。
    String key = "Id#taobao" + tradeId;
    cache.get(key);

例外：-1,0,1,2,3 不认为是魔法数

-   [Sonar-109: Magic numbers should not be used](https://rules.sonarsource.com/java/RSPEC-109)

**2.【强制】`long`或`Long`赋值时，数值后使用大写 L，不能是小写 l，小写容易跟数字混淆，造成误解。**

`public static final Long NUM = 2l;`写的是数字的 21，还是`Long`型的 2？

**3.【强制】浮点数类型的数值后缀统一为大写的 D 或 F。**

正例：

public static final double HEIGHT = 175.5D;

public static final float WEIGHT = 150.3F;

**4.【推荐】不要使用一个常量类维护所有常量，要按常量功能进行归类，分开维护。**

大而全的常量类，杂乱无章，使用查找功能才能定位到要修改的常量，不利于理解，也不利于维护。

正例：缓存相关常量放在类`CacheConstants`下；系统配置相关常量放在类`SystemConfigConstants`下。

**5.【推荐】常量的复用层次有五层：跨应用共享常量、应用内共享常量、子工程内共享常量、包内共享常量、类内共享常量。**

1.  跨应用共享常量：放置在二方库中，如 client.jar 中的 constant 目录下。

2.  应用内共享常量：放置在一方库中，如子模块中的 constant 目录下。

反例：易懂常量也要统一定义成应用内共享常量，两个程序员在两个类中分别定义了表示“是”的常量：

类 A 中：`public static final String YES = "yes";`

类 B 中：`public static final String YES = "y";`

`A.YES.equals(B.YES)`，预期是`true`，但实际返回为`false`，导致线上问题。

**6.【推荐】如果变量值仅在一个固定范围内变化用`enum`类型来定义。**

如果存在名称之外的延伸属性应使用`enum`类型，下面正例中的数字就是延伸信息，表示一年中的第几个季节。

正例：

    public enum Season {
        SPRING(1), SUMMER(2), AUTUMN(3), WINTER(4);

        private int sequence;

        SeasonEnum(int sequence) {
            this.sequence = sequence;
        }

        public int getSequence() {
            return sequence;
        }
    }

业务代码中不要依赖`ordinary()`函数进行业务运算，而是自定义数字属性，以免枚举值的增减调序造成影响。
