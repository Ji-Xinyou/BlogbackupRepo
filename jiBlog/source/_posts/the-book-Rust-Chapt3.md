---
title: the book(Rust) - Chapt3
date: 2022-02-05 02:28:34
tags: Rust
categories: ProgLang
---
<style>
img{
    width: 80%;
}
</style>

<!--more-->

# Rust - 常见编程概念

这一系列的博客我决定使用中文来进行完成。

## 变量与可变性(mutability)

### 可变性
```Rust
fn main() {
    let **mut** x = 5;
    println!("x: {}", x);
    x = 6; // 如果没有mut关键字，编译器会报错
    println!("x: {}", x);
}
```

可变变量有`mut`关键字，可变变量才可以二次赋值。

### 常量

```Rust
const SEC_IN_ONE_HOUR: u32 = 60 * 1;
```

* **常量必须要有注解**
* **常量永远不可变**
* 常量在作用域内一直有效

### Shadowing

```Rust
fn main() {
    let x = 5;

    let x = x + 1;

    {
        let x = x * 2;
        println!("The value of x in the inner scope is: {}", x);
    }

    println!("The value of x is: {}", x);
}

输出为
The value of x in the inner scope is: 12
The value of x is: 6
```
因为Shadowing只在作用域内有效

如果要让shadowing有不同的数据类型
* 必须声明类型
* 必须为可变变量

---

## 标量类型

### 整形
| length | signed | unsigned |
| -- | -- | -- |
| 8b  | i8 | u8 |
| 16b  | i16 | u16 |
| 32b  | i32 | u32 |
| 64b  | i64 | u64 |
| 128b  | i128  | u128  |
| arch  | isize | usize |

| 数字字面值 | 例子 |
| --- | --- |
Decimal (十进制) | 98_222
Hex (十六进制) | 0xff
Octal (八进制) | 0o77
Binary (二进制) | 0b1111_0000
Byte (单字节字符)(仅限于u8) | b'A'

也可以用`let x = 57u8`作为类型后缀

#### 整形溢出

在`release`中，不检查，使用wrapping。 在`debug`中，不允许溢出，出现panic。

* 所有模式下都可以使用 `wrapping_*` 方法进行包装，如 wrapping_add
* 如果 `check_*` 方法出现溢出，则返回 None值
* 用 `overflowing_*` 方法返回值和一个布尔值，表示是否出现溢出
* 用 `saturating_*` 方法在值的最小值或最大值处进行饱和处理

### 浮点
`f64`, `f32`，默认为f32

### 字符类型
单引号是`char`，双引号是字符串。
* **char是4个字节**，代表的是Unicode标量值

---

## 复合类型

将多个值组合成一个类型，Rust原生有`tuple`和`array`

### tuple
tuple 中不同的值可以有不同的类型。

```Rust
fn main() {
    let tup = (500, 6.4, 1);

    let (x, y, z) = tup;

    println!("The value of y is {}", y);
    println!("The value of y is {}", tup.1);
}
```

在Rust中, `()`为特殊的tuple，只有一个值，也写成`()`，被称为unit type，值被称为unit value。
如果表达式不返回任何值，则隐式返回unit value.

### array
array 中，所有的值必须是同一类型，分配在stack上
在确定元素数量不会改变时使用

```Rust
fn main() {

    let months = ["January", "February", "March", "April", "May", "June", "July",
              "August", "September", "October", "November", "December"];

    let a: [i32; 5] = [1, 2, 3, 4, 5];

    let b = [3; 5];
}
```

```Rust
use std::io;

fn main() {
    let a = [1, 2, 3, 4, 5];

    println!("Please enter an array index.");

    let mut index = String::new();

    io::stdin()
        .read_line(&mut index)
        .expect("Failed to read line");

    let index: usize = index
        .trim()
        .parse()
        .expect("Index entered was not a number");

    let element = a[index];

    println!(
        "The value of the element at index {} is: {}",
        index, element
    );
}
```
如果输入了一个超过末尾的数字，比如10，编译器会panic，即遇到exception而退出。

---

