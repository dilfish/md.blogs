---
title: "Rust 的 lifetime"
date: 2021-12-21T16:11:45+08:00
draft: false
---

本文是阅读[Validating References with Lifetimes](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html)  的笔记。

```
Lifetime annotations don’t change how long any of the references live. Rather, they describe the relationships of the lifetimes of multiple references to each other without affecting the lifetimes. Just as functions can accept any type when the signature specifies a generic type parameter, functions can accept references with any lifetime by specifying a generic lifetime parameter.
```

lifetime 这个特性对于 c/c++ 系的程序员是比较好接受的，归根结底就是一句话，指针指向的对象不能提前释放，以免指针解引用失败。

对于 rust 而言，一般的对象所谓"释放"就是生命周期结束。rust 的 lifetime 分为两类，一类是函数返回值，一类是 struct。

函数的例子如下：
```
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

返回值引用的对象可能是参数的二者之一，因此，返回值的生命周期必须小于等于两个参数中小的那个。如果大于，那么当参数释放之后，返回值引用的内存就非法了。因此必须是返回值先释放。rust 这块的标记方法和所谓的 rules 可以说是一塌糊涂。不说也罢。

对于结构体来说也是类似的，如果结构体里面有成员引用了其他内存，那引用的内存的生命周期必须大于结构体本身的生命周期，不然引用内存先释放，结构体成员引用的就是非法内存了。

```
struct ImportantExcerpt<'a> {
    part: &'a str,
}
```

再次吐槽下这个标记，真二啊。
