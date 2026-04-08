# （八）集合处理

**1.【强制】关于`hashCode`和`equals`的处理，遵循如下规则:**

1.  只要覆写`equals`，就必须覆写`hashCode`。

2.  因为`Set`存储的是不重复的对象，依据`hashCode`和`equals`进行判断，所以`Set`存储的对象必须覆写这两种方法。

3.  如果自定义对象作为`Map`的键，那么必须覆写`hashCode`和`equals`。

`String`因为覆写了`hashCode`和`equals`方法，所以可以愉快地将`String`对象作为`key`来使用。

-   [Sonar-2141: Classes that don’t define "hashCode()" should not be used in hashes](https://rules.sonarsource.com/java/RSPEC-2141)

**2.【强制】判断所有集合内部的元素是否为空，使用框架中的`CollectionUtils.isEmpty()`方法，而不是`size() == 0`的方式。**

在某些集合中，`isEmpty()`的时间复杂度为 O(1)，而且可读性更好。

正例：

    Map<String, Object> map = new HashMap<>(16);
    if (CollectionUtils.isEmpty(map)) {
        // do something
    }

**3.【强制】在需要将某个集合（如`List`）转换为其他集合（如`Set`）或`Map`时，统一使用框架中`CollectionUtils.transformToXXX`方法，禁止使用`java.util.stream.Stream.collector`方法。**

**4.【强制】`ArrayList`的`subList`结果不可强转成`ArrayList`，否则会抛出`ClassCastException`异常：`java.util.RandomAccessSubList cannot be cast to java.util.ArrayList`。**

`subList()`返回的是`ArrayList`的内部类`SubList`，并不是`ArrayList`本身，而是`ArrayList`的一个视图，对于`SubList`的所有操作最终会反映到原列表上。

**5.【强制】使用`Map`的方法`keySet()` / `values()` / `entrySet()`返回集合对象时，不可以对其进行添加元素操作，否则会抛出`UnsupportedOperationException`异常。**

**6.【强制】`Collections`类返回的对象，如`emptyList()` / `singletonList()`，以及JDK 11中的`List.of()` / `Set.of()` / `Map.of()`等返回的都是 immutable collection，不可对其进行添加或者删除元素的操作。**

如果查询无结果，返回`Collections.emptyList()`空集合对象，调用方一旦在返回的集合中进行了添加元素的操作，就会触发`UnsupportedOperationException`异常。

**7.【强制】在`subList`场景中，*高度注意*对父集合元素的增加或删除，均会导致子列表的遍历、增加、删除产生`ConcurrentModificationException`异常。**

抽查表明，90% 的程序员对此知识点都有错误的认知。

**8.【强制】使用集合转数组的方法，必须使用集合的`toArray(IntFunction<T[]> generator)`方法。**

该方法为JDK 11后引入，JDK 11前使用`toArray(T[] array)`方法。

反例：直接使用`toArray()`无参方法存在问题，此方法返回值只能是`Object[]`类，若强转其它类型数组将出现`ClassCastException`错误。

正例：

    List<String> list = new ArrayList<>(2);
    list.add("guan");
    list.add("bao");

    String[] array = list.toArray(String[]::new); // JDK 11
    array = list.toArray(new String[0]); // JDK 8

-   Facebook-Contrib: Correctness - Impossible downcast of toArray() result

**9.【强制】使用`Collection`接口任何实现类的`addAll()`方法时，要对输入的集合参数进行 NPE 判断。**

`ArrayList#addAll`方法的第一行代码即`Object[] a = c.toArray();`，其中`c`为输入集合参数，如果为`null`则直接抛出异常。

**10.【强制】使用工具类`Arrays.asList()`或`List.of()`把数组转换成集合时，不能使用其修改集合相关的方法，它的`add` / `remove` / `clear`方法会抛出`UnsupportedOperationException`异常。**

`asList`的返回对象是一个`Arrays`内部类，并没有实现集合的修改方法。`Arrays.asList`体现的是适配器模式，只是转换接口，后台的数据仍是数组。

    String[] str = new String[]{ "yang", "guan", "bao" };
    List<String> list = Arrays.asList(str);

第一种情况：`list.add("yangguanbao");`运行时异常。 第二种情况：`str[0] = "change";` list 中的元素也会随之修改，反之亦然。

JDK 11中的`List.of()`和`Arrays.afList()`有所不同，会对传入的数组进行复制，因此上例中对于数组中的修改并不会反应到 list 中。

`Arrays.asList(array)`，如果 array 是原始类型数组如`int[]`，会把整个 array 当作`List`的一个元素，`String[]`或`Foo[]`则无此问题。`Collections.addAll()`实际是循环加入元素，性能相对较低，同样会把`int[]`认作一个元素。

-   Facebook-Contrib: Correctness - Method calls Array.asList on an array of primitive values

**11.【强制】泛型通配符`<? extends T>`来接收返回的数据，此写法的泛型集合不能使用`add`方法，而`<? super T>`不能使用`get`方法，两者在接口调用赋值的场景中容易出错。**

扩展说一下 PECS(Producer Extends Consumer Super) 原则，即频繁往外读取内容的，适合用`<? extends T>`，经常往里插入的，适合用`<? super T>`。

**12.【强制】在无泛型限制定义的集合赋值给泛型限制的集合时，在使用集合元素时，需要进行`instanceof`判断，避免抛出`ClassCastException`异常。**

泛型是在 JDK5 后才出现，考虑到向前兼容，编译器是允许非泛型集合与泛型集合互相赋值。

反例：

    List<String> generics = null;
    List notGenerics = new ArrayList(10);
    notGenerics.add(new Object());
    notGenerics.add(new Integer(1));
    generics = notGenerics;
    // 此处抛出 ClassCastException 异常
    String string = generics.get(0);

**13.【强制】不要在`foreach`循环里进行元素的 remove / add 操作，remove 元素可使用 iterator 方式，如果并发操作，需要对 iterator 对象加锁。**

正例：

    List<String> list = new ArrayList<>();
    list.add("1");
    list.add("2");
    Iterator<String> iterator = list.iterator();
    while (iterator.hasNext()) {
        String item = iterator.next();
        if (删除元素的条件) {
            iterator.remove();
        }
    }

反例：

    for (String item : list) {
        if ("1".equals(item)) {
          list.remove(item);
        }
    }

-   Facebook-Contrib: Correctness - Method modifies collection element while iterating

-   Facebook-Contrib: Correctness - Method deletes collection element while iterating

**14.【强制】使用`entrySet`遍历`Map`类集合 Key / Value，而不是`keySet`方式进行遍历。**

`keySet`其实是遍历了 2 次，一次是转为`Iterator`对象，另一次是从`Map`中取出 key 所对应的 value。而`entrySet`只是遍历了一次就把 key 和 value 都放到了 entry 中，效率更高。如果是 JDK8，可使用`Map.forEach`方法。

正例：

    Map<String, Integer> map = new HashMap<>();
    for (Entry<String, Integer> entry : map.entrySet()) {
        String key = entry.getKey();
        Integer value = entry.getValue();
    }

反例：

    Map<String, Integer> map = new HashMap<>();
    for (String key : map.keySet()) {
        Integer value = entry.getValue();
    }

-   [Sonar-2864: "entrySet()" should be iterated when both the key and value are needed](https://rules.sonarsource.com/java/RSPEC-2864)

**15.【强制】在 JDK7 版本及以上，`Comparator`实现类要满足如下三个条件，不然`Arrays.sort`，`Collections.sort`会抛`IllegalArgumentException`异常。**

1.  x，y 的比较结果和 y，x 的比较结果相反。

2.  x &gt; y，y &gt; z，则 x &gt; z。

3.  x = y，则 x，z 比较结果和 y，z 比较结果相同。

反例：

    new Comparator<Student>() {
        @Override
        public int compare(Student o1, Student o2) {
            return o1.getId() > o2.getId() ? 1 : -1;
        }
    };

**16.【强制】高度注意`Map`类集合 Key / Value 能不能存储`null值`的情况，如下表格：**

<table>
<colgroup>
<col style="width: 20%" />
<col style="width: 20%" />
<col style="width: 20%" />
<col style="width: 20%" />
<col style="width: 20%" />
</colgroup>
<thead>
<tr class="header">
<th style="text-align: left;">集合类</th>
<th style="text-align: left;">Key</th>
<th style="text-align: left;">Value</th>
<th style="text-align: left;">Super</th>
<th style="text-align: left;">说明</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;"><p>Hashtable</p></td>
<td style="text-align: left;"><p>NotNull</p></td>
<td style="text-align: left;"><p>NotNull</p></td>
<td style="text-align: left;"><p>Dictionary</p></td>
<td style="text-align: left;"><p>线程安全</p></td>
</tr>
<tr class="even">
<td style="text-align: left;"><p>HashMap</p></td>
<td style="text-align: left;"><p>Nullable</p></td>
<td style="text-align: left;"><p>Nullable</p></td>
<td style="text-align: left;"><p>AbstractMap</p></td>
<td style="text-align: left;"><p>线程不安全</p></td>
</tr>
<tr class="odd">
<td style="text-align: left;"><p>TreeMap</p></td>
<td style="text-align: left;"><p>NotNull</p></td>
<td style="text-align: left;"><p>Nullable</p></td>
<td style="text-align: left;"><p>AbstractMap</p></td>
<td style="text-align: left;"><p>线程不安全</p></td>
</tr>
<tr class="even">
<td style="text-align: left;"><p>ConcurrentHashMap</p></td>
<td style="text-align: left;"><p>NotNull</p></td>
<td style="text-align: left;"><p>NotNull</p></td>
<td style="text-align: left;"><p>AbstractMap</p></td>
<td style="text-align: left;"><p>锁分段技术(JDK8:CAS)</p></td>
</tr>
</tbody>
</table>

反例：由于`HashMap`的干扰，很多人认为`ConcurrentHashMap`是可以置入`null`值，而事实上，存储`null`值时会抛出 NPE 异常。

**17.【强制】长生命周期的集合，里面内容需要及时清理，避免内存泄漏。包括下面情况，都要小心处理：**

1.  静态属性定义。

2.  长生命周期对象的属性。

3.  保存在`ThreadLocal`中的集合。

如无法保证集合的大小是有限的，使用合适的缓存方案代替直接使用`HashMap`。

另外，如果使用`WeakHashMap`保存对象，当对象本身失效时，就不会因为它在集合中存在引用而阻止回收。但 JDK 的`WeakHashMap`并不支持并发版本，如果需要并发可使用`Caffeine`。

**18.【强制】集合如果存在并发修改的场景，需要使用线程安全的版本。**

1.  著名的反例，`HashMap`扩容时，遇到并发修改可能造成 100% CPU 占用。

    推荐使用`java.util.concurrent`(JUC)工具包中的并发版集合，如`ConcurrentHashMap`等，优于使用`Collections.synchronizedXXX()`系列函数进行同步化封装（等价于在每个方法都加上`synchronized`关键字）。

    例外：`ArrayList`所对应的`CopyOnWriteArrayList`，每次更新时都会复制整个数组，只适合于读多写很少的场景。如果频繁写入，可能退化为使用`Collections.synchronizedList(list)`。

2.  即使线程安全类仍然要注意函数的正确使用

    例如：即使用了`ConcurrentHashMap`，但直接是用 `get` / `put`方法，仍然可能会多线程间互相覆盖。

    反例：

        E e = map.get(key);
        if (e == null) {
            e = new E();
            map.put(key, e); // 仍然能两条线程并发执行put，互相覆盖
        }
        return e;

    正例：

        return map.computeIfAbsent(key, _key -> new E());

**19.【强制】泛型集合使用时，在 JDK7 及以上，使用 diamond 语法或全省略。**

菱形泛型，即 diamond，直接使用`<>`来指代前边已经指定的类型。

正例：

    // diamond 方式，即<>
    Map<String, String> userCache = new HashMap<>(16); // 全省略方式
    List<User> users = new ArrayList(10);

**20.【推荐】集合初始化时，指定集合初始值大小。**

`HashMap`使用构造方法`HashMap(int initialCapacity)`进行初始化时，如果暂时无法确定集合大小，那么指定默认值(16)即可。

正例： initialCapacity = (需要存储的元素个数 / 负载因子) + 1。注意负载因子(即 loaderfactor)默认为 0.75，如果暂时无法确定初始值大小，请设置为 16(即默认值)。

反例： `HashMap`需要放置 1024 个元素，由于没有设置容量初始大小，随着元素增加而被迫不断扩容，`resize()`方法总共会调用 8 次，反复重建哈希表和数据迁移。当放置的集合元素个数达千万级时会影响程序性能。

**21.【推荐】尽量使用新式的`foreach`语法遍历`Collection`与数组。**

`foreach`是语法糖，遍历集合的实际字节码等价于基于`Iterator`的循环。

`foreach`代码一来代码简洁，二来有效避免了有多个循环或嵌套循环时，因为不小心的复制粘贴，用错了 iterator 或循环计数器(i, j)的情况。

**22.【推荐】`List`，`List<?>`与`List<Object>`的选择。**

定义成`List`，会被 IDE 提示需要定义泛型。如果实在无法确定泛型，就仓促定义成`List<?>`来蒙混过关的话，该 list 只能读，不能增改。定义成`List<Object>`，如规则 11 所述，`List<String>`并不是`List<Object>`的子类，除非函数定义使用了通配符。

因此实在无法明确其泛型时，使用`List`也是可以的。

**23.【推荐】如果 Key 只有有限的可选值，先将 Key 封装成`Enum`，并使用`EnumMap`。**

`EnumMap`内部存储结构为`Object[enum.size]`，访问时以`value = Object[enum.ordinal()]`获取值，同时具备`HashMap`的清晰结构与数组的性能。

正例：

    public enum COLOR {
        RED, GREEN, BLUE, ORANGE;
    }

    EnumMap<COLOR, String> moodMap = new EnumMap<>(COLOR.class);

-   [Sonar-1640: Maps with keys that are enum values should be replaced with EnumMap](https://rules.sonarsource.com/java/RSPEC-1640)

**24.【参考】合理利用好集合的有序性(sort)和稳定性(order)，避免集合的无序性(unsort)和不稳定 性(unorder)带来的负面影响。**

有序性是指遍历的结果是按某种比较规则依次排列的，稳定性指集合每次遍历的元素次序是一定的。如`ArrayList`是 order / unsort；`HashMap`是 unorder / unsort；`TreeSet`是 order / sort。

**25.【参考】利用`Set`元素唯一的特性，可以快速对一个集合进行去重操作，避免使用`List`的`contains()`进行遍历去重或者判断包含操作。**
