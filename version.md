---
title: "版本管理"
date: 2020-06-24
lastmod: 2020-06-25
authors: ["张丘洋"]
---

## 版本介绍

在 LevelDB 中对 SSTable 进行了版本管理。每当 LevelDB 中的 SSTable 发生变化时（如 Immutable Memtable 转换为 SSTable 或因为 Compaction 导致 SSTable 发生变化）都会产生一个新的版本。每个版本记录了每一层中都有哪些 SSTable 文件。

由于 SSTable 是只读的，且在 Compaction 时被合并的 SSTable 文件并不会删除，所以当一个新的版本生成之后旧的版本依然存在，用户依然可以从旧版本中获取数据。

LevelDB 会删除不再被用户使用的旧版本及其对应的不再被使用的 SSTable 文件。TODO: 在代码中查看旧版本删除相关的代码。

## 数据结构

### Version

LevelDB 使用 `Version` 类来代表一个版本。`Version` 类中主要的成员就是

    std::vector<FileMetaData*> files_[config::kNumLevels];

其中 `FileMetaData` 类用来表示一个文件的元数据，在这里也就代表了一个 SSTable 文件。能够看到，`files_` 保存了版本中所有层的 SSTable 文件信息。

### VersionEdit

`VersionEdit` 类用于表示下一个版本与上一个版本之间的差别，其主要的成员是 `deleted_files_` 和 `new_files_`，分别用于表示相较于上一个版本删除的 SSTable 文件及新增的 SSTable 文件。

{{< figure src="version-edit.png" alt="VersionEdit 类" caption="VersionEdit 示意图" >}}

每当 SSTable 发生变化时，LevelDB 都会生成一个 `VersionEdit` 对象。当版本变化完成时，会调用 `VersionSet` 的 `LogAndApply()` 函数，新的 `VersionEdit` 对象是这个函数的参数。正如这个函数的名字描述的那样，该函数主要做两件事：Log（写日志）和 Apply（应用 `VersionEdit` 生成新的 `Version`。其中写日志并不是向 LevelDB 的 WAL 日志中写入，而是向 Manifest 文件中写入。要注意的是，这里日志写也是先进行的，以免发生宕机等故障。

LevelDB 中存在 Manifest 文件来保存当前数据库的基本信息，用于下次打开数据库时能够根据该文件进行恢复。Manifest 文件和日志文件一样，在写的时候也是向后追加的。当版本发生改变时，会将变化对应的 `VersionEdit` 对象进行编码，将编码后的 `VersionEdit` 内容追加到 Manifest 文件中。

### VersionSet

LevelDB 使用 `VersionSet` 来保存由于 SSTable 变化而生成的一系列版本。`VersionSet` 中 `Version` 构成的双向链表，这些 `Version` 按时间顺序先后产生，记录了对应版本的元信息，链表头指向当前最新的 `Version`，同时维护了每个 `Version` 的引用计数，被引用中的 `Version` 不会被删除，其对应的 SSTable 文件也因此得以保留，通过这种方式，使得 LevelDB 可以在一个稳定的快照视图上访问文件。`VersionSet` 中除了 `Version` 的双向链表外还会记录一些如 `LogNumber`，`Sequence`，下一个 SSTable 文件编号的状态信息等。

{{< figure src="version-set.png" alt="VersionSet 类" caption="Version 类示意图" >}}

在 LevelDB 启动的时候，会建立一个 `VersionSet` 类，并从 Manifest 文件中读取一个初始的 `Version` 状态。之后在 `DBImpl::Recover()` 函数中会从 Manifest 文件中依次读取之前记录的 `VersionEdit` 对象，并将 `VersionEdit` 对象顺序地应用到初始化的版本上来获得最新的版本。

为了避免在恢复最新的版本时生成过多的中间临时 `Version` 对象，`VersionSet` 类引入了 `VersionSet::Builder` 帮助类来快速的恢复出最新的版本。在使用时，将 `VersionEdit` 顺序地应用到 `Builder` 中，最终使用 `Builder::SaveTo()` 函数来生成最新的版本。

{{< figure src="version-builder.png" alt="VersionSet::Builder" caption="Builder 示意图" >}}

## 为什么要引入版本管理

LevelDB 提供了快照（Snapshot）来访问统一的数据。快照实际上就是一个序列号，在按快照访问时只能查询到序列号不大于快照序列号的数据。引入快照就要求 LevelDB 不能够随意的删除旧的 SSTable 文件，因为当前使用中的快照其数据可能保存在旧的 SSTable 文件中。这样就让旧文件的清理变得困难，如果不清理旧文件那么旧文件会越来越多，占用大量的空间；如果旧文件的清理过于频繁就可能导致依然被快照使用的旧文件被删除。而版本管理就可以解决这个问题，通过引入版本来确定当前快照对应的 SSTable 文件，当有快照使用时该版本就不能够被删除。当该版本对应的快照都结束使用时该版本就可以被清理，其对应的旧文件也就可以被删除。
