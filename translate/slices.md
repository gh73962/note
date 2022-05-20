---
title: "Arrays, slices (and strings): The mechanics of 'append'"
date: 2013-09-26
by:
- Rob Pike
tags:
- array
- slice
- string
- copy
- append
summary: How Go arrays and slices work, and how to use copy and append.
---

## Introduction

One of the most common features of procedural programming languages is
the concept of an array.
Arrays seem like simple things but there are many questions that must be
answered when adding them to a language, such as:

程序编程语言最常见的特征之一是数组的概念。
数组看起来很简单，但在将它们添加到语言时必须回答许多问题，例如：

  - fixed-size or variable-size? (固定长度还是可变长度?)
  - is the size part of the type? (size是其他类型的一部分吗?)
  - what do multidimensional arrays look like? (多维数组是什么样的)
  - does the empty array have meaning? (空数组意味着什么)

The answers to these questions affect whether arrays are just
a feature of the language or a core part of its design.

这些问题的答案会影响数组是否只是语言的一个特性还是其设计的核心部分。

In the early development of Go, it took about a year to decide the answers
to these questions before the design felt right.
The key step was the introduction of _slices_, which built on fixed-size
_arrays_ to give a flexible, extensible data structure.
To this day, however, programmers new to Go often stumble over the way slices
work, perhaps because experience from other languages has colored their thinking.

在早期的Go开发中, 在这个设计成熟之前, 这些问题花费我们快一年的时间.关键的一步是引入切片, 切片是一种灵活易拓展的数据结构.然而，直到今天，刚接触 Go 的程序员经常对切片的工作方式感到困惑，也许是因为其他语言的经验影响了他们的思维。

In this post we'll attempt to clear up the confusion.
We'll do so by building up the pieces to explain how the `append` built-in function works, and why it works the way it does.

我们将在这篇博客中为你答疑解惑.
还会阐述append内置函数是如何工作的
## Arrays

Arrays are an important building block in Go, but like the foundation of a building they are often hidden below more visible components.
We must talk about them briefly before we move on to the more interesting,powerful, and prominent idea of slices.
在Go中,数组是一个非常重要的基本构成元素,像建筑的桩基一样,他们都是隐藏在可见的组件之下.
在我们继续讨论更有趣、更强大、更突出的切片概念之前，我们必须简要讨论一下它们。

Arrays are not often seen in Go programs because
the size of an array is part of its type, which limits its expressive power.

数组在Go中并不常见,因为数组的大小是它类型的一部分,这个特性限制了他们的表现力

The declaration
`var buffer [256]byte`,
declares the variable `buffer`, which holds 256 bytes.
The type of `buffer` includes its size, `[256]byte`.
An array with 512 bytes would be of the distinct type `[512]byte`.

The data associated with an array is just that: an array of elements.
Schematically, our buffer looks like this in memory,

	buffer: byte byte byte ... 256 times ... byte byte byte

That is, the variable holds 256 bytes of data and nothing else. We can
access its elements with the familiar indexing syntax, `buffer[0]`, `buffer[1]`,
and so on through `buffer[255]`. (The index range 0 through 255 covers
256 elements.) Attempting to index `buffer` with a value outside this
range will crash the program.

也就是说，该变量只保存 256 个字节的数据，仅此而已。 我们可以
使用熟悉的下标索引语法访问其元素，`buffer[0]`, `buffer[1]`,
等等通过`buffer[255]`。 （索引范围 0 到 255 涵盖
256 个元素。）尝试使用超出此范围的值来索引“缓冲区”
会使程序崩溃。

There is a built-in function called `len` that returns the number of elements
of an array or slice and also of a few other data types.
For arrays, it's obvious what `len` returns.
In our example, `len(buffer)` returns the fixed value 256.

这里有个内置函数`len` ,它会返回数组\切片\或其他类型的元素数量.对于数组来说,`len`返回了什么一目了然.在我们的示例中,`len(buffer)`返回了固定值256.

Arrays have their place—they are a good representation of a transformation
matrix for instance—but their most common purpose in Go is to hold storage
for a slice.

~~数组有自己的位置，例如，它们是转换矩阵的良好表示，但它们在 Go 中最常见的用途是为切片保存存储空间.~~

## Slices: The slice header

Slices are where the action is, but to use them well one must understand 
exactly what they are and what they do.

