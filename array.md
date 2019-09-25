# 从数组说起

# 1. 自定义数组
在日常开发中，我们可能无时无刻都在使用数组，我们都知道数组是有局限性的，比如固定大小，无法扩容，但是你能自己手写一个数组解决上面的痛点呢？
看过 `Java` 源代码的都知道，`Java` 中的 `ArrayList` 底层实现就是数组，那我们就自己通过数组来实现自己的 `ArrayList` 具体代码见 [码云](https://gitee.com/sontekliu/datastruct-core/blob/master/src/main/java/com/javaliu/collection/array/Array.java) 和
[Github](https://github.com/sontekliu/datastruct-core/blob/master/src/main/java/com/javaliu/collection/array/Array.java) 

其实实现自己的 `ArrayList` 不是我们的目的，关键是我们要分析实现细节，以及其中的时间复杂度。

> 值得分析的方法 `add`:

```java
public void add(int index, E e) {
    if (index < 0 || index > size) {
        throw new IllegalArgumentException("参数不合法");
    }

    if (size == data.length) {
        resize(2 * size);
    }
    for (int i = size; i >= index; i--) {
        data[i + 1] = data[i];
    }
    data[index] = e;
    size++;
}
```
向数组中指定位置添加数据，首先进行数据校验和数组扩容的校验，之后就是向数组中插入元素，具体思路就是先将数组中 `index` 位置之后的元素都进行向后移动一位，然后再将 `index` 位置的元素使用新的元素替换掉，最后执行 `size++`。


> 值得分析的方法 `remove`:

```java
public E remove(int index) {
    if (index < 0 || index >= size) {
        throw new IllegalArgumentException("移出失败，index 非法");
    }

    E ret = data[index];
    for (int i = index + 1; i < size; i++) {
        data[i - 1] = data[i];
    }
    size--;
    data[size] = null;  // loitering objects != memory leak 闲荡的对象 != 内存溢出
    if (size == data.length / 4 && data.length / 2 != 0) {
        resize(data.length / 2);
    }
    return ret;
}
```

`remove` 同 `add` 方法正好相反，是将元素从数组中移出，并返回。移出思路也很简单，先进行参数校验，然后将 `index` 位置的元素取出保存到临时变量里面用于返回，然后将 `index` 后面的元素依次向前移动一位，直到 `size`，再执行 `size--`。

执行 `size--` 之后，其实 `data[size]` 还是有值的，如果我们再执行 `data[size] = null` 这样可以加快 `Java` 对象的垃圾回收，多说一嘴，其实不执行也是没有问题的，如果执行的赋值为空这句代码之后，原来的数据就变成 `loitering objects(闲荡对象)`，在 `Java` 执行垃圾回收时，就将其回收掉了，这里要注意的是，`loitering objects != memory leak(内存溢出)` 。

如果数组中移出元素比较多之后，再对数组进行缩容，数组即可以扩容，也可以缩容，节约内存的使用。

关于 `if (size == data.length / 4 && data.length / 2 != 0) ` 还是非常有分析意义的，这等到数组的复杂度分析时，具体再详细说。

* 值得分析的方法 `resize`:

```java
private void resize(int newCapacity) {
    E[] newData = (E[]) new Object[newCapacity];

    for (int i = 0; i < size; i++) {
        newData[i] = data[i];
    }
    data = newData;
}
```

这个方法分析的意义不大，一看就能明白，其实就是新开辟一个数组，然后将元素依次移到新的数组中，那为啥还单独说一下呢，关键就是对 `newCapacity` 参数的把控。

就拿扩容来说吧，`newCapacity` 是在原来的基础上增加几个元素呢，还是 `1000` 个, `10000` 个，还是多少倍呢？假如是在原来的基础上增加 `10` 个，那实际上可能需要添加很多, `10` 个根本不够，此时就会产生多次数据移动，这样性能消耗就很大。假如是在原来基础上增加 `10000` 个，但是实际上可能只用了几个，这样就会极大的浪费了内存空间，所以综合所述，这两种极端的方式均是不可取的。

可以设置成 `2倍`，`1.5倍`，`3倍` 可能更合适一点，当然了此处也没有固定值。


# 2. 简单时间复杂度分析

### 添加操作 O(n)

* addLast(e)

在不考虑扩容的情况下，直接向 `data[size]` 赋值即可，所以是 `O(1)`，也就意味着向数组末尾添加元素所消
耗的时间和数组的大小(size)是没有关系的，都能在常数时间内完成添加操作。

* addFirst(e)

同样在不考虑扩容的情况下，向数组中的头部添加元素，需要将其他元素向后移动一位，所以对应的时间复杂度是
`O(n)`，即随着数组大小的增加，所消耗的时间也就越多。

* add(index, e)

同样在不考虑扩容的情况下，向数组中添加元素所消耗的时间和 `index` 取值有关，在极端的情况下，如果 `index=size` 时间复
杂度则是 `O(1)`，如果 `index=0` 则是 `O(n)`，但是此处 `index` 取值是 `0~size` 任意值，并且取得
`0~size` 值的概率都是相等的，所以此处所消耗的时间应该是 `index` 取值的期望，此处涉及数据概率相关知识
，有兴趣的同学可以查询相关资料。可简单计算其时间复杂度是 `O(n/2)`，时间复杂度一般省略常数，所以添加
操作的时间复杂度是: `O(n)`

综上，可得出如下结论:


| 操作          | 时间复杂度    | 备注           |
|---------------|---------------|----------------|
| addLast(e)    | O(1)          | 不涉及数据移动 |
| addFirst(e)   | O(n)          | 涉及数据移动   |
| add(index, e) | O(n/2) = O(n) | 涉及数据移动   |


从表中来看数组的添加操作整体是 `O(n)`，**通常**时间复杂度分析是关注最糟糕的情况。通常在现实生活中也是如此，
如从家到公司上班路程最快需要 5 分钟，如果每次上班只提前 5 分钟的话，那样迟到的概率是很高的。

再比如在考试的时候，理论上所有的选择题都可以使用蒙的方式是可以作对的，但是这个可能性太低了，难道考试
的时候就不准备了，靠蒙就行吗？所以在大多数情况下，考虑最好的情况意义是不大的。

### 删除操作 O(n)

可根据添加操作得出删除操作的时间复杂度。如下：


| 操作             | 时间复杂度    | 备注           |
|------------------|---------------|----------------|
| removeLast(e)    | O(1)          | 不需要移动元素 |
| removeFirst(e)   | O(n)          | 需要移动元素   |
| remove(index, e) | O(n/2) = O(n) | 需要移动元素   |

所以删除的时间复杂度整体是 `O(n)`

## 修改操作 O(1)

* set(index, e)

根据索引就可以找到对应的元素，所以时间复杂度是 `O(1)` 这也是数组的优势，支持随机访问。

### 查找操作 O(n)

可根据以上可知查找操作的时间复杂度，如下表：

| 操作        | 时间复杂度 | 备注           |
|-------------|------------|----------------|
| get(index)  | O(1)       | 不需要遍历元素 |
| contains(e) | O(n)       | 涉及元素遍历   |
| indexOf(e)  | O(n)       | 涉及元素遍历   |

所以查询的时间复杂度整体是 `O(n)`

综上：

| 增 | O(n)                      |
|----|---------------------------|
| 增 | O(n)                      |
| 删 | O(n)                      |
| 改 | 已知索引O(1);未知索引O(n) |
| 查 | 已知索引O(1);未知索引O(n) |

所以在使用数组时，尽量让其索引含有一定的语义, 这样就能很好提高数组的性能。

了解完数组了，再看看链表


