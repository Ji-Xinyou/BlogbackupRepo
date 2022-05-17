---
title: the book(Rust) - Chapt6
date: 2022-02-07 23:53:37
tags: Rust
categories: ProgLang
---

<div></div>

<!--more-->

# 枚举与模式匹配

[toc]

## 定义

```rust
enum IPAddrKind {
  V4, V6
}
```



### How to use it

```Rust
let four = IPAddrKind::V4;
let six = IPAddrKind::V6;
```



### 将枚举类型和数据类型关联

常见的写法

```Rust
struct IpAddr {
  kind: IPAddrKind,
  addr: String,
}

let home = IpAddr {
  kind: IpAddrKind::V4,
  addr: String::from("127.0.0.1"),
}
```

一个更**简洁**而且更好的写法

```Rust
enum IpAddr {
  V4(String),
  V6(String),
}

let home = IpAddr::V4(String::from("127.0.0.1"));
```

事实上，我们现在的`IpAddr::V4(String)`是一个`函数调用`

它以一个String为输入，返回一个IpAddr的类型



用枚举类型代替结构体还有一个好处

* 不同的枚举类型的成员可以**处理不同类型与数量的数据**

```Rust
fn main() {
    enum IpAddr {
        V4(u8, u8, u8, u8),
        V6(String),
    }

    let home = IpAddr::V4(127, 0, 0, 1);

    let loopback = IpAddr::V6(String::from("::1"));
}
```

这里IPV4变成了4个u8类型的数据，但是IPV6仍然是一个字符串



### 枚举类型十分的灵活

```Rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
```

不同的成员可以有不同的关联的数据，成员的成员也可以有自己的名字！



### 枚举类型也可以定义函数

```Rust
impl Message {
  fn call(&self) {
    // nothing to do
  }
}

let m = Message::Quit()
m.call()
```



## Option 枚举 (标准库)

Rust并没有空值**null**，因为如果像非空值一样使用空值，会出现各种形式的错误。但是空值的概念是很有意义的，因此

**Rust编码了存在于不存在概念的一个枚举** - `Option<T>`

```Rust
enum Option<T> {
  None,
  Some<T>,
}
```

### Option 枚举的使用

```Rust
fn main() {
    let some_number = Some(5); // Option<i32>
    let some_string = Some("a string"); // Option<String>
  
	  // Rust does not know the **type** of None
    let absent_number: Option<i32> = None; 
}
```

请注意，在使用`None`的时候，我们**需要给定类型**，因为编译器需要知道`Option<T>`的类型是什么



Option的存在让我们知道，**只要一个值不是`Option<T>`类型，那么他一定不是空值**

但是当我们对`Option<T>`进行计算的时候，我们也需要将他的**值取出来**，而不能直接对他进行使用。

```Rust
Option<T>::is_some()
Option<T>::is_none()
Option<T>::unwrap()
```

* 如果你对C++17有所了解，这跟`std::optional<T>`很相似，而空值其实就是`nullopt_t`类型的变量。

---

## match 控制流

```rust
match VALUE {
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
    _ => EXPRESSION,
}
```



* 一个简单的例子

```Rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => {
            println!("Lucky penny!");
            1
        }
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```



### 绑定值的模式

* 我们先来看一段代码，重点关注`Coin::Quarter(state)`的内容

```Rust
#[derive(Debug)]
enum UsState {
    Alabama,
    Alaska,
    // --snip--
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {:?}!", state);
            25
        }
    }
}
```

我们调用

```rust
value_in_cents(Coin::Quarter(UsState::Alaska))
```

`coin`将匹配`Coin::Quarter(state)`，然后变量`state`就会绑定为`UsState::Alaska`

---

### match guard

```rust
let num = Some(4);

match num {
    Some(x) if x < 5 => println!("less than five: {}", x),
    Some(x) => println!("{}", x),
    None => (),
}
```

会匹配第一个分支，打印`less than five: 4`

---

### 匹配Option\<T\>

下面这段代码对于`Option<i32>`中有值的变量，将对他们的值加一。而None则不变。

```rust
fn main() {
    fn plus_one(x: Option<i32>) -> Option<i32> {
        match x {
            None => None,
            Some(i) => Some(i + 1),
        }
    }

    let five = Some(5);
    let six = plus_one(five);
    let none = plus_one(None);
}
```



### Match is exhaustive

对于match的表达式来说，**所有的可能情况都应该被包括**

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        Some(i) => Some(i + 1),
    }
}
```

在上面的代码中，`None`没有被匹配，编译会报错！



### 通配模式和  _ 占位符

#### 通配模式

```rust
fn main() {
    let dice_roll = 9;
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        other => move_player(other),
    }

    fn add_fancy_hat() {}
    fn remove_fancy_hat() {}
    fn move_player(num_spaces: u8) {}
}
```

`other`匹配了所有除了3和7以外的u8类型的数，叫做`通配分支`，**必须放在最后**



#### _ 占位符

```rust
fn main() {
    let dice_roll = 9;
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        _ => reroll(),
    }

    fn add_fancy_hat() {}
    fn remove_fancy_hat() {}
    fn reroll() {}
}
```

**_占位符**告诉编译器，我们不会使用这个值，**因此RUST不会警告我们有unused variable**



---

## if let 简单控制流

```rust
if let PATTERN = VALUE {
  
}
```



`if let`语句是一种**更短的**代替`match`关键字的方法

但是我们牺牲了穷尽性检查，先看看下面两段代码吧！

```rust
fn main() {
    let config_max = Some(3u8);
    match config_max {
        Some(max) => println!("The maximum is configured to be {}", max),
        _ => (),
    }
}
```

上面这段代码使用的是match关键字，我们根本不care在None的情况下会发生什么，但是我们仍然要因为穷尽性检查而写下`_ => ()`这行语句



我们可以用`if let`来代替它，它可以看做是`match`的一个语法糖



`if let`后接了一个`模式` + `表达式`，用表达式来匹配这个模式之后，再运行里面的代码。

比如下面我们用`Some(3u8)`匹配了`Some(max)`这个模式，max就变成了3u8



**它当值匹配某一模式时，执行代码并忽略所有其他值**

```rust
fn main() {
    let config_max = Some(3u8);
    if let Some(max) = config_max {
        println!("The maximum is configured to be {}", max);
    }
}
```

或者我们可以用`if let`+ `else`

```rust
#[derive(Debug)]
enum UsState {
    Alabama,
    Alaska,
    // --snip--
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}

