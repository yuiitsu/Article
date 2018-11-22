# MySQL INT UNSINGED和SINGED类型

## 解释

先看看[MySQL](https://dev.mysql.com/doc/refman/5.7/en/numeric-type-attributes.html)的解释：

> All integer types can have an optional (nonstandard) attribute `UNSIGNED`. Unsigned type can be used to permit only nonnegative numbers in a column or when you need a larger upper numeric range for the column. For example, if an [`INT`](https://dev.mysql.com/doc/refman/5.7/en/integer-types.html) column is `UNSIGNED`, the size of the column's range is the same but its endpoints shift from `-2147483648`and `2147483647` up to `0` and `4294967295`.

字面意思就是无符号和有符号之分，无符号也就是正整数，有符号包括负数。INT类型长度分别为`0 - 4294967295`和`-2147483648 - 2147483647`，不同INT类型长度自然不同。MySQL默认使用的是SINGED类型。

## 创建UNSINGED INT字段

```mysql
CREATE `tbl_name` (
	`number` INT UNSINGED NOT NULL DEFAULT 0
)
```

## 使用场景

如果你确认字段中不会包含负数，那么就使用UNSIGNED。

注意：主键AUTO-INCREMENT INT时，从0开始自增，默认使用SIGNED，负值永远不会用到，但正整数长度确少了一半。所以，指定为UNSINGED是比较好的方法。

> In MySQL 5.7, negative values for `AUTO_INCREMENT` columns are not supported.

## 性能问题

严格来讲，UNSINGED比SINGED稍好。如果一个字段只存正整数，但使用的是SINGED，那么，在做这样查询的时候：

```mysql
SELECT * FROM `product` WHERE `quantity` <= 500;
```

MySQL定义的查询范围是：`-2147483648 - 500`。如果是UNSINGED，范围是：`0 - 500`。所以，如果数据量不大，那差别几乎没有，但如果数据量足够大的时候，即使为quantity添加索引，两种类型的速度也是有区别的。

## 版本问题

如上UNSINGED范围从0开始，如果将其减1为负数，正常情况下都希望MySQL执行失败，给出错误信息。比如，简单操作库存，不希望超卖。那么这里有个问题，在MySQL 5.5.5之前的版本，执行此操作，MySQL不仅不会报错执行失败，还会自动将字段值置为类型的最大值，如：INT类型为置为`4294967295`。而高于等于5.5.5版本的MySQL，会直接报错，执行失败：

```mysql 
BIGINT UNSIGNED value is out of range in ...
```

PS: Centos大多数的YUM源安装的MySQL版本都为5.1，而MariaDB版本都在5.5.5以上，所以安装MariaDB是一个优选。

### 参考：

1.https://dev.mysql.com/doc/refman/5.7/en/numeric-type-attributes.html

