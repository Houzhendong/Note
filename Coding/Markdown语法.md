# Markdown语法

## 标题

``` markdown
# header level 1
## header level 2
### header level 3
#### header level 4
##### header level 5
###### header level 6
```

## 段落

- 用空白行分隔段落 或者用两个以上空格加回车分隔段落

```markdown
This is first paragraph

This is second paragraph
```

* 字体

```markdown
*斜体*
_斜体_
**粗体**
__粗体__
***粗斜体文本***
___粗斜体文本___
```

* 分割线

```markdown
***
---
```

* 删除线

```markdown
~~删除线~~  双波浪线中间加文本
```

* 下划线

```markdown
<u>underscore</u>
```

* 脚注

```markdown
[^Footnotes]
[^Footnotes]: this is a footnote
```

## 列表

* 无序列表

  ```markdown
  * first
  * second
  * third
  
  - first
  - second
  - third
  
  + first
  + second
  + third
  ```

* 有序列表

  ```markdown
  1. first
  2. second
  3. third
  ```

* 列表嵌套

  列表嵌套只需在子列表中的选项前面添加四个空格即可

  ```markdown
  1. 第一项：
      - 第一项嵌套的第一个元素
      - 第一项嵌套的第二个元素
  2. 第二项：
      - 第二项嵌套的第一个元素
      - 第二项嵌套的第二个元素
  ```


## 区块

  在段落开头使用`>`,然后紧跟一个空格

  ```markdown
  > this is quote
  ```

  > this is quote 

  ## 代码

* 代码语句

  ```markdown
  `print("hello world")`
  ```

* 代码块

  ```markdown
  ​```语言
  print("hello world")
  ​```
  ```

## 链接

```markdown
[link name](link url)

<link url>
```

## 图片

```markdown
![image title](image link)
```

## 表格

```markdown
| header | header |
| ----   | ----   |
| cell   | cell   |

:----  左对齐
:----: 居中
----:  右对齐
```

  ## 公式

```markdown
$$
latex code
$$
```

|  描述  |        语法         |
| :----: | :-----------------: |
| 上下标 |   `a^x`  / `a_x`    |
| 平方根 |     `\sqrt{x}`      |
|  向量  |      `\vec a`       |
|  分数  | `\frac{分子}{分母}` |
|  求和  |   `\sum_{i=1}^n`    |
|  积分  |  `\int_0^{\pi/2}`   |

