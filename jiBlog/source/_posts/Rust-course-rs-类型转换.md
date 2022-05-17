---
title: Rust(course.rs)-类型转换
date: 2022-02-28 14:23:16
tags: Rust
categories: ProgLang
---

<div></div>

<!--more-->



# Rust - 类型转换

## as cast

我们一般将**较小的**数据类型转为**较大**的

```rust
let a = 100_i8 as i32;
let c = 'a' as u8;
```

### 还可以将内存地址转为指针

重点是`values.as_mut_ptr()`

```rust
let mut values: [i32; 2] = [1, 2];
let p1: *mut i32 = values.as_mut_ptr();
let first_address = p1 as usize; // 将p1内存地址转换为一个整数
let second_address = first_address + 4; // i32类型占用4个字节，因此将内存地址 + 4
let p2 = second_address as *mut i32; // 访问该地址指向的下一个整数p2
unsafe {
    *p2 += 1;
}
assert_eq!(values[1], 3);
```

### 边角知识

1. 数组切片原生指针之间的转换，不会改变数组占用的内存字节数，尽管数组元素的类型发生了改变：

```rust
fn main() {
    let a: *const [u16] = &[1, 2, 3, 4, 5];
    let b = a as *const [u8];
    assert_eq!(std::mem::size_of_val(&a), std::mem::size_of_val(&b))
}
```

---

## Tryinto cast

`try_into`是`TryInto Trait`的一个方法，他会捕获我们大类型向小类型转换的溢出错误

* 返回的是`Result`

```rust
use std::convert::TryInto;
fn main() {
    let b: i16 = 1500;
    let b_: u8 = match b.try_into() {
        Ok(b1) => b1,
        Err(e) => {
            println!("{:?}", e.to_string());
            0
        }
    };
}
```

* 如果我们用`unwrap()`或者`expect()`来处理`Result<T, E>`
  * unwrap会panic！，expect会发送错误信息

---

## General cast

1. 一个`Trait`为`&i32`实现了，`&mut i32`可以转成`&i32`，但是他仍然不能用这个`Trait`

```rust
trait Trait {}

fn foo<X: Trait>(t: X) {}
impl<'a> Trait for &'a i32 {}

fn main() {
    let t: &mut i32 = &mut 0;
    foo(t);
}

error[E0277]: the trait bound `&mut i32: Trait` is not satisfied
--> src/main.rs:9:9
|
9 |     foo(t);
|         ^ the trait `Trait` is not implemented for `&mut i32`
|
= help: the following implementations were found:
        <&'a i32 as Trait>
= note: `Trait` is implemented for `&i32`, but not for `&mut i32`
```



### 点操作符

建议看网页

https://course.rs/basic/converse.html



暂时决定停更了，语言还是边写边学比较好，尽管Rust语言特性很多，可能一开始记录下来是很低效的