A slice is a data structure describing a contiguous section of an array
stored separately from the slice variable itself.
_A slice is not an array_.
A slice _describes_ a piece of an array.

切片是一种数据结构，描述了一个数组的连续部分,但不与切片变量本身存储在一起。
一个切片不是一个数组, 一个切片描述了一个数组的一部分.

Given our `buffer` array variable from the previous section, we could create
a slice that describes elements 100 through 150 (to be precise, 100 through 149,
inclusive) by _slicing_ the array:

	var slice []byte = buffer[100:150]

In that snippet we used the full variable declaration to be explicit.
The variable `slice` has type `[]byte`, pronounced "slice of bytes",
and is initialized from the array, called
`buffer`, by slicing elements 100 (inclusive) through 150 (exclusive).

在这个代码段中,我们使用了完整的变量声明. 变量`slice`拥有类型`[]byte`,意为bytes的切片, 并从一个`buffer`数组里面切分第100个(包含)元素至第150个(不含)元素而来.

The more idiomatic syntax would drop the type, which is set by the initializing expression:
以下是更简洁的初始化语法, 在之前的声明上隐藏了声明类型

	var slice = buffer[100:150]

Inside a function we could use the short declaration form,

在函数内部,我们可以使用这种短声明的方式

	slice := buffer[100:150]

What exactly is this slice variable?

那么这个切片变量到底是什么?

It's not quite the full story, but for now think of a
slice as a little data structure with two elements: a length and a pointer to an element
of an array.

你可以把切片当作一个拥有两个元素的数据结构,一个元素是长度, 一个元素是指针,这个指针指向一个数组里的元素

You can think of it as being built like this behind the scenes:

	type sliceHeader struct {
		Length        int
		ZerothElement *byte
	}

	slice := sliceHeader{
		Length:        50,
		ZerothElement: &buffer[100],
	}

Of course, this is just an illustration.
Despite what this snippet says that `sliceHeader` struct is not visible
to the programmer, and the type
of the element pointer depends on the type of the elements,
but this gives the general idea of the mechanics.

当然，这只是一个例证。 尽管这段代码片段中的 `sliceHeader `结构对程序员是不可见的，并且元素指针的类型取决于元素的类型，但这这段代码展示出了这个机制设计理念。

So far we've used a slice operation on an array, but we can also slice a slice, like this:

到目前为止,我们都是对数组进行切片,我们也可以对切片进行切片,如下

	slice2 := slice[5:10]

Just as before, this operation creates a new slice, in this case with elements
5 through 9 (inclusive) of the original slice, which means elements
105 through 109 of the original array.

像前面这个操作,它创建了一个新的切片, 在这个例子中, 原始切片第5至第9之间的元素,同时也是之前原始数组第105至109之间的元素

The underlying `sliceHeader` struct for the `slice2` variable looks like
this:

变量`slice2`底层数据结构`sliceHeader`长这样:

	slice2 := sliceHeader{
		Length:        5,
		ZerothElement: &buffer[105],
	}

Notice that this header still points to the same underlying array, stored in
the `buffer` variable.

注意这个header仍然指向同一个存储在`buffer`变量里的底层数组

We can also _reslice_, which is to say slice a slice and store the result back in
the original slice structure. After

	slice = slice[5:10]

the `sliceHeader` structure for the `slice` variable looks just like it did for the `slice2`
variable.
You'll see reslicing used often, for example to truncate a slice. This statement drops
the first and last elements of our slice:

	slice = slice[1:len(slice)-1]

[Exercise: Write out what the `sliceHeader` struct looks like after this assignment.]

