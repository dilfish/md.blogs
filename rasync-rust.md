---
title: "异步rust"
date: 2023-02-10T20:35:29+08:00
draft: false
---

# Future
```rust
pub enum Poll<T> {
    Ready(T),
    Pending,
}

pub trait Future {
    type Output;
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

Poll 是一个 enum，他要么返回一个 Pending，要么返回一个 Ready(T)，而 Future 中的 Output 就是这里的 T 类型。

如果返回 Pending，表示任务没有完成，如果返回 Ready(T)，表示任务结束。

如果一个函数返回了 Future，则这个函数是异步的，可以被异步调用。

但是直接使用 Future 类型无法调用 fut:Future, fut.poll，而必须使用 Pin<&mut Fut>

## Pin
Pin 是为了解决自指问题，例如：
```c
struct item {
    int data;
    int *ptr;
};
struct item item;
item.ptr = &item.data;
```
当 item 在内存中移动时，item.ptr 就指向了非预期的内存。

不存在自指情况的类型都自动实现了一个 trait 叫 Unpin。而像上述的类型叫做!Unpin，如果想要安全地使用这些类型，则必须要使用 Pin<T>.

访问 Pin<T> 类型使用的方法叫 projection （投射）：
```rust
// Putting data into Pin
pub        fn new          <P: Deref<Target:Unpin>>(pointer: P) -> Pin<P>;
pub unsafe fn new_unchecked<P>                     (pointer: P) -> Pin<P>;

// Getting data from Pin
pub        fn into_inner          <P: Deref<Target: Unpin>>(pin: Pin<P>) -> P;
pub unsafe fn into_inner_unchecked<P>                      (pin: Pin<P>) -> P;
```

## 不能阻塞执行器(executor)
```rust
use std::time::Duration;
use tokio::time::sleep;

#[tokio::main(flavor = "current_thread")]
async fn main() {
    let one = tokio::spawn(greet());
    let two = tokio::spawn(greet());
    let (_, _) = tokio::join!(one, two);
}

async fn greet() {
    println!("Hello!");
    // await 执行结束之后打印
    // await 让程序异步 sleep 而不阻塞
    sleep(Duration::from_millis(500)).await;
    println!("Goodbye!");
}
```

## Future 通知被 Poll
```rust
impl Future for MyFuture {
    type Output = ();

    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output> {
        println!("MyFuture::poll()");
        // 通知再次被调用
        cx.waker().wake_by_ref();
        Poll::Pending
    }
}
```
ref:https://blog.cloudflare.com/pin-and-unpin-in-rust/
ref:https://fasterthanli.me/articles/pin-and-suffering
