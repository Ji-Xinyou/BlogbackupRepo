---
title: Rust(course.rs)-generic&trait
date: 2022-02-14 09:01:39
tags: Rust
categories: ProgLang
---

<div></div>

<!--more-->



# Generic and Trait

---

## Generic

下面这段代码**无法编译**

```rust
fn largest<T>(list: &[T]) -> T {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!("The largest number is {}", result);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest(&char_list);
    println!("The largest char is {}", result);
}
```



报错的原因是

* 不是所有类型都可以进行`>`的比较
  * 因此我们需要给`T`天界一个类型限制 -> trait
  * 我们使用`std::cmp::PartialOrd`Trait进行限制



```rust
fn add<T: std::ops::Add<Output = T>>(a:T, b:T) -> T {
    a + b
}
```

---

### Generic in methods

```rust
struct Point<T, U> {
    x: T,
    y: U,
}

impl<T, U> Point<T, U> {
    fn mixup<V, W>(self, other: Point<V, W>) -> Point<T, W> {
        Point {
            x: self.x,
            y: other.y,
        }
    }
}

fn main() {
    let p1 = Point { x: 5, y: 10.4 };
    let p2 = Point { x: "Hello", y: 'c'};

    let p3 = p1.mixup(p2);

    println!("p3.x = {}, p3.y = {}", p3.x, p3.y);
}
```

---

### Generic in const

`[i32; 2], [i32; 3]`是两种类型，我们可以用泛型打印所有的数组元素

通过**引用**，我们可以实现

```rust
fn display_array<T: std::fmt::Debug>(arr: &[T]) {
    println!("{:?}", arr);
}
fn main() {
    let arr: [i32; 3] = [1, 2, 3];
    display_array(&arr);

    let arr: [i32;2] = [1,2];
    display_array(&arr);
}
```



* 需要注意的是，我们需要对类型`T`添加限制，因为我们需要他支持Debug输出`{:?}`



**有时候，引用是不合适的**，对于任意的长度，我们可以使用const泛型

```rust
fn display_array<T: std::fmt::Debug, const N: usize>(arr: [T; N]) {
    println!("{:?}", arr);
}
fn main() {
    let arr: [i32; 3] = [1, 2, 3];
    display_array(arr);

    let arr: [i32; 2] = [1, 2];
    display_array(arr);
}
```

---

## Trait

如果不同类型有相同行为，我们可以为他定义一个trait

* 把一些方法组合在一起，实现某些目标必需的行为的集合



Trait只定义一个行为**看起来是什么样的**，而不具体定义行为是什么样的

* 因此我们只指定特征方法的签名

```rust
pub trait Summary {
  fn summarize(&self) -> String;
}
```



### 孤儿规则

> **如果你想要为类型 `A` 实现特征 `T`，那么 `A` 或者 `T` 至少有一个是在当前作用域中定义的!**

因此你没有办法为`String`实现`Display`，这保证了其他人编写的代码不会破坏你的代码



下面的代码没有办法编译，因为我们没有在`std::fmt`中实现Display，由于孤儿规则，这是不被允许的，那我们该怎么跳过他呢，我们有时候就是需要这样！

```rust
use std::fmt;

impl fmt::Display for Point {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "({}, {})", self.x, self.y)
    }
}
```

我们可以使用`newtype`，具体做法如下

```rust
use std::fmt;

struct Wrapper(Vec<String>);

impl fmt::Display for Wrapper {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "[{}]", self.0.join(", "))
    }
}

fn main() {
    let w = Wrapper(vec![String::from("hello"), String::from("world")]);
    println!("w = {}", w);
}
```

这个不好的地方在于，我们需要从`Wrapper`中取出`vec<string>`，类似所有的数组方法都需要使用`self.0`来调用数组



那用什么解决方法解决这个`self.0`的问题呢，Rust有一个Trait叫做`Deref`，实现之后可以自动做类型转换的操作，像数组一样使用`newtype`



// TODO: deref trait



### 默认实现

```rust
pub trait Summary {
    fn summarize(&self) -> String {
        String::from("(Read more...)")
    }
}
```



默认实现可以调用相同trait里面的其他方法，这个方法被对应的struct实现了就可以了，而不一定要在trait里有默认实现



## Trait 作为函数参数

trait牛逼的地方之一

```rust
pub fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}
```

什么叫做`item: &impl Summary`

> 它的意思是 `实现了Summary特征` 的 `item` 参数

如果没有实现Summary trait，不能作为参数

### trait还可作为返回值

返回值是一个`实现了Summary Trait`的类

```rust
fn returns_summarizable() -> impl Summary {
    Weibo {
        username: String::from("sunface"),
        content: String::from(
            "m1 max太厉害了，电脑再也不会卡",
        )
    }
}
```

但是，这种返回值只能有一个具体的类型

具体如下