You'll often hear experienced Go programmers talk about the "slice header"
because that really is what's stored in a slice variable.
For instance, when you call a function that takes a slice as an argument, such as
[bytes.IndexRune](/pkg/bytes/#IndexRune), that header is
what gets passed to the function.
In this call,

	slashPos := bytes.IndexRune(slice, '/')

the `slice` argument that is passed to the `IndexRune` function is, in fact,
a "slice header".

你经常会听到有经验的Go程序员谈论"slice header",因为这实际上是存储在切片变量中.
例如，当你调用切片为入参的函数时，例如bytes.IndexRune，slice header是传递给函数的。
在 `slashPos := bytes.IndexRune(slice, '/')`调用中,实际传入`IndexRune`函数的是"slice header"

There's one more data item in the slice header, which we talk about below,
but first let's see what the existence of the slice header means when you
program with slices.

slice header 中还有一个元素,我们将在后面讨论,在此之前我们应先明白slice header在编程中意味着什么
## Passing slices to functions
将切片传入函数

It's important to understand that even though a slice contains a pointer,
it is itself a value.
Under the covers, it is a struct value holding a pointer and a length.
It is _not_ a pointer to a struct.

你一定要知道即使切片包含指针它本身也是一个值.
在表象之下,slice是一个包含指针和长度的结构体,它不是指向一个结构体的指针

This matters.
这很重要

When we called `IndexRune` in the previous example,
it was passed a _copy_ of the slice header.
That behavior has important ramifications.

在上一个调用`IndexRune`示例中，他实际传入的是slice header的值拷贝
这种行为有着重要的影响.

Consider this simple function:

```go
func AddOneToEachElement(slice []byte) {
    for i := range slice {
        slice[i]++
    }
}
```

It does just what its name implies, iterating over the indices of a slice
(using a `for` `range` loop), incrementing its elements.
这个函数的功能如其函数命名,遍历切片每一个元素,并为其+1
Try it:

```go
func main() {
    slice := buffer[10:20]
    for i := 0; i < len(slice); i++ {
        slice[i] = byte(i)
    }
    fmt.Println("before", slice)
    AddOneToEachElement(slice)
    fmt.Println("after", slice)
}
```

(You can edit and re-execute these runnable snippets if you want to explore.)

Even though the slice _header_ is passed by value, the header includes
a pointer to elements of an array, so both the original slice header
and the copy of the header passed to the function describe the same
array.

尽管Slice header是值传递，但header包含一个指向数组元素的指针，
因此传递给函数的原始slice header和header拷贝都描述相同的数组。

Therefore, when the function returns, the modified elements can
be seen through the original slice variable.

因此，当函数返回时，可以通过原始slice变量查看修改后的元素。

The argument to the function really is a copy, as this example shows:

```go
func SubtractOneFromLength(slice []byte) []byte {
    slice = slice[0 : len(slice)-1]
    return slice
}

func main() {
    fmt.Println("Before: len(slice) =", len(slice))
    newSlice := SubtractOneFromLength(slice)
    fmt.Println("After:  len(slice) =", len(slice))
    fmt.Println("After:  len(newSlice) =", len(newSlice))
}
```

Here we see that the _contents_ of a slice argument can be modified by a function,
but its _header_ cannot.
The length stored in the `slice` variable is not modified by the call to the function,
since the function is passed a copy of the slice header, not the original.
Thus if we want to write a function that modifies the header, we must return it as a result
parameter, just as we have done here.
The `slice` variable is unchanged but the returned value has the new length,
which is then stored in `newSlice`,

这里我们看到，slice参数的内容可以由函数修改，但其标头不能。
存储在`slice`变量里的长度,是不能通过调用函数来修改的,
只要函数传入一个slice header,函数的slice就不再是原始slice
因此，要是我们想写一个可以修改header的函数,我们必须将其作为结果参数返回，就像我们在这里所做的那样。
之前的`slice`变量并未被改动，但返回的`newSlice`具有新的长度
## Pointers to slices: Method receivers 切片的指针：方法接收器

Another way to have a function modify the slice header is to pass a pointer to it.

让函数修改切片标头的另一种方法是向其传递指针。

Here's a variant of our previous example that does this:

下面是我们前面示例的一个变体，它可以做到这一点：

```Go
func PtrSubtractOneFromLength(slicePtr *[]byte) {
    slice := *slicePtr
    *slicePtr = slice[0 : len(slice)-1]
}

func main() {
    fmt.Println("Before: len(slice) =", len(slice))
    PtrSubtractOneFromLength(&slice)
    fmt.Println("After:  len(slice) =", len(slice))
}
```

It seems clumsy in that example, especially dealing with the extra level of indirection(a temporary variable helps),
but there is one common case where you see pointers to slices.
It is idiomatic to use a pointer receiver for a method that modifies a slice.

在那个例子中，它看起来很笨拙，尤其是需要额外使用临时变量来完成，
但有一种常见的情况，你会看到指向切片的指针。
对于修改切片的方法，使用指针接收器是惯用做法。

Let's say we wanted to have a method on a slice that truncates it at the final slash.
We could write it like this:

假设我们有个切片的方法是在最后一个斜杠的地方截断,可以这样写:

```Go
type path []byte

func (p *path) TruncateAtFinalSlash() {
    i := bytes.LastIndex(*p, []byte("/"))
    if i >= 0 {
        *p = (*p)[0:i]
    }
}

func main() {
    pathName := path("/usr/bin/tso") // Conversion from string to path.
    pathName.TruncateAtFinalSlash()
    fmt.Printf("%s\n", pathName)
}
```

If you run this example you'll see that it works properly, updating the slice in the caller.

如果运行此示例，你可以看到调用这个方法后切片就被更新了。

[Exercise: Change the type of the receiver to be a value rather
than a pointer and run it again. Explain what happens.]

On the other hand, if we wanted to write a method for `path` that upper-cases
the ASCII letters in the path (parochially ignoring non-English names), the method could
be a value because the value receiver will still point to the same underlying array.

另一方面，如果我们想为`path`编写一个方法，使`path`中的ASCII字母大写（局部忽略非英语名称），
那么该方法可以是一个值，因为值接收器仍将指向相同的底层数组。

```Go
type path []byte

func (p path) ToUpper() {
    for i, b := range p {
        if 'a' <= b && b <= 'z' {
            p[i] = b + 'A' - 'a'
        }
    }
}

func main() {
    pathName := path("/usr/bin/tso")
    pathName.ToUpper()
    fmt.Printf("%s\n", pathName)
}
```

Here the `ToUpper` method uses two variables in the `for` `range` construct
to capture the index and slice element.
This form of loop avoids writing `p[i]` multiple times in the body.

这里，ToUpper方法使用`for` `range`构造中的两个变量来捕获索引和切片元素。

[Exercise: Convert the `ToUpper` method to use a pointer receiver and see if its behavior changes.]

[Advanced exercise: Convert the `ToUpper` method to handle Unicode letters, not just ASCII.]

## Capacity 容量

Look at the following function that extends its argument slice of `ints` by one element:

查看以下函数，该函数将`ints`的参数片扩展了一个元素：

```Go
func Extend(slice []int, element int) []int {
    n := len(slice)
    slice = slice[0 : n+1]
    slice[n] = element
    return slice
}
```

(Why does it need to return the modified slice?) Now run it:

```go
func main() {
    var iBuffer [10]int
    slice := iBuffer[0:0]
    for i := 0; i < 20; i++ {
        slice = Extend(slice, i)
        fmt.Println(slice)
    }
}
```

See how the slice grows until... it doesn't.
查看切片如何增长，直到。。。 它不能运行

It's time to talk about the third component of the slice header: its _capacity_.
Besides the array pointer and length, the slice header also stores its capacity:

现在我们来谈谈`slice header`的第三个组成部分：它的容量。
除了数组指针和长度之外，slice header还存储其容量：

	type sliceHeader struct {
		Length        int
		Capacity      int
		ZerothElement *byte
	}

The `Capacity` field records how much space the underlying array actually has; it is the maximum
value the `Length` can reach.
Trying to grow the slice beyond its capacity will step beyond the limits of the array and will trigger a panic.

字段`Capacity` 记录了它底层数组实际有多少内存空间,这是长度可以达到的最大值。
试图将切片扩展到超出array的容量限制，则会导致panic。

通过这个例子创建一个切片后,

	slice := iBuffer[0:0]

它的hear是这样的:

	slice := sliceHeader{
		Length:        0,
		Capacity:      10,
		ZerothElement: &iBuffer[0],
	}

The `Capacity` field is equal to the length of the underlying array,
minus the index in the array of the first element of the slice (zero in this case).
If you want to inquire what the capacity is for a slice, use the built-in function `cap`:

	if cap(slice) == len(slice) {
		fmt.Println("slice is full!")
	}

容量字段等于底层数组的长度，减去切片第一个元素的数组中的索引（在本例中为零）。
如果要查询片的容量，请使用内置函数`cap`：

## Make

What if we want to grow the slice beyond its capacity?
You can't!
By definition, the capacity is the limit to growth.
But you can achieve an equivalent result by allocating a new array, copying the data over, and modifying the slice to describe the new array.

如果我们想将该片扩大到超出其容量，该怎么办？你不能！
根据定义，容量是增长的极限。
但是，你可以通过分配一个新数组、复制数据以及修改切片来描述新数组来实现相同的结果。

Let's start with allocation.
We could use the `new` built-in function to allocate a bigger array
and then slice the result,
but it is simpler to use the `make` built-in function instead.
It allocates a new array and
creates a slice header to describe it, all at once.
The `make` function takes three arguments: the type of the slice, its initial length, and its capacity, which is the
length of the array that `make` allocates to hold the slice data.
This call creates a slice of length 10 with room for 5 more (15-10), as you can see by running it:

先从分配开始,我们可以使用`new`的内置函数来分配到更大的数组，然后对结果进行切片，但使用make内置函数更简单。
它一次分配一个新数组并创建一个`slice header`来描述它。
make函数有三个参数：切片的类型、初始长度和容量，这是make函数分配用来保存切片数据的数组的长度。
这个命令创建了一个长度为10的切片，可以再容纳5个（15-10），运行它可以看到：

```Go
    slice := make([]int, 10, 15)
    fmt.Printf("len: %d, cap: %d\n", len(slice), cap(slice))
	```

This snippet doubles the capacity of our `int` slice but keeps its length the same:

此片段将int切片的容量加倍，但保持其长度不变：

```Go
    slice := make([]int, 10, 15)
    fmt.Printf("len: %d, cap: %d\n", len(slice), cap(slice))
    newSlice := make([]int, len(slice), 2*cap(slice))
    for i := range slice {
        newSlice[i] = slice[i]
    }
    slice = newSlice
    fmt.Printf("len: %d, cap: %d\n", len(slice), cap(slice))
	```

After running this code the slice has much more room to grow before needing another reallocation.

运行此代码后，切片在需要再次重新分配之前有更大的增长空间。

When creating slices, it's often true that the length and capacity will be same.
The `make` built-in has a shorthand for this common case.
The length argument defaults to the capacity, so you can leave it out
to set them both to the same value.
After

	gophers := make([]Gopher, 10)

the `gophers` slice has both its length and capacity set to 10.

创建切片时，长度和容量通常是相同的。
`make`内置函数有此常见情况的简洁使用方式.
长度参数默认为容量，因此可以省去它，将两者设置为相同的值。

在`gophers := make([]Gopher, 10)`之后, 切片变量`gophers`的长度和容量就都被设为10了.

## Copy

When we doubled the capacity of our slice in the previous section,
we wrote a loop to copy the old data to the new slice.
Go has a built-in function, `copy`, to make this easier.
Its arguments are two slices, and it copies the data from the right-hand argument to the left-hand argument.
Here's our example rewritten to use `copy`:

```Go
    newSlice := make([]int, len(slice), 2*cap(slice))
    copy(newSlice, slice)
```

The `copy` function is smart.
It only copies what it can, paying attention to the lengths of both arguments.
In other words, the number of elements it copies is the minimum of the lengths of the two slices.
This can save a little bookkeeping.
Also, `copy` returns an integer value, the number of elements it copied, although it's not always worth checking.

The `copy` function also gets things right when source and destination overlap, which means it can be used to shift
items around in a single slice.
Here's how to use `copy` to insert a value into the middle of a slice.

```Go
// Insert inserts the value into the slice at the specified index,
// which must be in range.
// The slice must have room for the new element.
func Insert(slice []int, index, value int) []int {
    // Grow the slice by one element.
    slice = slice[0 : len(slice)+1]
    // Use copy to move the upper part of the slice out of the way and open a hole.
    copy(slice[index+1:], slice[index:])
    // Store the new value.
    slice[index] = value
    // Return the result.
    return slice
}
```

There are a couple of things to notice in this function.
First, of course, it must return the updated slice because its length has changed.
Second, it uses a convenient shorthand.
The expression

	slice[i:]

means exactly the same as

	slice[i:len(slice)]

Also, although we haven't used the trick yet, we can leave out the first element of a slice expression too;
it defaults to zero. Thus

	slice[:]

just means the slice itself, which is useful when slicing an array.
This expression is the shortest way to say "a slice describing all the elements of the array":

	array[:]

Now that's out of the way, let's run our `Insert` function.

```Go
    slice := make([]int, 10, 20) // Note capacity > length: room to add element.
    for i := range slice {
        slice[i] = i
    }
    fmt.Println(slice)
    slice = Insert(slice, 5, 99)
    fmt.Println(slice)
```

## Append: An example

A few sections back, we wrote an `Extend` function that extends a slice by one element.
It was buggy, though, because if the slice's capacity was too small, the function would
crash.
(Our `Insert` example has the same problem.)
Now we have the pieces in place to fix that, so let's write a robust implementation of
`Extend` for integer slices.

```Go
func Extend(slice []int, element int) []int {
    n := len(slice)
    if n == cap(slice) {
        // Slice is full; must grow.
        // We double its size and add 1, so if the size is zero we still grow.
        newSlice := make([]int, len(slice), 2*len(slice)+1)
        copy(newSlice, slice)
        slice = newSlice
    }
    slice = slice[0 : n+1]
    slice[n] = element
    return slice
}
```

In this case it's especially important to return the slice, since when it reallocates
the resulting slice describes a completely different array.
Here's a little snippet to demonstrate what happens as the slice fills up:

```Go
    slice := make([]int, 0, 5)
    for i := 0; i < 10; i++ {
        slice = Extend(slice, i)
        fmt.Printf("len=%d cap=%d slice=%v\n", len(slice), cap(slice), slice)
        fmt.Println("address of 0th element:", &slice[0])
    }
	```

Notice the reallocation when the initial array of size 5 is filled up.
Both the capacity and the address of the zeroth element change when the new array is allocated.

With the robust `Extend` function as a guide we can write an even nicer function that lets
us extend the slice by multiple elements.
To do this, we use Go's ability to turn a list of function arguments into a slice when the
function is called.
That is, we use Go's variadic function facility.

Let's call the function `Append`.
For the first version, we can just call `Extend` repeatedly so the mechanism of the variadic function is clear.
The signature of `Append` is this:

	func Append(slice []int, items ...int) []int

What that says is that `Append` takes one argument, a slice, followed by zero or more
`int` arguments.
Those arguments are exactly a slice of `int` as far as the implementation
of `Append` is concerned, as you can see:

```Go
// Append appends the items to the slice.
// First version: just loop calling Extend.
func Append(slice []int, items ...int) []int {
    for _, item := range items {
        slice = Extend(slice, item)
    }
    return slice
}
```

Notice the `for` `range` loop iterating over the elements of the `items` argument, which has implied type `[]int`.
Also notice the use of the blank identifier `_` to discard the index in the loop, which we don't need in this case.

Try it:

```Go
    slice := []int{0, 1, 2, 3, 4}
    fmt.Println(slice)
    slice = Append(slice, 5, 6, 7, 8)
    fmt.Println(slice)

```

Another new technique in this example is that we initialize the slice by writing a composite literal,
which consists of the type of the slice followed by its elements in braces:

	    slice := []int{0, 1, 2, 3, 4}

The `Append` function is interesting for another reason.
Not only can we append elements, we can append a whole second slice
by "exploding" the slice into arguments using the `...` notation at the call site:

```Go
    slice1 := []int{0, 1, 2, 3, 4}
    slice2 := []int{55, 66, 77}
    fmt.Println(slice1)
    slice1 = Append(slice1, slice2...) // The '...' is essential!
    fmt.Println(slice1)
	```

Of course, we can make `Append` more efficient by allocating no more than once,
building on the innards of `Extend`:

```Go
// Append appends the elements to the slice.
// Efficient version.
func Append(slice []int, elements ...int) []int {
    n := len(slice)
    total := len(slice) + len(elements)
    if total > cap(slice) {
        // Reallocate. Grow to 1.5 times the new size, so we can still grow.
        newSize := total*3/2 + 1
        newSlice := make([]int, total, newSize)
        copy(newSlice, slice)
        slice = newSlice
    }
    slice = slice[:total]
    copy(slice[n:], elements)
    return slice
}

```

Here, notice how we use `copy` twice, once to move the slice data to the newly
allocated memory, and then to copy the appending items to the end of the old data.

Try it; the behavior is the same as before:

```Go
    slice1 := []int{0, 1, 2, 3, 4}
    slice2 := []int{55, 66, 77}
    fmt.Println(slice1)
    slice1 = Append(slice1, slice2...) // The '...' is essential!
    fmt.Println(slice1)
	```

## Append: The built-in function

And so we arrive at the motivation for the design of the `append` built-in function.
It does exactly what our `Append` example does, with equivalent efficiency, but it
works for any slice type.

A weakness of Go is that any generic-type operations must be provided by the
run-time. Some day that may change, but for now, to make working with slices
easier, Go provides a built-in generic `append` function.
It works the same as our `int` slice version, but for _any_ slice type.

Remember, since the slice header is always updated by a call to `append`, you need
to save the returned slice after the call.
In fact, the compiler won't let you call append without saving the result.

Here are some one-liners intermingled with print statements. Try them, edit them and explore:

```Go
    // Create a couple of starter slices.
    slice := []int{1, 2, 3}
    slice2 := []int{55, 66, 77}
    fmt.Println("Start slice: ", slice)
    fmt.Println("Start slice2:", slice2)

    // Add an item to a slice.
    slice = append(slice, 4)
    fmt.Println("Add one item:", slice)

    // Add one slice to another.
    slice = append(slice, slice2...)
    fmt.Println("Add one slice:", slice)

    // Make a copy of a slice (of int).
    slice3 := append([]int(nil), slice...)
    fmt.Println("Copy a slice:", slice3)

    // Copy a slice to the end of itself.
    fmt.Println("Before append to self:", slice)
    slice = append(slice, slice...)
    fmt.Println("After append to self:", slice)
	```

It's worth taking a moment to think about the final one-liner of that example in detail to understand
how the design of slices makes it possible for this simple call to work correctly.

There are lots more examples of `append`, `copy`, and other ways to use slices
on the community-built
[["Slice Tricks" Wiki page](https://github.com/golang/go/wiki/SliceTricks)

## Nil

As an aside, with our newfound knowledge we can see what the representation of a `nil` slice is.
Naturally, it is the zero value of the slice header:

	sliceHeader{
		Length:        0,
		Capacity:      0,
		ZerothElement: nil,
	}

or just

	sliceHeader{}

The key detail is that the element pointer is `nil` too. The slice created by

	array[0:0]

has length zero (and maybe even capacity zero) but its pointer is not `nil`, so
it is not a nil slice.

As should be clear, an empty slice can grow (assuming it has non-zero capacity), but a `nil`
slice has no array to put values in and can never grow to hold even one element.

That said, a `nil` slice is functionally equivalent to a zero-length slice, even though it points
to nothing.
It has length zero and can be appended to, with allocation.
As an example, look at the one-liner above that copies a slice by appending
to a `nil` slice.

## Strings

Now a brief section about strings in Go in the context of slices.

Strings are actually very simple: they are just read-only slices of bytes with a bit
of extra syntactic support from the language.

Because they are read-only, there is no need for a capacity (you can't grow them),
but otherwise for most purposes you can treat them just like read-only slices
of bytes.

For starters, we can index them to access individual bytes:

	slash := "/usr/ken"[0] // yields the byte value '/'.

We can slice a string to grab a substring:

	usr := "/usr/ken"[0:4] // yields the string "/usr"

It should be obvious now what's going on behind the scenes when we slice a string.

We can also take a normal slice of bytes and create a string from it with the simple conversion:

	str := string(slice)

and go in the reverse direction as well:

	slice := []byte(usr)

The array underlying a string is hidden from view; there is no way to access its contents
except through the string. That means that when we do either of these conversions, a
copy of the array must be made.
Go takes care of this, of course, so you don't have to.
After either of these conversions, modifications to
the array underlying the byte slice don't affect the corresponding string.

An important consequence of this slice-like design for strings is that
creating a substring is very efficient.
All that needs to happen
is the creation of a two-word string header. Since the string is read-only, the original
string and the string resulting from the slice operation can share the same array safely.

A historical note: The earliest implementation of strings always allocated, but when slices
were added to the language, they provided a model for efficient string handling. Some of
the benchmarks saw huge speedups as a result.

There's much more to strings, of course, and a
[separate blog post](https://blog.golang.org/strings) covers them in greater depth.

## Conclusion

To understand how slices work, it helps to understand how they are implemented.
There is a little data structure, the slice header, that is the item associated with the slice
variable, and that header describes a section of a separately allocated array.
When we pass slice values around, the header gets copied but the array it points
to is always shared.

Once you appreciate how they work, slices become not only easy to use, but
powerful and expressive, especially with the help of the `copy` and `append`
built-in functions.

## More reading

There's lots to find around the intertubes about slices in Go.
As mentioned earlier,
the ["Slice Tricks" Wiki page](/wiki/SliceTricks)
has many examples.
The [Go Slices](https://blog.golang.org/go-slices-usage-and-internals) blog post
describes the memory layout details with clear diagrams.
Russ Cox's [Go Data Structures](https://research.swtch.com/godata) article includes
a discussion of slices along with some of Go's other internal data structures.

There is much more material available, but the best way to learn about slices is to use them.
