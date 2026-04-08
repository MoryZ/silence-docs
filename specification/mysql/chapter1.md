# （一）建表规约

**1.【强制】表达是与否概念的字段，必须使用`is_xxx`的方式命名，数据类型是`BIT`(1 表示是，0 表示否)。**

任何字段如果为非负数，必须是`UNSIGNED`。

**2.【强制】表名、字段名必须控制在32个字符以内，必须使用小写字母或数据，禁止出现数字开头，禁止两个下划线中间只出现数字。字段名称需要慎重考虑，上线后再修改代价很大。**

MySQL在`Windows`下不区分大小写，但在`Linux`下默认是区分大小写的。因此数据库名、表名、字段名都不允许出现任何大写字母，避免节外生枝。

正例：aliyun\_admin / rdc\_config / level3\_name

反例：AliyunAdmin / rdcConfig / level\_3\_name

**3.【强制】表名不使用复数名词。**

表名应该仅仅表示表里面的实体内容，不应该表示实体数量，对应于`DO`类名也是单数形式，符合表达习惯。

**4.【强制】禁用保留字，如`desc`、`range`、`match`、`delayed`等，请参考`MySQL`官方保留字。**

**5.【强制】主键索引名为`pk_字段名`；唯一索引名为`uk_字段名`；普通索引名为`idx_字段名`。**

`pk_`即`primary key`；`uk_`即`unique key`；`idx_`即`index`的简称。

索引中如有多个字段名则按照索引设置的字段顺序罗列。

正例：idx\_name\_age

**6.【强制】小数类型为`DECIMAL`，禁止使用`FLOAT`和`DOUBLE`类型。**

在存储的时候，`FLOAT`和`DOUBLE`都存在精度损失的问题，很可能在比较值的时候得到不正确的结果。如果存储的数据范围超过`DECIMAL`的范围，建议将数据拆成整数和小数分开存储。

**7.【强制】如果存储的字符串长度几乎相等，使用`CHAR`定长字符串类型。**

**8.【强制】`VARCHAR`是可变长字符串，不预先分配存储空间，长度不要超过 5000，如果存储长度大于此值，定义字段类型为`TEXT`，独立出来一张表，用主键来对应，避免影响其它字段索引效率。**

**9.【强制】表主键的设置及生成规则：**

1.  对于中间表，设置复合主键，类型分别对应引用表的主键类型。

2.  对于扩展表，如对于用户明细表`user_details`来说，该表数据和用户表`user`一一对应，可直接使用`user`表的主键`id`作为主键，不需要创建自增主键`id`和`user_id`来对应`user`表的`id`，在某些场景下使用更方便，无需多一步关联。

3.  对于主键可能暴露给外部用户的表，如`user`表，尽量不要设置自增主键，而采用如雪花算法生成主键，一方面避免攻击者对系统进行爆破攻击，另一方面也可以避免商业竞争对手猜出公司大概的用户规模，间接造成泄露某些商业机密。

4.  除以上几种情况，统一设置自增主键`id`，类型为`BIGINT UNSIGNED`。

**10.【强制】表必备字段：主键 / `created_by` / `created_date` / `updated_by` / `updated_date`。**

-   主键规则见上条。

-   `created_by`和`updated_by`类型统一为`VARCHAR(100)`。

-   `created_date`和`updated_date`类型统一为`DATETIME`。

**11.【强制】表示时间的字段类型为`TIMESTAMP(3)`，仅表示日期的字段类型为`DATE`，一般情况下禁止使用`DATETIME`。**

1.  `TIMESTAMP`是标准`SQL`字段类型，拥有更好的兼容性及性能。

2.  `TIMESTAMP`可以进行不同时区转换，而`DATETIME`仅表示数据库所设置时区的时间。

**12.【强制】对于程序中的枚举字段，使用`TINYINT`类型存储，如果为非负数则使用`TINYINT UNSIGNED`类型，禁止使用`VARCHAR`或`ENUM`类型。**

使用`TINYINT`可以节约存储空间，同时还能提高查询效率。

**13.【推荐】库名与应用名称尽量一致。**

**14.【推荐】如果修改字段含义或对字段表示的状态追加时，需要及时更新字段注释。**

**15.【推荐】字段允许适当冗余，以提高查询性能，但必须考虑数据一致。冗余字段应遵循：**

1.  不是频繁修改的字段。

2.  不是唯一索引的字段。

3.  不是`VARCHAR`超长字段，更不能是`TEXT`字段。

正例：各业务线冗余存储商品名称，避免查询时需要调用远程服务获取。

**16.【推荐】单表行数超过 500 万行或者单表容量超过 2GB，才推荐进行分库分表。**

如果预计三年后的数据量根本达不到这个级别，请不要在创建表时就分库分表。

**17.【推荐】合适的字符存储长度，不但节约数据库表空间、节约索引存储，更重要的是提升检索速度。**

正例：无符号值可以避免误存负数，且扩大了表示范围。

<table>
<colgroup>
<col style="width: 14%" />
<col style="width: 14%" />
<col style="width: 14%" />
<col style="width: 14%" />
<col style="width: 14%" />
<col style="width: 28%" />
</colgroup>
<thead>
<tr class="header">
<th style="text-align: left;">对象</th>
<th style="text-align: left;">年龄区间</th>
<th style="text-align: left;">类型</th>
<th style="text-align: left;">Java类型</th>
<th style="text-align: left;">字节</th>
<th style="text-align: left;">表示范围</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;"><p>人</p></td>
<td style="text-align: left;"><p>150 岁之内</p></td>
<td style="text-align: left;"><p>TINYINT UNSIGNED</p></td>
<td style="text-align: left;"><p>Short</p></td>
<td style="text-align: left;"><p>1</p></td>
<td style="text-align: left;"><p>无符号值：0 到 255</p></td>
</tr>
<tr class="even">
<td style="text-align: left;"><p>龟</p></td>
<td style="text-align: left;"><p>数百岁</p></td>
<td style="text-align: left;"><p>SMALLINT UNSIGNED</p></td>
<td style="text-align: left;"><p>Short</p></td>
<td style="text-align: left;"><p>2</p></td>
<td style="text-align: left;"><p>无符号值：0 到 65535</p></td>
</tr>
<tr class="odd">
<td style="text-align: left;"><p>恐龙化石</p></td>
<td style="text-align: left;"><p>数千万年</p></td>
<td style="text-align: left;"><p>INT UNSIGNED</p></td>
<td style="text-align: left;"><p>Long</p></td>
<td style="text-align: left;"><p>4</p></td>
<td style="text-align: left;"><p>无符号值：0 到约 43 亿</p></td>
</tr>
<tr class="even">
<td style="text-align: left;"><p>太阳</p></td>
<td style="text-align: left;"><p>约 50 亿年</p></td>
<td style="text-align: left;"><p>BIGINT UNSIGNED</p></td>
<td style="text-align: left;"><p>Long</p></td>
<td style="text-align: left;"><p>8</p></td>
<td style="text-align: left;"><p>无符号值：0 到 约 10 的 19 次方</p></td>
</tr>
</tbody>
</table>
