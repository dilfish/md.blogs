---
title: "Rust 杂项"
date: 2022-08-12T10:53:03+08:00
draft: false
---

### async, .await
```rust
async fn do_something() { /* ... */ }
```
async 函数返回的值叫 Future，而 Future 必须由 executor 执行：
```rust
use futures::executor::block_on;
let future = hello_world(); // Nothing is printed
block_on(future); // `future` is run and "hello, world!" is printed
```
```rust
async fn learn_and_sing() {
    // Wait until the song has been learned before singing it.
    // We use `.await` here rather than `block_on` to prevent blocking the
    // thread, which makes it possible to `dance` at the same time.
    let song = learn_song().await;
    sing_song(song).await;
}
```
上面的代码在调度过程中，learn_song 函数异步执行，但是执行完成之后才会执行 sing_song，而 sing_song 执行完之后函数 learn_and_sing 才会返回。

sing_song 的 await 也是必要的，不然 sing_song 不会执行。

### phantomdata
phantomdata 解释最好的文章是这个：
[why phantomdata](http://troubles.md/why-phantomdata/)
本文也解释了 subtyping 和 variance 的内容。
![subtypoing](/pics/variance.png)
当然还有官方的文档：
[phantom data](https://doc.rust-lang.org/nomicon/phantom-data.html)
[sub typing](https://doc.rust-lang.org/nomicon/subtyping.html)

phantomdata 例如：
```rust
use std::marker;

struct Iter<'a, T: 'a> {
    ptr: *const T,
    end: *const T,
    // _marker: marker::PhantomData<&'a T>,
}
```
对于 [T] 的 iter 实现可能像上面这样，raw pointer 没有 lifetime，则可以添加一个 phantom data 来实现 lifetime 约束。

```rust
use std::marker;

struct Vec<T> {
    data: *const T, // *const for variance!
    len: usize,
    cap: usize,
    // _marker: marker::PhantomData<T>,
}
```
因为 const raw pointer 不能修改内容，所以 drop checker 认为 Vec 不拥有数据，也可以通过 phantom data 来指示，销毁 Vec 的同时，销毁 T。

![如图](/pics/phantomdata.png)

### infallable
Infallable 的用处比较简单，就是在 trait 或者参数需求的时候的一个占位符。

### dyn
https://doc.rust-lang.org/rust-by-example/trait/dyn.html

### ?
? 叫做 error propagation，相当于简化了 go 语言里面的 if err != nil 这个表达式。

### Fn, FnMut, FnOnce
这三个 trait 实现之后，对象就可以进行调用(callable)。
