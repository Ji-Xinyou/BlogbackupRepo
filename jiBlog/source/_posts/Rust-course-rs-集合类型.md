---
title: Rust(course.rs)-集合类型
date: 2022-02-24 23:40:33
tags: Rust
categories: ProgLang
---

<div></div>

<!--more-->



# 集合类型

主要关于`Vector`和`Hashmap`

---

## Vector

### Vec::new()

rust编译器可以根据操作推测类型

```rust
let v: Vec<i32>: Vec::new()

or...

let mut v = Vec::new()
v.push(1)
```

> 如果预先知道要存储的元素个数，可以使用 `Vec::with_capacity(capacity)` 创建动态数组，这样可以避免因为插入大量新数据导致频繁的内存分配和拷贝，提升性能



### vec![];

```rust
let v = vec![1, 2, 3];
```



* Vector和他的元素的作用域一致
  * Vector被删除后，内容也会被删除
    * 如果内部有元素被引用之后，会比较复杂



### index

有`索引`或者`vec.get()`的方式，需要注意的是`vec.get()`返回的是`Option<&T>`

```rust
let v = vec![1, 2, 3, 4, 5];

let third: &i32 = &v[2];
println!("第三个元素是 {}", third);

match v.get(2) {
    Some(third) => println!("第三个元素是 {}", third),
    None => println!("去你的第三个元素，根本没有！"),
}
```



#### tricky point

下面这段代码无法编译，为什么呢？

原因是，虽然看似`不可变引用`以及`push`不相干，但是如果vec的大小不够用的时候，**vec会开辟一个新的空间，然后把内存拷贝过去！**

**这样的话原本的不可变引用就指向了一块无效内存！！！！**

How amazing!

```rust

#![allow(unused)]
fn main() {
let mut v = vec![1, 2, 3, 4, 5];

let first = &v[0];

v.push(6);

println!("The first element is: {}", first);
}
```



### iterate

```rust
let mut v = vec![1, 2, 3];
for i in &mut v {
    *i += 10
}
```



### 存不同类型的变量

有两种方式存储不同类型的变量

* enum
* Trait对象 (&dyn trait)

#### enum

优点是简单，缺点是无法动态增加类型，且限制较多

```rust
#[derive(Debug)]
enum IpAddr {
    V4(String),
    V6(String)
}

fn show_addr(ip: IpAddr) {
    println!("{:?}",ip);
}

fn main() {
    let v = vec![
        IpAddr::V4("127.0.0.1".to_string()),
        IpAddr::V6("::1".to_string())
    ];

    for ip in v {
        show_addr(ip)
    }
}
```

#### trait object

比较常用，灵活，但比较复杂

```rust
trait IpAddr {
    fn display(&self);
}

struct V4(String);
impl IpAddr for V4 {
    fn display(&self) {
        println!("ipv4: {:?}",self.0)
    }
}
struct V6(String);
impl IpAddr for V6 {
    fn display(&self) {
        println!("ipv6: {:?}",self.0)
    }
}

fn main() {
    let v: Vec<Box<dyn IpAddr>> = vec![
        Box::new(V4("127.0.0.1".to_string())),
        Box::new(V6("::1".to_string())),
    ];

    for ip in v {
        ip.display();
    }
}
```

---

## HashMap

1. 不在`prelude`中，需要

```rust
use std::collections::HashMap;
```

## 创建

`key`和`value`分别必须为相同的类型

```rust
use std::collections::HashMap;

let mut my_gems = HashMap::new();

my_gems.insert("红宝石", 1);
```

* 如果提前知道大小，可以指定大小 `HashMap::with_capacity(capacity)`

### 用iter和collect

```rust
let mut info_vec: vec<(String, u32)>;
let mut info_map = HashMap::new()

for info in &info_vec {
  info_map.insert(&team.0, &team.1);
}
```

* **MORE RUSTY**

```rust
fn main() {
    use std::collections::HashMap;

    let teams_list = vec![
        ("中国队".to_string(), 100),
        ("美国队".to_string(), 10),
        ("日本队".to_string(), 50),
    ];

    let teams_map: HashMap<_,_> = teams_list.into_iter().collect();
    
    println!("{:?}",teams_map)
}
```

* `into_iter()`将Vec转成iter
* `collect()`将iter收集
  * 内置支持多种集合类型，因此需要告诉编译器为`HashMap<_, _>`

### ownership

* 如果有`copy trait`，复制进去
* 如果没有，所有权转移



如果将`reference`放入了`Hashmap`，请保证`ref`的生命周期至少跟`HashMap`一样久

```rust
fn main() {
    use std::collections::HashMap;

    let name = String::from("Sunface");
    let age = 18;

    let mut handsome_boys = HashMap::new();
    handsome_boys.insert(&name, age);

    std::mem::drop(name);
    println!("因为过于无耻，{:?}已经被除名", handsome_boys);
    println!("还有，他的真实年龄远远不止{}岁",age);
}
```

上面这段代码是有问题的，因为name已经被析构了，而Hashmap使用的是他的引用



### 查询

#### get()

```rust
let mut scores: HashMap<String, u32> = HashMap::new();
...
let info: Option<&i32> = scores.get("Team1");
```

`get(key)`返回的是`value`的`reference`，如果咨询不到，会是None

#### iter

```rust
let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

for (key, value) in &scores {
    println!("{}: {}", key, value);
}
```



### 更新

```rust
fn main() {
    use std::collections::HashMap;

    let mut scores = HashMap::new();

    scores.insert("Blue", 10);

    // 覆盖已有的值
    let old = scores.insert("Blue", 20);
    assert_eq!(old, Some(10));

    // 查询新插入的值
    let new = scores.get("Blue");
    assert_eq!(new, Some(&20));
    
    // 查询Yellow对应的值，若不存在则插入新值
    let v = scores.entry("Yellow").or_insert(5);
    assert_eq!(*v, 5); // 不存在，插入5

    // 查询Yellow对应的值，若不存在则插入新值
    let v = scores.entry("Yellow").or_insert(50);
    assert_eq!(*v, 5); // 已经存在，因此50没有插入
}
```

* `HashMap.insert(K, V)`，如果原本存在`K`对应的值，会把原本的`V`返回，并将新的`<K, V>`插入
* `HashMap.get(K)`，返回`Option(&V)`
* `HashMap.entry(K).or_insert(V)`返回的是`&mut v`引用
  * 如果要修改值，要先进行`deref`



### 第三方库 - 哈希函数

```rust

#![allow(unused)]
fn main() {
use std::hash::BuildHasherDefault;
use std::collections::HashMap;
// 引入第三方的哈希函数
use twox_hash::XxHash64;

// 指定HashMap使用第三方的哈希函数XxHash64
let mut hash: HashMap<_, _, BuildHasherDefault<XxHash64>> = Default::default();
hash.insert(42, "the answer");
assert_eq!(hash.get(&42), Some(&"the answer"));
}
```

