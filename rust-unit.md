---
title: "Rust 的 Unit 类型"
date: 2022-08-10T15:42:36+08:00
draft: false
---

### 起因
我在看 rust 的 [http](https://docs.rs/http/latest/http/) 库时，发现了这样的一个例子：
```rust
use http::{Request, Response};

fn response(req: Request<()>) -> http::Result<Response<()>> {
    match req.uri().path() {
        "/" => index(req),
        "/foo" => foo(req),
        "/bar" => bar(req),
        _ => not_found(req),
    }
}
```

其中比较引人注目的是 Request<()> 这个表达式。

### 文档
- https://www.justanotherdot.com/posts/the-many-uses-of-the-empty-tuple.html
- https://stackoverflow.com/questions/24842271/what-is-the-purpose-of-the-unit-type-in-rust
- https://doc.rust-lang.org/std/iter/trait.Iterator.html

### 含义
可以看到 () 名字叫做 unit type，而 () 既是他的类型，也是他的唯一值。用来表示没有值的意思。和 C 系的 void 或者 NULL 类似，但又不完全相同。

实例中的那个 Request 的泛型类型写 ()，则表示没有泛型类型，也就是最基础的 Request 实例，如果需要特别的定义，则需要写另外的类型。例如 body 里面写值，则需要将 Request 写为 Request<&str>。

### 其他

而 collect 的文档是：
collect() can also create instances of types that are not typical collections. For example, a String can be built from chars, and an iterator of Result<T, E> items can be collected into Result<Collection<T>, E>. See the examples below for more.

Because collect() is so general, it can cause problems with type inference. As such, collect() is one of the few times you’ll see the syntax affectionately known as the ‘turbofish’: ::<>. This helps the inference algorithm understand specifically which collection you’re trying to collect into.