```rust
fn returns_summarizable(switch: bool) -> impl Summary {
    if switch {
        Post {
            title: String::from(
                "Penguins win the Stanley Cup Championship!",
            ),
            author: String::from("Iceburgh"),
            content: String::from(
                "The Pittsburgh Penguins once again are the best \
                 hockey team in the NHL.",
            ),
        }
    } else {
        Weibo {
            username: String::from("horse_ebooks"),
            content: String::from(
                "of course, as you probably already know, people",
            ),
        }
    }
}
```

这段代码无法通过，因为一个branch返回了`Post`另一个返回了`Weibo`

如果需要返回不同的类型，需要使用`特征对象` - `trait object`

## 特征对象

可以通过 `&` 引用或者 `Box<T>` 智能指针的方式来创建特征对象:

```rust
trait Draw {
    fn draw(&self) -> String; 
}

impl Draw for u8 {
    fn draw(&self) -> String {
        format!("u8: {}", *self) 
    } 
}

impl Draw for f64 {
    fn draw(&self) -> String {
        format!("f64: {}", *self) 
    } 
}

fn draw1(x: Box<dyn Draw>) {
    x.draw();
}

fn draw2(x: &dyn Draw) {
    x.draw();
}

fn main() {
    let x = 1.1f64;
    // do_something(&x);
    let y = 8u8;

    draw1(Box::new(x));
    draw1(Box::new(y));
    draw2(&x);
    draw2(&y);
}
```

* `&dyn traitname`或者`Box<dyn Draw>`表达的是实现了`Draw Trait`的对象

也许你会想，和泛型有什么不同呢？为什么不用泛型呢

我们来看看泛型和特征对象代码的区别



* **特征对象**对于screen类的定义

```rust
pub struct Screen {
    pub components: Vec<Box<dyn Draw>>,
}

impl Screen {
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}
```



* **泛型**

```rust
pub struct Screen<T: Draw> {
    pub components: Vec<T>,
}

impl<T> Screen<T>
    where T: Draw {
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}
```

我们可以看出来，对于第二种来说，`components: Vec<T>`限制了所有的component必须是**同一类型**，这是不灵活的，但是开销更低

---

## Trait Bound

```rust
pub fn notify(item1: &impl Summary, item2: &impl Summary) {}

pub fn notify<T: Summary>(item1: &T, item2: &T) {}
```

第一个语法只能保证两个参数都实现了Summary Trait

第二个可以保证他们是同一个类型

### multiple bound

```rust
pub fn notify(item: &(impl Summary + Display)) {}

pub fn notify<T: Summary + Display>(item: &T) {}
```

### where bound

有时候，函数签名会变得很复杂

```rust
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32
```

我们用**where**可以简化

```rust
fn some_function<T, U>(t: &T, u: &U) -> i32
    where T: Display + Clone,
          U: Clone + Debug
{}
```

### 用Trait 约束 Trait

在下面的代码我们可以看到，如果我们要实现`OutlinkPrint` 这个Trait，我们必须先实现`Display`这个Trait

```rust
use std::fmt::Display;

trait OutlinePrint: Display {
    fn outline_print(&self) {
        let output = self.to_string();
        let len = output.len();
        println!("{}", "*".repeat(len + 4));
        println!("*{}*", " ".repeat(len + 2));
        println!("* {} *", output);
        println!("*{}*", " ".repeat(len + 2));
        println!("{}", "*".repeat(len + 4));
    }
}
```

### 使用derive来派生Trait

#[derive(Debug, Copy ....)]

---

## 调用不同trait的同名函数

```rust
trait Pilot {
    fn fly(&self);
}

trait Wizard {
    fn fly(&self);
}

struct Human;

impl Pilot for Human {
    fn fly(&self) {
        println!("This is your captain speaking.");
    }
}

impl Wizard for Human {
    fn fly(&self) {
        println!("Up!");
    }
}

impl Human {
    fn fly(&self) {
        println!("*waving arms furiously*");
    }
}
```



1. 默认调用Human::fly()

2. 如果需要调用Pilot::fly()或者Wizard::fly

   1. ```rust
      fn main() {
          let person = Human;
          Pilot::fly(&person); // 调用Pilot特征上的方法
          Wizard::fly(&person); // 调用Wizard特征上的方法
          person.fly(); // 调用Human类型自身的方法
      }
      ```

**那如果没有self参数呢？**

```rust
trait Animal {
    fn baby_name() -> String;
}

struct Dog;

impl Dog {
    fn baby_name() -> String {
        String::from("Spot")
    }
}

impl Animal for Dog {
    fn baby_name() -> String {
        String::from("puppy")
    }
}

fn main() {
    println!("A baby dog is called a {}", Dog::baby_name());
}
```

如果我们调用

`Animal::baby_name()`，时会报错的，因为**有很多struct可能实现了`Animal Trait`**

我们需要用**完全限定语法**

`<Dog as Animal>::baby_name();`



完全限定语法的形式为

```rust
<Type as Trait>::function(receiver_if_method, next_arg, ...);
```

`receiver_if_method`就是三种`self`, 如果function是关联函数就没有这一项