## 函数

### 命名风格
使用snake case，即**所有字母小写**，**使用下滑线分隔单词**。

用关键字`fn`声明函数

### 参数
在Rust函数中，所有的参数都必须要有**类型注解**，Rust是静态类型语言！
```Rust
fn main() {
    print_labeled_measurement(5, 'h');
}

fn print_labeled_measurement(value: i32, unit_label: char) {
    println!("The measurement is: {}{}", value, unit_label);
}

```

### 语句与表达式
**语句（Statements）**是执行一些操作但不返回值的指令。
**表达式（Expressions）**计算并产生一个值。让我们看一些例子。

`let y = 6;`是一个语句。
语句不返回任何值，因此`let x = (let y = 6);`是错误的

那么什么是表达式呢？

函数调用是表达式, macro调用是一个表达式，大括号的作用于也是表达式。

```Rust
fn main() {
    let x = 5;

    let y = {
        let x = 3;
        x + 1
    };

    println!("The value of y is: {}", y);
}
```
在这里面，y的值是4。因为这个表达式的计算结果是`x + 1`。

### 返回值

在Rust中，我们不命名返回值，但是**必须声明类型**
在Rust中，函数的表达式为函数体**最后一个表达式的值**

```Rust
fn five() -> i32 {
    5
}

fn main() {
    let x = five();

    println!("The value of x is: {}", x);
}
```

**请注意，语句并不会返回值**
```Rust
fn main() {
    let x = plus_one(5);

    println!("The value of x is: {}", x);
}

fn plus_one(x: i32) -> i32 {
    x + 1;
}
```
这段代码中，会报错，因为`x + 1;`有分号，他是一个语句，因此没有返回值。
与函数签名相冲突了。

编译器会告诉你，这个函数实际返回`()`，即unit value。

---

## Control Flow

### if
if语句的条件**必须是bool**类型。
不一定需要小括号把条件括起来
`if number != 0 { ... }`也是可以的

如果有多个`else if`建议使用`match`分支结构

#### let if
```Rust
fn main() {
    let condition = true;
    let number = if condition {
        5
    } else {
        6
    };

    println!("The value of number is: {}", number);
}
```

需要注意的是，在`let if`中，`if`与`else`中表达式返回的**类型必须一致**，比如上面返回的都是`i32`整形，但是如果else中返回了"six"，就会报错。

### 循环

#### loop
相当于while(1)，用break退出

##### 循环标签
```Rust
fn main() {
    let mut count = 0;
    'counting_up: loop {
        println!("count = {}", count);
        let mut remaining = 10;

        loop {
            println!("remaining = {}", remaining);
            if remaining == 9 {
                break;
            }
            if count == 2 {
                break 'counting_up;
            }
            remaining -= 1;
        }

        count += 1;
    }
    println!("End count = {}", count);
}
```

输出如下
```Rust
$ cargo run
   Compiling loops v0.1.0 (file:///projects/loops)
    Finished dev [unoptimized + debuginfo] target(s) in 0.58s
     Running `target/debug/loops`
count = 0
remaining = 10
remaining = 9
count = 1
remaining = 10
remaining = 9
count = 2
remaining = 10
End count = 2
```

可以看到，`break 'counting_up`语句，将程序从外层循环中跳出

##### 从循环返回
```Rust
fn main() {
    let mut counter = 0;

    let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2;
        }
    };

    println!("The result is {}", result);
}
```
这个写法会让result的值变为20。

#### for, while loop
和大部分程序语言尤其是C++是一样的

```Rust
fn main() {
    let a = [10, 20, 30, 40, 50];

    for element in a.iter() {
        println!("the value is: {}", element);
    }
}
```
重点是，需要用`a.iter()`方法

如果想要**遍历一个范围**我们可以。。。

在python中我们使用`for i in range(1, 10)`
在Rust中我们可以使用`for i in (1..10)`

---

我们已经掌握了第三章的内容，我们现在去看第四章吧！
第四章也是Rust很核心的一个内容，即`ownership`机制，他让Rust无需GC就可以保障内存的安全！