fn main() {
    let coin = Coin::Penny;
    let mut count = 0;
    if let Coin::Quarter(state) = coin {
        println!("State quarter from {:?}!", state);
    } else {
        count += 1;
    }
}
```

---

## matches! Macro

```rust
enum MyEnum {
  Foo,
  Bar,
}

fn main() {
  let v = vec![MyEnum::Foo, MyEnum::Bar, MyEnum::Foo];
}
```

过滤，只留下`MyEnum::Foo`类型的entry



* 不可以这么写`v.iter().filter(|x| x == MyEnum::Foo);`
  * 因为不可以将x与枚举成员比较



`matches!()`: 将**表达式**与**模式**匹配，返回true or false



可以用

```rust
v.iter().filter(|x| matches!(x, MyEnum::Foo));
```

---

## while let

```rust
// Vec是动态数组
let mut stack = Vec::new();

// 向数组尾部插入元素
stack.push(1);
stack.push(2);
stack.push(3);

// stack.pop从数组尾部弹出元素
while let Some(top) = stack.pop() {
    println!("{}", top);
```

* Or `loop + if let`

---

## 变量覆盖

```rust
fn main() {
    let x = Some(5);
    let y = 10;

    match x {
        Some(50) => println!("Got 50"),
        Some(y) => println!("Matched, y = {:?}", y),
        _ => println!("Default case, x = {:?}", x),
    }

    println!("at the end: x = {:?}, y = {:?}", x, y);
}
```

需要注意的是，在match块中，`y=5`，但是离开match块后，这个y的作用域就结束了，因此最后的输出会是

```text
Matched, y = 5
at the end: x = Some(5), y = 10
```



### 用match guard解决变量覆盖

```rust
fn main() {
    let x = Some(5);
    let y = 10;

    match x {
        Some(50) => println!("Got 50"),
        Some(n) if n == y => println!("Matched, n = {}", n),
        _ => println!("Default case, x = {:?}", x),
    }

    println!("at the end: x = {:?}, y = {}", x, y);
}
```

---

## Single branch, multiple pattern

```rust
let x = 1;
let y = true;

match x {
    1 | 2 if y => println!("one or two"),
    3 => println!("three"),
    _ => println!("anything"),
}
```

### 通过序列..

```rust
let x = 5;

match x {
    1..=5 => println!("one through five"),
    _ => println!("something else"),
}
```

---

## match struct

可以注意一下match的第三个分支

```rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7 };

    match p {
        Point { x, y: 0 } => println!("On the x axis at {}", x),
        Point { x: 0, y } => println!("On the y axis at {}", y),
        Point { x, y } => println!("On neither axis: ({}, {})", x, y),
    }
}
```

也可以结构**枚举类型的不同成员**

```rust
match msg {
        Message::ChangeColor(Color::Rgb(r, g, b)) => {
            println!(
                "Change the color to red {}, green {}, and blue {}",
                r,
                g,
                b
            )
        }
        Message::ChangeColor(Color::Hsv(h, s, v)) => {
            println!(
                "Change the color to hue {}, saturation {}, and value {}",
                h,
                s,
                v
            )
        }
        _ => ()
    }
```

---

## 用..忽略剩余值

```rust
fn main() {
    let numbers = (2, 4, 8, 16, 32);

    match numbers {
        (first, .., last) => {
            println!("Some numbers: {}, {}", first, last);
        },
    }
}
```

---

## @ 绑定

如果需要绑定另外一个变量，可以用@绑定

`id: id_var @ 3..7`

**既想要限定分值范围，又想要使用分支变量**

```rust
enum Message {
    Hello { id: i32 },
}

let msg = Message::Hello { id: 5 };

match msg {
    Message::Hello { id: id_variable @ 3..=7 } => {
        println!("Found an id in range: {}", id_variable)
    },
    Message::Hello { id: 10..=12 } => {
        println!("Found an id in another range")
    },
    Message::Hello { id } => {
        println!("Found some other id: {}", id)
    },
}
```

### @前绑定后解构

可以绑定新变量的同时解构

```rust
fn main() {
    // 绑定新变量 `p`，同时对 `Point` 进行解构
    let p @ Point {x: px, y: py } = Point {x: 10, y: 23};
    println!("x: {}, y: {}", px, py);
    println!("{:?}", p);

    
    let point = Point {x: 10, y: 5};
    if let p @ Point {x: 10, y} = point {
        println!("x is 10 and y is {} in {:?}", y, p);
    } else {
        println!("x was not 10 :(");
    }
}
```

### @新特性

```rust
fn main() {
    match 1 {
        num @ (1 | 2) => {
            println!("{}", num);
        }
        _ => {}
    }
}
```

---

