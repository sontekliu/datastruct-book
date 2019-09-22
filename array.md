# 从数组说起

在我们日常开发中，使用 `List` 但是你能自己手写一个 `List` 吗？看过 `Java` 源代码的都知道，`Java` 中
的 `ArrayList` 底层是使用数组实现的，那我们就自己通过数组来实现自己的 `List` 具体代码见 [码云](https://gitee.com/sontekliu/datastruct-core/blob/master/src/main/java/com/javaliu/collection/array/Array.java) 和
[Github](https://github.com/sontekliu/datastruct-core/blob/master/src/main/java/com/javaliu/collection/array/Array.java) 

其实实现自己的 `List` 不是我们的目的，关键是我们怎么实现，以及其中的复杂度分析。

* 值得分析的方法 `add`:

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
向数组中指定位置添加数据，首先进行数据校验和数组扩容的校验，之后就是向数组中插入元素，具体思路就是
先将数组中 `index` 位置之后的元素都进行向后移动一位，然后再将 `index` 位置的元素使用新的元素替换掉
，最后执行 `size++`


* 值得分析的方法 `remove`:

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

`remove` 同 `add` 方法正好相反，是将元素从数组中移出，并返回。移出思路也很简单，先进行参数校验，然后
将 `index` 位置的元素取出保存到临时变量里面用于返回，然后将 `index` 后面的元素依次向前移动一位，直到
`size`，再执行 `size--`，执行 `size--` 之后，其实 `data[size]` 还是有值的，如果我们再执行 `data[size] = null` 这样可以加快 `Java` 对象的垃圾回收，多说一嘴，其实不执行也是没有问题的，如果执行的赋值为空这句代码之后，原来的数据就变成 `loitering objects(闲荡对象)`，在 `Java` 执行垃圾回收时，就将其回收掉了，但是 `loitering objects != memory leak(内存溢出)` 。如果数组中移出元素比较多之后，再对数组进行缩容，数组即可以扩容，也可以缩容，减少内存的使用。关于 `if (size == data.length / 4 && data.length / 2 != 0) ` 还是非常有分析意义的，这等到数组的复杂度分析时，具体再详细说。

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

这个方法分析的意义不大，一看就能明白，其实就是新开辟一个数组，然后将元素依次移到新的数组中，那为啥还
单独说一下呢，关键就是对 `newCapacity` 参数的把控。

就拿扩容来说吧，`newCapacity` 是在原来的基础上增加几个元素呢，还是 `1000` 个，还是 `10000` 个，还是
多少倍？假如是在原来的基础上增加 `10` 个，那实际上可能需要添加很多 `10` 个根本不够，此时就会产生多次
数据移动，这样性能消耗就很大。假如是在原来基础上增加 `10000` 个，但是实际上可能只用了几个空间，这样
就会极大的浪费了内存空间，所以综合所述，这两种极端的方式均是不可取的。可以设置成 `2倍`，`1.5倍`，`3倍` 可能感觉更合适一点，当然此处没有固定值。






