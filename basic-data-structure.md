---
title: "基本数据结构"
date: 2020-05-28
lastmod: 2020-06-19
authors: ["张丘洋"]
---

## Cleanable

Cleanable 是一个接口类

## Slice

Slice 是 rocksdb 中用于表示变长字符串的数据结构。一个 Slice 的数据表示非常简单，只需要 `char*` 和 `size_t`。Slice 提供了一些函数可供使用，如下表所示。

| 函数名                | 功能                                       |
| --------------------- | ------------------------------------------ |
| `remove_prefix()`     | 去掉前 n 个字符                            |
| `remove_suffix()`     | 去掉后 n 个字符                            |
| `starts_with()`       | 是否包含某一前缀                           |
| `ends_with()`         | 是否包含某一后缀                           |
| `difference_offset()` | 返回两个 Slice 第一个不同的字符偏移        |
| `DecodeHex()`         | 将表示 16 进制数字的字符串转换为对应的数据 |

### SliceParts

SliceParts 代表由多个 Slice 逻辑拼接而成的字符串。通过一个 Slice 数组和数组的长度来表示。

### PinnableSlice

PinnableSlice 是带有一些清理工作的 Slice。它继承了 Slice 和 Cleanable 类。当对象调用 Reset() 函数或者对象析构时会调用对应的清理函数。PinnableSlice 可以用来避免 memcpy：一个 PinnableSlice 字符串会被锁在内存中，当数据被消化时才会释放。

## varint

为了保存数据时**节约空间**并且**避免大小端**带来的歧义，rocksdb 使用了变长整数 varint 来保存数据。varint 可以说是一种数据编码方式，具体的编码方式为：将每个整数从低位开始存储，每7个比特保存为一个字节，直到高位全部为0。编码中每个字节的最高位代表编码是否结束，若是1则代表编码还没有结束，若是0则代表编码结束。编码示例如下图所示。

{{< figure src="varint.svg" alt="varint编码" >}}

使用 varint 编码时一个字节包含的有效比特数为7，所以`uint32_t`最多需要**5**个字节，`uint64_t`最多需要**10**个字节。

在编码**有符号数**时，为了减少存储空间，使用了 ZigZag 编码方法。ZigZag 编码主要是在 Google Protocol Buffers 中提出的，它的编码方式如下：将有符号数向左移1位，之后让每一位与符号位进行异或来得到最终的结果。如果是64位有符号数，则编码方式为：

    (n << 1) ^ (n >> 63)

这里的右移是**算数右移**，即左边补符号位。这样的方法主要是为了让绝对值小的数有一个较小的编码长度。比如对于正数1来说，varint 编码只需要1字节，然而对于负数-1来说，varint 编码就需要10字节（假设正整数为 int64_t），因为-1的所有位都是1。而使用了 ZigZag 编码后1变成了2，-1变成了1，虽然对于正数来说多了一位，但对于小的负数来说却少了很多位，使用 varint 只需要很少的字节即可。

| ZigZag 编码前 | ZigZag 编码后 |
| ------------- | ------------- |
| 0             | 0             |
| -1            | 1             |
| 1             | 2             |
| -2            | 3             |
| 2             | 4             |
| 2147483647    | 4294967294    |
| -2147483648   | 4294967295    |

ZigZag 的说明详见 [维基百科](https://en.wikipedia.org/wiki/Variable-length_quantity#Zigzag_encoding) 和 [Protocol Buffers](https://developers.google.com/protocol-buffers/docs/encoding#signed-integers)

## 内存池 Arena

LevelDB 为了减少频繁的小内存分配带来的较多的系统调用，LevelDB 使用了内存池来管理小内存的分配。内存池以`Arena`类实现，该内存池在分配小内存时以块为单位，每块大小4KB。在分配小内存时会根据需要内存的大小进行不同的分配，若需要的内存超过块的1/4，则直接新分配一个需要大小的内存。而在分配小内存时，若当前用于分配小内存的块内剩余的空间足够使用，则直接将该块中空闲的空间分配出去。若不够则重新分配一个块，并将该块中对应大小的内存分配出去。

使用内存池时分配的内存不能够显式地回收（delete），只能在内存池回收时被释放。
