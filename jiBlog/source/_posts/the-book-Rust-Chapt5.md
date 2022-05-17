---
title: the book(Rust) - Chapt5
date: 2022-02-07 22:26:03
tags: Rust
categories: ProgLang
---
<style>
img{
    width: 80%;
}
</style>

<!--more-->

# 结构体

结构体不可以指定某一个**字段**是否可变，但是可以让整一个实例可变

```Rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}

let mut user1 = User {
    email: String::from("someone@example.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
}
```

## 字段初始化
使用和参数一样的名字的变量名，可以不写`email: email`
```Rust
fn build_user(email: String, username: String) -> User {
    User {
        email,
        username,
        active: true,
        sign_in_count: 1,
    }
}

fn main() {
    let user1 = build_user(
        String::from("someone@example.com"),
        String::from("someusername123"),
    );
}
```

上下两段代码的效果是一样的，
`..user1`证明别的变量都是一样的，但是**必须放在最后。

```Rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}

fn main() {
    // --snip--

    let user1 = User {
        email: String::from("someone@example.com"),
        username: String::from("someusername123"),
        active: true,
        sign_in_count: 1,
    };

    let user2 = User {
        email: String::from("another@example.com"),
        ..user1
    };
}
```

由于我们的user2使用了user1的username，user1**不再有效**，因为在堆上。

但是如果仅仅使用了user1的`active`与`sign_in_count`是没关系的，他们有`copy trait`，在栈上。

我们也可以使用**没有命名字段**的结构体

```Rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

fn main() {
    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);
}
```

可以用`black.0, black.1, black.2`来访问

**如果要在结构体内使用别的变量的引用**，需要用**生命周期**，保证结构体引用的数据有效性与结构体本身一致。



`需要注意`

* **把结构体中具有所有权的字段转移出去后，将无法再访问该字段，但是可以正常访问其它的字段**。

---

## println! 宏打印

`println!("rect1 is {}", rect1);`需要我们实现`std::fmt::Display`

`println!("rect1 is {:?}", rect1);`需要我们实现`Debug trait`
* 我们可以在结构体上面加上`#[derive(Debug)]`让编译器自动生成

```Rust
#[derive(Debug)] 
struct Rectangle {
    width: u32,
    height: u32,
}
```

## dbg!宏

这是另外一种`Debug`格式打印数值的方法就是`dbg!`
* 他会接受一个表达式的所有权，打印出代码中调用它的文件与行号

```Rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let scale = 2;
    let rect1 = Rectangle {
        width: dbg!(30 * scale),
        height: 50,
    };

    dbg!(&rect1);
}
```
我们可以把 `dbg!` 放在表达式 `30 * scale` 周围，因为 `dbg!` **返回表达式的值的所有权**，所以 `width` 字段将获得相同的值，就像我们在那里没有 `dbg!` 调用一样。我们不希望 `dbg!` 拥有 `rect1` 的所有权，所以我们在下一次调用 `dbg!` 时传递一个引用。下面是这个例子的输出结果：

```
$ cargo run
   Compiling rectangles v0.1.0 (file:///projects/rectangles)
    Finished dev [unoptimized + debuginfo] target(s) in 0.61s
     Running `target/debug/rectangles`
[src/main.rs:10] 30 * scale = 60
[src/main.rs:14] &rect1 = Rectangle {
    width: 60,
    height: 50,
}
```

除了`Debug`, derive关键字还有很多可以使用的trait

---

## method syntax (impl block)

让我们给Rectangle定义一个area方法，作为成员函数
重点是下面的`impl块`

```Rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!(
        "The area of the rectangle is {} square pixels.",
        rect1.area()
    );
}
```

---

## 关联函数

关联函数就是参数里面**没有self**的函数，可以用于定义**constructor**

* 没有self作为参数，不能用`struct.method()`的方法来调用

```rust
impl Rect {
	fn new(w: u32, h: u32) -> Rect {
    Rect { width: w, height: h }
  }
}
```

---



