---
title: Strings, bytes, runes and characters in Go
date: 2013-10-23
by:
- Rob Pike
tags:
- strings
- bytes
- runes
- characters
summary: How strings work in Go, and how to use them.
---

## Introduction

[上篇博客](https://blog.golang.org/slices) 阐述了切片在Go中如何工作, 并列举了很多例子来说明它的机制。
基于这个背景,我们将在这篇博客中讨论Go中字符串。
首先,对于一篇博客来说字符串这个话题似乎太简单了,但要用好字符串不仅要理解字符串是怎么工作的,
还要掌握一个byte,一个character,一个rune的区别,
Unicode和UTF-8的区别,
字符串和字符串字面量的区别,
以及其他更细微的区别。

只有一种方法可以让你切入这个话题, 那就是思考一些常见问题的答案,
如 "为什么我不能在Go字符串的第n个位置,得到第n个字符?"
如你所见，这个问题会引导我们了解有关文本在现代世界中如何工作的许多细节。

对其中一些问题的精彩介绍，是 Joel Spolsky 的著名博文,[The Absolute Minimum Every Software Developer Absolutely, Positively Must Know About Unicode and Character Sets (No Excuses!)](http://www.joelonsoftware.com/articles/Unicode.html)。
他提出的许多观点将在这里得到回应。

## What is a string? 什么是字符串？

让我们先从一些基础的东西开始

在Go中,字符串时间上是只读的字节切片。
如果你完全不确定字节切片是什么以及它是如何工作的，请阅读[上篇博客](https://blog.golang.org/slices)；
我们假设你有这些知识。

重要的是要预先声明。
首先我们要在前面声明一个可以包含任意字节的字符串。
它不需要保存 Unicode 文本、UTF-8文本或任何其他预定义格式。
就字符串的内容而言，它完全等同于一个字节切片。

Here is a string literal (more about those soon) that uses the
`\xNN` notation to define a string constant holding some peculiar byte values.
(Of course, bytes range from hexadecimal values 00 through FF, inclusive.)

这里是使用`\xNN` 符号来定义一个包含一些特殊字节值的字符串常量。
（字节范围从十六进制值 00 到 FF，包括在内。）
```Go
	const sample = "\xbd\xb2\x3d\xbc\x20\xe2\x8c\x98"
```

## 打印字符串

Because some of the bytes in our sample string are not valid ASCII, not even
valid UTF-8, printing the string directly will produce ugly output.
The simple print statement


因为我们sample字符串有些字符并不是有效的ASCII字符,甚至不是有效的UTF-8字符,
直接打印它们会导致你的输出非常难看。这个简单的打印模式
```Go
	fmt.Println(sample)
```

会导致这种乱码 (它的最终外观因环境而异):

	��=� ⌘

为了搞清楚字符串包含了什么东西,我们需要把它拆开检查一下。
我们有几种方法可以做这件事情。
一个最明显的方法就算遍历它的内容,打出所有的字符。
比如下面这个`for`循环

```Go
    for i := 0; i < len(sample); i++ {
        fmt.Printf("%x ", sample[i])
    }
```

在前面这个演示中,对字符串进行索引访问的是单个独立字节, 并不是整个字符。
现在,我们只用字节。
整个是从字节到字节的循环

	bd b2 3d bc 20 e2 8c 98

请注意各个字节如何与定义字符串的十六进制转义匹配。

A shorter way to generate presentable output for a messy string
is to use the `%x` (hexadecimal) format verb of `fmt.Printf`.
It just dumps out the sequential bytes of the string as hexadecimal
digits, two per byte.

在`fmt.Printf`中使用`%x`(十六进制)可以将一个有乱码的字符串打印的更美观易看些。
它只是将字符串的连续字节转储为十六进制数字，每个字节两个。

```Go
    fmt.Printf("%x\n", sample)
```

可以用这个打印结果与上面的做对比:

	bdb23dbc20e28c98

有个很好的技巧是在格式化字符串中使用空格标志,在`%`和`x`之间多打一个空格。
可以用下面这个格式化打印和上面比较一下,

```Go
	fmt.Printf("% x\n", sample)
```

注意字节之间的空格是如何出现的:

	bd b2 3d bc 20 e2 8c 98

[还有更多的占位符](https://pkg.go.dev/fmt)。`%q`（带引号的,译注:使用 Go 语法安全转义的单引号字符文字。）
动词将转义字符串中任何不可打印的字节序列，因此输出是明确的。

```Go
    fmt.Printf("%q\n", sample)
```

当一个字符串大部分都是易读的文本时这种方法会很方便,但有一些特殊的字符要移除；它会打印出:

	"\xbd\xb2=\xbc ⌘"

如果仔细观察, 我们可以发现这段乱码中藏了一个ASCII等号，以及一个规则的空格，最后面出现的是著名的瑞典“野营地”符号。
该符号的Unicode值是U+2318，将空格后面的字节编码为UTF-8十六进制值 `20`）：`e2` `8c` `98`。

如果一个字符串中有我们不认识的奇怪的值，我们可以在`%q`占位符使用"+"标志。
此标志在打印UTF-8时，不仅可以转义不可打印的序列，还能转义任何非ASCII字节。
这样，它会打印出一个正确的UTF-8编码的字符串，它包含了非ASCII字节的Unicode值:

```Go
	fmt.Printf("%+q\n", sample)
```

With that format, the Unicode value of the Swedish symbol shows up as a
`\u` escape:
在这个格式化打印中,瑞典符号的Unicode值则被转义为`\u`:

	"\xbd\xb2=\xbc \u2318"

这些打印技巧在调试字符串的内容时很好用，并且在接下来的讨论中会派上用场。
值得指出的是，所有这些方法对**字节切片**的操作与对**字符串**的操作完全一样。

Here's the full set of printing options we've listed, presented as
a complete program you can run (and edit) right in the browser:
这里我编写了一个完整的程序,这个程序列举刚才我们涉及到的所有打印选项, 你可以运行它(原网页可以运行):

```go
package main

import "fmt"

func main() {
    const sample = "\xbd\xb2\x3d\xbc\x20\xe2\x8c\x98"

    fmt.Println("Println:")
    fmt.Println(sample)

    fmt.Println("Byte loop:")
    for i := 0; i < len(sample); i++ {
        fmt.Printf("%x ", sample[i])
    }
    fmt.Printf("\n")

    fmt.Println("Printf with %x:")
    fmt.Printf("%x\n", sample)

    fmt.Println("Printf with % x:")
    fmt.Printf("% x\n", sample)

    fmt.Println("Printf with %q:")
    fmt.Printf("%q\n", sample)

    fmt.Println("Printf with %+q:")
    fmt.Printf("%+q\n", sample)
}
```

[练习：修改上面的示例,用字节切片替换字符串。 提示：使用类型转换来创建切片。]

[练习：遍历字符串,使用`%q`打印每个字节。 你发现了什么？]

## UTF-8 和 字符串字面量

如我们所见, 遍历字符串会返回它的每个字节,而不是字符: 字符串就是一堆字节。
这就意味着,当我们存储一个字符值到字符串中时,我们存储的是组成那个字符的字节。
我们来看一个更加灵活的例子,看看这个过程是如何发生的。

这里有个简单的程序,它用三种方式打印一个字符串常量,
一种是作为一个纯字符串,一种是作为ASCII字符串,一种是作为十六进制的字节。
为了避免任何混淆,我们创建一个“原始字符串”,用反斜杠括起来,
这样它只能包含字面量文本。（用双引号括起来的常规字符串可以包含如上所示的转义序列。）
```Go
func main() {
    const placeOfInterest = `⌘`

    fmt.Printf("plain string: ")
    fmt.Printf("%s", placeOfInterest)
    fmt.Printf("\n")

    fmt.Printf("quoted string: ")
    fmt.Printf("%+q", placeOfInterest)
    fmt.Printf("\n")

    fmt.Printf("hex bytes: ")
    for i := 0; i < len(placeOfInterest); i++ {
        fmt.Printf("%x ", placeOfInterest[i])
    }
    fmt.Printf("\n")
}
```

输出是这样的:

	plain string: ⌘
	quoted string: "\u2318"
	hex bytes: e2 8c 98

我们可以看到Unicode字符值是U+2318, 
瑞典“野营地”符号的字节则是`e2` `8c` `98`,
这些字节是十六进制值 2318 的 UTF-8 编码。

这可能一眼就可以看出来，也可能不那么明显，
这取决于你对UTF-8的熟悉程度，
但值得花一点时间来解释一下如何用UTF-8表示字符串。
一个简单的事实是：它是在编写源码时就定下了。

Source code in Go is _defined_ to be UTF-8 text; no other representation is
allowed. That implies that when, in the source code, we write the text

	`⌘`

the text editor used to create the program places the UTF-8 encoding
of the symbol ⌘ into the source text.
When we print out the hexadecimal bytes, we're just dumping the
data the editor placed in the file.
Go 中的源代码定义为 UTF-8 文本；其他均不被容许。
这意味着，在源码中，我们写入的文本中，编辑器将 UTF-8 编码的符号 ⌘ 放入源文本。
当我们打印出十六进制字节时，我们只是输出编辑器放入文件中的数据。

简而言之，Go源代码采用UTF-8编码，因此源码中的字符串字面量是UTF-8文本。
如果字符串字面量包含未转义字符序列，那么它的构造字符串将只包含引号中的源文本。
这意味着，原始字符串总是包含它的内容的有效UTF-8表示。
因此,经过定义和构造的原始字符串,它的内容总是包含有效的UTF-8表示。
同样，除非它包含UTF-8中断转义序列，否则一般的字符串字面量也将包含有效的UTF-8。

一些人认为Go字符串总是UTF-8，其实不是：只有字符串字面量是UTF-8。
我们在上一节中已经提到，字符串值可以包含任意字节；
我们在这一节中也提到，只要字符串字面量没有字节级别的转义序列，它就总是包含UTF-8文本。

总结一下就是，字符串可以包含任意字节，但当从字符串字面量构造时，这些字节都是(绝大多数情况下)UTF-8。
## 代码点，字符，和 runes

We've been very careful so far in how we use the words "byte" and "character".
That's partly because strings hold bytes, and partly because the idea of "character"
is a little hard to define.
The Unicode standard uses the term "code point" to refer to the item represented
by a single value.
The code point U+2318, with hexadecimal value 2318, represents the symbol ⌘.
(For lots more information about that code point, see
[its Unicode page](http://unicode.org/cldr/utility/character.jsp?a=2318).)

To pick a more prosaic example, the Unicode code point U+0061 is the lower
case Latin letter 'A': a.

But what about the lower case grave-accented letter 'A', à?
That's a character, and it's also a code point (U+00E0), but it has other
representations.
For example we can use the "combining" grave accent code point, U+0300,
and attach it to the lower case letter a, U+0061, to create the same character à.
In general, a character may be represented by a number of different
sequences of code points, and therefore different sequences of UTF-8 bytes.

The concept of character in computing is therefore ambiguous, or at least
confusing, so we use it with care.
To make things dependable, there are _normalization_ techniques that guarantee that
a given character is always represented by the same code points, but that
subject takes us too far off the topic for now.
A later blog post will explain how the Go libraries address normalization.

"Code point" is a bit of a mouthful, so Go introduces a shorter term for the
concept: _rune_.
The term appears in the libraries and source code, and means exactly
the same as "code point", with one interesting addition.

The Go language defines the word `rune` as an alias for the type `int32`, so
programs can be clear when an integer value represents a code point.
Moreover, what you might think of as a character constant is called a
_rune constant_ in Go.
The type and value of the expression

	'⌘'

is `rune` with integer value `0x2318`.

To summarize, here are the salient points:

  - Go source code is always UTF-8.
  - A string holds arbitrary bytes.
  - A string literal, absent byte-level escapes, always holds valid UTF-8 sequences.
  - Those sequences represent Unicode code points, called runes.
  - No guarantee is made in Go that characters in strings are normalized.

## Range loops

Besides the axiomatic detail that Go source code is UTF-8,
there's really only one way that Go treats UTF-8 specially, and that is when using
a `for` `range` loop on a string.

We've seen what happens with a regular `for` loop.
A `for` `range` loop, by contrast, decodes one UTF-8-encoded rune on each
iteration.
Each time around the loop, the index of the loop is the starting position of the
current rune, measured in bytes, and the code point is its value.
Here's an example using yet another handy `Printf` format, `%#U`, which shows
the code point's Unicode value and its printed representation:

{{play "strings/range.go" `/const/` `/}/`}}

The output shows how each code point occupies multiple bytes:

	U+65E5 '日' starts at byte position 0
	U+672C '本' starts at byte position 3
	U+8A9E '語' starts at byte position 6

[Exercise: Put an invalid UTF-8 byte sequence into the string. (How?)
What happens to the iterations of the loop?]

## Libraries

Go's standard library provides strong support for interpreting UTF-8 text.
If a `for` `range` loop isn't sufficient for your purposes,
chances are the facility you need is provided by a package in the library.

The most important such package is
[`unicode/utf8`](/pkg/unicode/utf8/),
which contains
helper routines to validate, disassemble, and reassemble UTF-8 strings.
Here is a program equivalent to the `for` `range` example above,
but using the `DecodeRuneInString` function from that package to
do the work.
The return values from the function are the rune and its width in
UTF-8-encoded bytes.

{{play "strings/encoding.go" `/const/` `/}/`}}

Run it to see that it performs the same.
The `for` `range` loop and `DecodeRuneInString` are defined to produce
exactly the same iteration sequence.

Look at the
[documentation](/pkg/unicode/utf8/)
for the `unicode/utf8` package to see what
other facilities it provides.

## Conclusion

To answer the question posed at the beginning: Strings are built from bytes
so indexing them yields bytes, not characters.
A string might not even hold characters.
In fact, the definition of "character" is ambiguous and it would
be a mistake to try to resolve the ambiguity by defining that strings are made
of characters.

There's much more to say about Unicode, UTF-8, and the world of multilingual
text processing, but it can wait for another post.
For now, we hope you have a better understanding of how Go strings behave
and that, although they may contain arbitrary bytes, UTF-8 is a central part
of their design.
