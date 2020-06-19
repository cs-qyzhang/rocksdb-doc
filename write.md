---
title: "写流程"
date: 2020-06-19
lastmod: 2020-06-19
authors: []
---

LevelDB 中 Put 和 Delete 都是写操作，其中 Put 是把对应的键值对进行插入或更新，Delete 是删除一个键值对。在删除时 LevelDB 是插入一个该键对应的删除标记，所以也属于写操作。

## WriteBatch

LevelDB 的写操作以 `WriteBatch` 为单位进行写入，即使只有一个写操作也会建立一个 `WriteBatch`。`WriteBatch` 中的操作可以保证原子性。

LevelDB 中使用了**序列号**来保证一致性，每一个`WriteBatch`都会赋予唯一的序列号，序列号递增。当一个`WriteBatch`中的写操作全部完成之后该`WriteBatch`的序列号才会被读请求看到。读请求在读取时会以当前完成的最大的序列号进行查找。这样就保证了一致性。

`WriteBatch`使用字符串来表示其具有的写操作。字符串的编码格式为：

    | sequence | count | record | record | ... |

其中 sequence 为 8 字节，代表这个写批次的序列号，count 为 4 字节，代表这个批次共有多少个写操作。sequence 和 count 一起组成为 `WriteBatch` 的 12 字节大小的头部。在头部之后就是 count 数量个写操作。其中每一个写操作分为两种情况：Put 和 Delete。对于这两个操作，record 的格式为：

    Put:    | kTypeValue | key | value |
    Delete: | kTypeDeletion | key |

其中`kTypeValue`和`kTypeDeletion`都是 1 个字节长度的常量，所以可以根据 record 的
第一个字节来判断当前操作的类型。

`WriteBatch` 里 key 和 value 在编码时使用了以下编码方式：

    | string length | string data |

其中 string length 使用的是 varint 编码。

`WriteBatch` 借助了 `WriteBatchInternal` 类来实现其方法。`WriteBatchInternal` 中包含的都是静态成员函数。

## 追加日志 WAL（Write Ahead Log）

LevelDB 的写操作主要分为两步，第一步向日志中写入，第二步向 Memtable 中写入。先写日志是为了避免故障发生后数据的丢失。

LevelDB 的日志是采用追加的方式写入的，为二进制文件，以块为单位进行存取，每块的大小为32KB。每个`WriteBatch`都会编码为日志中的一条或多条记录。日志文件的格式如下图所示。

{{< figure src="log-format.png" alt="日志文件格式" caption="日志文件格式" >}}

其中 checksum、length、type 组成一个记录的头部。checksum 使用 crc 校验，以小端形式存放，校验范围是这个记录的 type 和 data。

type 共有四种：FULL、FIRST、MIDDLE、LAST。如果一个`WriteBatch`编码后的大小小于当前块剩余的空间，则这个`WriteBatch`会以一个记录的形式进行存放，这个记录的类型就是 FULL。如果`WriteBatch`编码后的大小大于当前块剩余的空间，则会将这个`WriteBatch`分散成多个记录，第一个记录占据当前块剩余的空间，中间的记录每个记录都会占满一个块，最后一条记录则占据最后一块的前一部分。第一条记录的类型是 FIRST，若存在中间的记录则类型是 MIDDLE，最后一条记录的类型是 LAST。要注意的是，若当前块剩余的空间小于7个字节，则当前块剩余的空间连一个记录的头部（checksum+length+type 共7 个字节）都放不下，则会使用0来填充这个块剩余的空间，这个记录会从下一块开始存放。若当前块剩余的空间正好是7个字节，则会存放一个长度为 0 的记录。

`db/log_reader.{cc,h}` 和 `db/log_writer.{cc,h}` 包含了日志读和日志写操作的代码。

## Memtable

写操作在写入日志文件之后，就会把对应的数据插入到 Memtable 中。LevelDB 的 Memtable 使用的是**跳表**数据结构，实现位于`db/skiplist.h`中。跳表节点使用以下方式保存一个写操作。

    SkipList Node := KeyLength + InternalKey + ValueString
    InternalKey := UserKey + SequenceNum + Type
    Type := kTypeDeletion or kTypeValue
    ValueString := ValueLength + Value

{{< figure src="skip-list.png" alt="跳表" >}}

## 键的几种表示

在 LevelDB 中一共有以下几种键的表示：UserKey, `InternalKey`, `ParsedInternalKey`, `LookupKey`。

1. UserKey。UserKey 表示用户给定的原始键，在 LevelDB 中没有一个单独的类用于表示，一般使用的是 `Slice` 表示 UserKey。
2. `InternalKey`。`InternalKey` 表示 LevelDB 内部使用键类型，对原始键进行了封装，增加了**序列号**和**类型**两个内容。其中**类型**包括 `kTypeDeletion` 和 `kTypeValue` 两种，分别表示 Put 和 Delete 操作。`InternalKey` 内使用字符串来表示编码后的键，其编码方式为：
    UserKey + ((SequenceNum << 8) | Type)
其中序列号和类型一共占用了 8 字节长度。
3. `ParsedInternalKey`。`ParsedInternalKey` 类用于简化对 `InternalKey` 的操作，`InternalKey` 中使用字符串对原始键、序列号和类型进行编码，而 `ParsedInternalKey` 则将原始键、序列号和类型通过类的成员进行表示。

    ```C++
    struct ParsedInternalKey {
      Slice user_key;
      SequenceNumber sequence;
      ValueType type;
    };
    ```

4. `LookupKey`。`LookupKey` 用于 Get 操作，其在 `InternalKey` 的基础上又添加了 UserKey 的长度。`LookupKey` 的编码如下所示。
    | klength (varint32) | userkey (char[klength]) | tag (uint64) |
其中的 tag 和 `InternalKey` 中的后 8 位一样，为 `((SequenceNum << 8) | Type)`。
能够看到 `LookupKey` 的表示和 Memtable 中跳表节点所保存的前半部分内容一致，所以 `LookupKey` 可以作为在 Memtable 查找的键类型。另外 `LookupKey` 若去掉了最前面的 klength 则就转化为了 `InternalKey` 类型。

Memtable 使用 `LookupKey` 在查找时会进行键的比较，而在比较时会按照序列号的**降序**进行查找，也就是序列号大的优先级高。而序列号和键的类型（`kTypeDeletion` 或 `kTypeValue`）是编码到一起的：`((SequenceNum << 8) | Type)`，所以在构造 `LookupKey` 进行 Get 操作时键的类型会设置为enum中值最大的那个，在这里值最大的是`kTypeValue`。

## SST 文件（Sorted String Table）


