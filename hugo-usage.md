---
title: "Hugo用法"
date: 2020-05-15
lastmod: 2020-05-16
---

Hugo 可以使用多种标记式语言编写，如Markdown, reStructuredText, AsciiDoc, HTML
等。下面主要介绍Hugo中Markdown的用法。

Markdown的基本语法这里不再进行介绍，可自行搜索学习。

## Front Matter

Front Matter是Hugo中用来表示文档的元数据，如题目、时间、文档类型、分类等。
Front Matter在每个文档的开始，可以使用YAML, TOML, JSON或ORG这四种格式来表示。
其中YAML是使用`---`分隔的，TOML是使用`+++`分隔的。该文档采用的就是YAML格式。
在编写文档时，请保证每个文档的Front Matter都有**title**(标题), **date**(创建
日期), **lastmod**(最后一次修改日期)和**authors**(作者)这四种信息，如：

```yaml
---
title: "文档编写方法"
date: 2020-05-15
lastmod: 2020-05-16
authors: ["周树人", "鲁迅"]
---
```

## 插入图片

有两种方法可以插入图片：

**使用Markdown语法**：

```markdown
![example](example.png)
```

![example](example.png)

**使用Hugo短代码**：

    {{</* figure src="example.png" title="仙剑奇侠传" alt="example" */>}}

{{< figure src="example.png" title="仙剑奇侠传" alt="example" >}}

建议使用Hugo短代码，不仅能够添加标题，而且图片也可以放大查看。

使用Hugo短代码时还可以添加其他的属性如宽度width或高度height：

    {{</* figure src="example.png" title="仙剑奇侠传" width=70% */>}}

{{< figure src="example.png" title="仙剑奇侠传" width=70% >}}

{{< alert note >}}
**注意**：图片要放在与该文档名称相同的文件夹下
{{< /alert >}}

## 链接与交叉引用

带名字的链接：`[Hugo官网](https://gohugo.io/)` -> [Hugo官网](https://gohugo.io/)

没有名字的链接：`<https://gohugo.io/>` -> <https://gohugo.io/>

同一文档交叉引用：`[图片插入](#插入图片)` -> [图片插入](#插入图片)

不同文档交叉引用：`[编译]({{</* relref "compile.md#custom-theme" */>}})` -> [编译]({{< relref "compile.md#custom-theme" >}})

## 代码

```c
#include <stdio.h>

int main(void) {
    printf("hello, hugo!\n");
    return 0;
}
```

```go
import "text/template"
```

## 表格

## 警告⚠️框

```markdown
{{%/* alert note */%}}
**警方提示：**

道路千万条，安全第一条，行车不规范，亲人两行泪
{{%/* /alert */%}}
```

{{% alert note %}}
**警方提示：**

道路千万条，安全第一条，行车不规范，亲人两行泪
{{% /alert %}}

```md
{{%/* alert warning */%}}
Here's some important information...
{{%/* /alert */%}}
```

{{% alert warning %}}
Here's some important information...
{{% /alert %}}
