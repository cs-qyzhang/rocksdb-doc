---
title: "Hugo 用法"
date: 2020-05-15
lastmod: 2020-05-17
authors: ["张丘洋"]
---

Hugo可以使用多种标记式语言编写，如Markdown, reStructuredText, AsciiDoc, HTML等。下面主要介绍Hugo中Markdown的用法。

Markdown的基本语法这里不再进行介绍，可自行搜索学习。

## Front Matter

Front Matter是Hugo中用来表示文档的元数据，如题目、时间、文档类型、分类等。Front Matter在每个文档的开始，可以使用YAML, TOML, JSON或ORG这四种格式来表示。其中YAML是使用`---`分隔的，TOML是使用`+++`分隔的。该文档采用的就是YAML格式。在编写文档时，请保证每个文档的Front Matter都有**title**（标题）, **date**（创建日期）, **lastmod**（最后一次修改日期）和**authors**（作者）这四种信息，如：

```yaml
---
title: "文档编写方法"
date: 2020-05-15
lastmod: 2020-05-16
authors: ["周树人", "鲁迅"]
---
```

## 目录组织

Hugo原有的目录组织方式比较麻烦，需要在每一个文件的Front Matter中指定所在的章节以及章节中的次序，这样会造成难于修改目录组织方式。为了方便进行目录组织，我修改了模板生成目录的方式，加入了`table-of-contents.md`这一文件，之后只需要在该文件中进行目录组织即可。

`table-of-contents.md`应该具有以下形式（注意该文件不需要Front Matter）：

```md
## _index.md

## "准备工作"

### hugo-usage.md

### install.md

## data-structure.md

### slice.md

### cache.md
```

这里##代表的是目录的一级标题，###代表的是目录的二级标题，目录的层次和顺序按照在该文件中出现的顺序指定。若一个目录项对应的是一个.md文件，则应该写上该文件的文件名。若一个目录项没有实际的内容，只是作为一个大标题，那么可以直接用双引号将标题括起来，如上面的"准备工作"。

如果用.md文件代表一个目录项那么该目录项的标题默认为.md文件Front Matter中指定的title。如果在.md文件的Front Matter中指定了linktitle这一属性，那么目录项对应的标题就变成了linktitle对应的内容。比如`_index.md`文件的title是“RocksDB”，linktitle是“概述”，那么在目录中出现的题目就是“概述”而不是“RocksDB”。

{{< alert note >}}
文件名在命名时请使用小写英文字母，且使用连字符`-`进行连接而不要使用下划线`_`。比如要使用`hugo-usage.md`而不是`hugo_usage.md`。
{{< /alert >}}

{{< alert warning >}}
`_index.md`是一个特殊的文件，指定了该文档的首页，请不要修改该文件的文件名
{{< /alert>}}

## 插入图片

有两种方法可以插入图片：

**使用Markdown语法**：

    ![example](example.png)

![example](example.png)

**使用Hugo短代码**：

    {{</* figure src="example.png" title="仙剑奇侠传" alt="example" */>}}

{{< figure src="example.png" title="仙剑奇侠传" alt="example" >}}

建议使用Hugo短代码，不仅能够添加标题，而且图片也可以放大查看。

使用Hugo短代码时还可以添加其他的属性如宽度width或高度height：

    {{</* figure src="example.png" title="仙剑奇侠传" width=70% */>}}

{{< figure src="example.png" title="仙剑奇侠传" width=70% >}}

{{< alert note >}}
**注意**：图片要放在与该文档名称相同的文件夹下。
{{< /alert >}}

## 链接与交叉引用

带名字的链接：`[Hugo官网](https://gohugo.io/)` -> [Hugo官网](https://gohugo.io/)

没有名字的链接：`<https://gohugo.io/>` -> <https://gohugo.io/>

同一文档交叉引用：`[图片插入](#插入图片)` -> [图片插入](#插入图片)

不同文档交叉引用：`[编译]({{</* relref "install.md#编译" */>}})` -> [编译]({{< relref "install.md#编译" >}})

## 代码

    ```C++
    #include <iostream>

    using namespace std;

    int main(void) {
        cout << "hello, hugo!" << endl;
        return 0;
    }
    ```

会生成：

```C++
#include <iostream>

using namespace std;

int main(void) {
    cout << "hello, hugo!" << endl;
    return 0;
}
```

## 句子引用

    > All problems in computer science can be solved by
    > another level of indirection, except for the problem
    > of too many layers of indirection. – David J. Wheeler

会生成：

> All problems in computer science can be solved by
> another level of indirection, except for the problem
> of too many layers of indirection. – David J. Wheeler

## 表格

    | Format                            | Type              |
    |-----------------------------------|-------------------|
    | `January 2, 2006`                 | Date              |
    | `01/02/06`                        |                   |
    | `Jan-02-06`                       |                   |
    | `Mon, 02 Jan 2006`                |                   |
    | `Monday, 02 Jan 2006`             |                   |
    | `15:04`                           | Time              |
    | `3:04 PM`                         |                   |
    | `15:04 MST`                       |                   |

会生成：

| Format                            | Type              |
|-----------------------------------|-------------------|
| `January 2, 2006`                 | Date              |
| `01/02/06`                        |                   |
| `Jan-02-06`                       |                   |
| `Mon, 02 Jan 2006`                |                   |
| `Monday, 02 Jan 2006`             |                   |
| `15:04`                           | Time              |
| `3:04 PM`                         |                   |
| `15:04 MST`                       |                   |

## 代办事项

    - [x] 吃饭
    - [x] 睡觉
    - [ ] 打豆豆

会生成：

- [x] 吃饭
- [x] 睡觉
- [ ] 打豆豆

## 脚注

    本网站使用的是Hugo框架[^1]，主体采用的为Academic[^2]。
    [^1]: <https://gohugo.io/>
    [^2]: <https://sourcethemes.com/academic/>

会生成：

本网站使用的是Hugo框架[^1]，主体采用的为Academic[^2]。
[^1]: <https://gohugo.io/>
[^2]: <https://sourcethemes.com/academic/>

可以使用脚注来引用参考文献。

## LaTeX 数学公式

```tex
$$\gamma_{n} = \frac{ \left | \left (\mathbf x_{n} - \mathbf x_{n-1} \right )^T \left [\nabla F (\mathbf x_{n}) - \nabla F (\mathbf x_{n-1}) \right ] \right |} {\left \|\nabla F(\mathbf{x}_{n}) - \nabla F(\mathbf{x}_{n-1}) \right \|^2}$$
```

会生成：

$$\gamma_{n} = \frac{ \left | \left (\mathbf x_{n} - \mathbf x_{n-1} \right )^T \left [\nabla F (\mathbf x_{n}) - \nabla F (\mathbf x_{n-1}) \right ] \right |} {\left \|\nabla F(\mathbf{x}_{n}) - \nabla F(\mathbf{x}_{n-1}) \right \|^2}$$

## 警告⚠️框

    {{</* alert note */>}}
    **警方提示：**

    道路千万条，安全第一条，行车不规范，亲人两行泪
    {{</* /alert */>}}

{{< alert note >}}
**警方提示：**

道路千万条，安全第一条，行车不规范，亲人两行泪
{{< /alert >}}

    {{</* alert warning */>}}
    Here's some important information...
    {{</* /alert */>}}

{{< alert warning >}}
Here's some important information...
{{< /alert >}}
