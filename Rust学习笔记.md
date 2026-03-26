# Rust学习笔记



## Cargo

### 命令

1. cargo run 编译运行项目
2. cargo new 创建一个新项目
3. cargo add 添加依赖 不加版本号默认最新  
4. cargo build 编译项目
5. cargo check 确保项目可以编译
6. cargo build --release 启用优化的方式编译项目

### cargo.toml

​	cargo的依赖版本管理，cargo add rand@0.8.5这样会在依赖里加上

 ```yaml
 rand = "0.8.5" ##表是大于0.8.5版本但小于0.9.0版本的随机数依赖
 ```

### rand

```rust
use rand::Rng;

let secret_number = rand::thread_rng().gen_range(1..101);  
//生成一个 1~100 之间的随机整数
//1..101 是 Rust 的范围语法，表示 1 到 100（左闭右开，不包含 101）
```



## 常见概念



### 变量的可变与不可变

有mut表示可变
```rust
let mut var = "123"
var = "223" // 可行
```

无则表示不可变

```rust
let var = "123"
var = "223" //报错
```



### 常量

1. 使用const作为关键字
2. 永远不可变
3. 可以声明在任何作用域
4. 不能是运行时计算出来的值(用变量计算)
5. 命名约定大小写加下划线
6. 将硬编码的提取为常量，有助于别人维护项目

```rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```

## 遮蔽(可以定义与之前命名相同的变量)

### 一个变量定义了居然还能再定义，将原来的遮蔽

注意有作用域的区分

```rust
    let x = 5;
    let x = x + 1; //这样是可以的

    let spaces = "   ";
    let spaces = spaces.len(); //也是可以的
```



## 数据类型



### 当上下文不足以唯一确定类型时，Rust 会要求显式标注类型

```rust
let guess= "42".parse().expect("Not a number!"); //这样写是错误的,因为返回泛型

let guess: u32 = "42".parse().expect("Not a number!");
```

会报错，不是因为 Rust “推断能力差”，而是因为 parse() 返回的是泛型结果，编译器无法唯一确定目标类型。



-----



### 使用移动的变量与数据交互

```rust
    let x = 5;
    let y = x; //猜猜是赋值是拷贝值还是指针指向？
```

以上数据都是在栈上，那么栈就是指针了，所以拷贝的是指针、长度和容量。没有复制指向堆的数据

```rust
    let str = String::from("Hello, world!");
    let str1 = str;
    println!("{}", str); //这样写喜提报错。
//这个是为了避免二次释放，为什么呢。这两个变量是在同一个作用域吧！
//那么作用域结束的时候，是不是会释放呢？
//那就会释放两次(二次释放)两次释放（相同）内存会导致内存污染，它可能会导致潜在的安全漏洞。
//所以在你执行了 let str1 = str;后 str将被rust认为不再有效，所以不要使用str。
```

这个操作被称为移动(类似浅拷贝)，但原来的变量不能使用。解读为 str移动到str1。

### 使用克隆与数据交互

```rust
    let s1 = String::from("hello");
    let s2 = s1.clone();

    println!("s1 = {s1}, s2 = {s2}"); //这段代码可以运行，复制了堆上的数据给他
```

### 只在栈上的数据拷贝

```rust
    let x = 5;
    let y = x;

    println!("x = {x}, y = {y}"); //这样是可以的，因为已知大小的数据在栈上，所以这样拷贝是可以的
```

Rust 有一个叫做 `Copy` trait 的特殊注解，可以用在类似整型这样的存储在栈上的类型上

如果一个类型实现了 `Copy` trait，那么一个旧的变量在将其赋值给其他变量后仍然有效

Rust 不允许自身或其任何部分实现了 `Drop` trait 的类型使用 `Copy` trait。

那么哪些类型实现了 `Copy` trait 呢？

可以查看给定类型的文档来确认，不过作为一个通用的规则，任何一组简单标量值的组合都可以实现 `Copy`

如下是一些 `Copy` 的类型：

- 所有整数类型，比如 `u32`。
- 布尔类型，`bool`，它的值是 `true` 和 `false`。
- 所有浮点数类型，比如 `f64`。
- 字符类型，`char`。
- 元组，当且仅当其包含的类型也都实现 `Copy` 的时候。比如，`(i32, i32)` 实现了 `Copy`，但 `(i32, String)` 就没有。

### 所有权与函数

将值传递给函数与给变量赋值的原理相似。向函数传递值可能会移动或者复制，就像赋值语句一样。

```rust
fn main() {
    let s = String::from("hello");  // s 进入作用域

    takes_ownership(s);             // s 的值移动到函数里 ...
                                    // ... 所以到这里不再有效 再用s就报错

    let x = 5;                      // x 进入作用域

    makes_copy(x);                  // x 应该移动函数里， 
                                    // 但 i32 是 Copy 的，再用就可以
    println!("{}", x);              // 所以在后面可继续使用 x

} // 这里，x 先移出了作用域，然后是 s。但因为 s 的值已被移走，
  // 没有特殊之处

fn takes_ownership(some_string: String) { // some_string 进入作用域
    println!("{some_string}");
} // 这里，some_string 移出作用域并调用 `drop` 方法。
  // 占用的内存s被释放,s在这里就被释放了

fn makes_copy(some_integer: i32) { // some_integer 进入作用域
    println!("{some_integer}");
} // 这里，some_integer 移出作用域。没有特殊之处
```

### 返回值与作用域

返回值也可以转移所有权。

```rust
fn main() {
    let s1 = gives_ownership();        // gives_ownership 将它的返回值传递给 s1

    let s2 = String::from("hello");    // s2 进入作用域

    let s3 = takes_and_gives_back(s2); // s2 被传入 takes_and_gives_back, 
                                       // 它的返回值又传递给 s3
} // 此处，s3 移出作用域并被丢弃。s2 被 move，所以无事发生
  // s1 移出作用域并被丢弃

fn gives_ownership() -> String {       // gives_ownership 将会把返回值传入
                                       // 调用它的函数

    let some_string = String::from("yours"); // some_string 进入作用域

    some_string                        // 返回 some_string 并将其移至调用函数
}

// 该函数将传入字符串并返回该值
fn takes_and_gives_back(a_string: String) -> String {
    // a_string 进入作用域

    a_string  // 返回 a_string 并移出给调用的函数
}
```

变量的所有权总是遵循相同的模式：将值赋给另一个变量时它会移动。当持有堆中数据值的变量离开作用域时，其值将通过 `drop` 被清理掉，除非数据被移动为另一个变量所有。

虽然这样是可以的，但是在每一个函数中都获取所有权并接着返回所有权有些啰嗦。

如果我们想要函数使用一个值但不获取所有权该怎么办呢？如果我们还要接着使用它的话，每次都传进去再返回来就有点烦人了，除此之外，我们也可能想返回函数体中产生的一些数据。

我们可以使用元组来返回多个值

```rust
fn main() {
    let s1 = String::from("hello");

    let (s2, len) = calculate_length(s1);

    println!("The length of '{s2}' is {len}.");
}

fn calculate_length(s: String) -> (String, usize) {
    let length = s.len(); // len() 返回字符串的长度

    (s, length)
}
```

但是这未免有些形式主义，而且这种场景应该很常见。幸运的是，Rust 对此提供了一个不用获取所有权就可以使用值的功能，叫做 **引用**（*references*）。



## 引用与借用

### 引用

  String 是有所有权的类型。
  当你把一个 String 作为参数传给 calculate_length 时，所有权会移动到这个函数里。

所谓借用就是函数里传递引用。示例如下

```rust
fn main() {
    let str = String::from("Hello, world!");
    let length = calculate_length(&str); //函数通过引用借用原来的值
    println!("The length of '{}' is {}.", str, length);
}

fn calculate_length(str: &String) -> usize {
    return str.len();
}

```

与使用 `&` 引用相反的操作是 **解引用**（*dereferencing*），它使用解引用运算符 `*` 实现。

但是上面的数据没有加mut所以是不可变的引用。

```rust
fn main() {
    let mut s = String::from("hello"); //加mut

    change(&mut s); //可变引用
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

### 可变引用

可变引用有一个很大的限制：如果你有一个对该变量的可变引用，你就不能再创建对该变量的引用。

(在同一个作用域不能出现两个可变引用)

```rust
    let mut s = String::from("hello");  //下面的行为会报错。

    let r1 = &mut s;
    let r2 = &mut s;

    println!("{r1}, {r2}");
```

```rust
    let mut s = String::from("hello"); //以下行为完全没有问题

    {
        let r1 = &mut s;
    } // r1 在这里离开了作用域，所以我们完全可以创建一个新的引用

    let r2 = &mut s;
```

Rust 在同时使用可变与不可变引用时也强制采用类似的规则。 

可变引用可以有多个。

不能在拥有不可变引用的同时拥有可变引用

如果不可变引用的作用域，在其最后一次使用后消失，所以可变引用可以在不可变消失后使用

```rust
    let mut s = String::from("hello");

    let r1 = &s; // 没问题
    let r2 = &s; // 没问题
    println!("{r1} and {r2}");
    // 此位置之后 r1 和 r2 不再使用

    let r3 = &mut s; // 没问题
    println!("{r3}");
```

```rust
fn main() {
    let mut str = String::from("Hello, world!");
    let mystr = &mut str;
    let length = calculate_length(mystr); //从可变引用到不可变引用的自动强制转换
    mystr.push_str("!");
    println!("{},{}", mystr, length);
}

fn calculate_length(str: &String) -> usize {
    return str.len();
}

```

### 悬垂引用

在带有指针的语言中，如果释放了一块内存，却保留了指向它的指针，就很容易错误地制造出一个**悬垂指针**（*dangling pointer*）。

在 Rust 中，编译器保证引用永远不会变成悬垂引用。

```rust
fn main() {
    let reference_to_nothing = dangle();
}

fn dangle() -> &String {
    let s = String::from("hello"); //作用域到我就死了

    &s //作为死人你为什么要返回我的引用？
}
```

引用不等于所有权，给你引用，所有权还在我这里，生命到了，我就走了，不管你了

```rust
fn dangle() -> &String { // dangle 返回一个字符串的引用

    let s = String::from("hello"); // s 是一个新字符串

    &s // 返回字符串 s 的引用
} // 这里 s 离开作用域并被丢弃。其内存被释放。
  // 危险！
```

解决方案返回所有权

```rust
fn no_dangle() -> String {
    let s = String::from("hello");

    s
}
```

规则

- 在任意给定时间，**要么**只能有一个可变引用，**要么**只能有多个不可变引用。
- 引用必须总是有效的。

## 切片

### slice

**字符串 slice**（*string slice*）是 `String` 中一部分值的引用，它看起来像这样：

```rust
    let s = String::from("hello world");

    let hello = &s[0..5];
    let world = &s[6..11];
```

```rust
fn first_word(s: &String) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i]; 
        }
    }

    &s[..] //&s作为返回就完全不行了
}
```



  - s：路标，指向外部字符串
  - &s：借路标本身，不行
  - &s[..1]：顺着路标找到字符串，再借其中一段，可以

### 字符串字面值就是slice

```rust
let s = "Hello, world!";
```

这里 `s` 的类型是 `&str`：它是一个指向二进制程序特定位置的 slice。这也就是为什么字符串字面值是不可变的；`&str` 是一个不可变引用。

在知道了能够获取字面值和 `String` 的 slice 后，我们对 `first_word` 做了改进，这是它的签名：

```rust
fn first_word(s: &String) -> &str {
```

而更有经验的 Rustacean 会编写出示例 4-9 中的签名，因为它使得可以对 `&String` 值和 `&str` 值使用相同的函数：

```rust
fn first_word(s: &str) -> &str {
```

字符串 slice，正如你想象的那样，是针对字符串的。不过也有更通用的 slice 类型。考虑一下这个数组：

```rust
let a = [1, 2, 3, 4, 5];
```

就跟我们想要获取字符串的一部分那样，我们也会想要引用数组的一部分。我们可以这样做：

```rust
let a = [1, 2, 3, 4, 5];

let slice = &a[1..3];

assert_eq!(slice, &[2, 3]);
```

----



### 通过派生trait增加功能

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!("rect1 is {rect1}"); //报错
}

```

`println!` 宏能处理很多类型的格式，`{}` 默认告诉 `println!` 使用被称为 `Display` 的格式。

意在提供给直接终端用户查看的输出。

目前为止见过的基本类型都默认实现了 `Display`，因为它就是向用户展示 `1` 或其他任何基本类型的唯一方式。

不过对于结构体，`println!` 应该用来输出的格式是不明确的，因为这有更多显示的可能性：是否需要逗号？需要打印出大括号吗？所有字段都应该显示吗？由于这种不确定性，Rust 不会尝试猜测我们的意图，所以结构体并没有提供一个 `Display` 实现来使用 `println!` 与 `{}` 占位符。

```rust
error[E0277]: `Rectangle` doesn't implement `std::fmt::Display`
  --> src\main.rs:12:24
   |
12 |     println!("rect1 is {rect1}"); //报错
   |                        ^^^^^^^ `Rectangle` cannot be formatted with the default formatter
   |
help: the trait `std::fmt::Display` is not implemented for `Rectangle`
  --> src\main.rs:1:1
   |
 1 | struct Rectangle {
   | ^^^^^^^^^^^^^^^^
   = note: in format strings you may be able to use `{:?}` (or {:#?} for pretty-print) instead
   = note: this error originates in the macro `$crate::format_args_nl` which comes from the expansion of the macro `println` (in Nightly builds, run with -Z macro-backtrace for more info)

```

告诉我是不是想要一种{:?} 这样的形式

```rust
    println!("rect1 is {rect1:?}"); //报错

```

```rust
12 |     println!("rect1 is {rect1:?}"); //报错
   |                        ^^^^^^^^^ `Rectangle` cannot be formatted using `{:?}` because it doesn't implement `Debug`
   |
```

此时依然报错，指出没有实现debug

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!("rect1 is {rect1:?}"); //成功输出
}
```

 另一种使用`Debug` 格式打印数值的方法是使用 [`dbg!` 宏](https://doc.rust-lang.org/std/macro.dbg.html)。`dbg!` 宏接收一个表达式的所有权（与 `println!` 宏相反，后者接收的是引用）

```rust
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

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    dbg!(&rect1);
    println!("{rect1:?}", );
    println!("{:#?}", rect1);
}
```

想要弄清楚代码在做什么dbg很有用

## 方法

### 方法

**方法**（method）与函数类似：它们使用 `fn` 关键字和名称声明，可以拥有参数和返回值，同时包含在某处调用该方法时会执行的代码

不过方法与函数是不同的，因为它们在**结构体的上下文**中被定义

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle { //接口
    fn area(&self) -> u32 { //& 来表示这个方法借用了 Self 实例
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

除了可使用方法语法和不需要在每个函数签名中重复 `self` 的类型之外，其主要好处在于组织性。我们将某个类型实例能做的所有事情都一起放入 `impl` 块中，而不是让将来的用户在我们的库中到处寻找 `Rectangle` 的功能。

```rust
impl Rectangle {
    fn width(&self) -> bool {
        self.width > 0
    }
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    if rect1.width() {
        println!("The rectangle has a nonzero width; it is {}", rect1.width); //字段与方法同名
    }
}
```

字段与方法同名的时候

在 `main` 中，当我们在 `rect1.width` 后面加上括号时。Rust 知道我们指的是方法 `width`。当我们不使用圆括号时，Rust 知道我们指的是字段 `width`。

**有没有圆括号的区别**



### 关联函数

所有在 `impl` 块中定义的函数被称为 **关联函数**（*associated functions*），因为它们与 `impl` 后面命名的类型相关。

我们可以定义不以 `self` 为第一参数的关联函数（**因此不是方法**），因为它们并**不作用于一个结构体的实例**。我们已经使用了一个这样的函数：在 `String` 类型上定义的 `String::from` 函数。

不是方法的关联函数经常被用作返回一个结构体新实例的**构造函数**。这些函数的名称通常为 `new` ，但 `new` 并不是一个关键字。例如我们可以提供一个叫做 `square` 关联函数，它接受一个维度参数并且同时作为宽和高，这样可以更轻松的创建一个正方形 `Rectangle`

```rust
impl Rectangle {
    fn square(size: u32) -> Self {
        Self {
            width: size,
            height: size,
        } //构造函数
    }
}
```

**不带 &self**，所以可以直接类型调用

用这个关联函数，我们使用结构体名和 `::` 语法；比如 `let sq = Rectangle::square(3);`

`::` 语法用于关联函数和模块创建的命名空间。

### 多个impl块

每个结构体都允许拥有多个 `impl` 块

没有理由将这些方法分散在多个 `impl` 块中，不过这是有效的语法。

## 枚举

**枚举**（*enumerations*），也叫作 *enums*。枚举让你可以通过列举某个类型所有可能的 **变体**（*variants*）来定义这个类型。

### 枚举的定义

结构体给予你将字段和数据聚合在一起的方法，像 `Rectangle` 结构体有 `width` 和 `height` 两个字段。

而枚举给予你一个途径去声明某个值是一个集合中的一员。比如，我们想让 `Rectangle` 是一些形状的集合，包含 `Circle` 和 `Triangle` 。

为此，Rust 允许我们将这些可能性编码为一个枚举类型。

让我们看看一个需要诉诸于代码的场景，来考虑为何此时使用枚举更为合适且实用。

假设我们要**处理 IP 地址**。目前被广泛使用的两个主要 IP 标准：**IPv4**（version four）和 **IPv6**（version six）。

这是我们的程序可能会遇到的所有可能的 IP 地址类型：所以可以**枚举**出所有可能的值，这也正是枚举一词的由来。

任何一个 IP 地址要么是 IPv4 的要么是 IPv6 的，而且不能两者都是。

可以通过在代码中定义一个 `IpAddrKind` 枚举来表现这个概念并列出可能的 IP 地址类型，`V4` 和 `V6`。这被称为枚举的**变体**（*variants*）：

```rust
enum IpAddrKind {
    V4,
    V6,
}
```

现在 `IpAddrKind` 就是一个可以在代码中使用的自定义数据类型了。

### 枚举值

可以像这样创建 `IpAddrKind` 两个不同变体的实例：

```rust
	let four = IpAddrKind::V4;
    let six = IpAddrKind::V6;
```

注意枚举的变体位于其标识符的命名空间中，并使用两个冒号分开。这么设计的益处是现在 `IpAddrKind::V4` 和 `IpAddrKind::V6` 都是 `IpAddrKind` 类型的。例如，接着可以定义一个函数来接收任何 `IpAddrKind`类型的参数：

```rust
fn main() {
    let four = IpAddrKind::V4;
    let six = IpAddrKind::V6;
    route(four);
    route(six);
}
fn route(ip_type: IpAddrKind) {
    println!("route ip {:?}", ip_type);
}
//输出
//route ip V4
//route ip V6
```

结构体结合枚举

```rust
fn main() {
    let myIp = IpAddr::new(IpAddrKind::V4,String::from("192.168.1.1"));
    println!("{myIp:?}", );
}
#[derive(Debug)]
enum IpAddrKind {
    V4,
    V6
}
impl IpAddr {
    // 静态方法：关联函数
    fn new(kind: IpAddrKind, address: String) -> Self {
        Self { kind, address }
    }
}
#[derive(Debug)]
struct IpAddr {
    kind: IpAddrKind,
    address: String
}
```

更简单的用法

可以使用一种更简洁的方式来表达相同的概念，仅仅使用枚举并将数据直接放进每一个枚举变体而**不是将枚举作为结构体**的一部分。

```rust
use std::net::{IpAddr, Ipv6Addr};

fn main() {
    let home = IpAddrKind::V4(String::from("127.0.0.1"));
    let loopback = IpAddrKind::V6(String::from("::1"));
    println!("Home: {:?}", home);
    println!("Loopback: {:?}", loopback);
}
//输出
//Home: V4("127.0.0.1")
//Loopback: V6("::1")

#[derive(Debug)]
enum IpAddrKind {
    V4(String),
    V6(String),
}
```

枚举替代结构体还有另一个优势：每个变体可以处理**不同类型和数量**的数据 (**你java做的到**？)

```rust
    enum IpAddr {
        V4(u8, u8, u8, u8),
        V6(String),
    }

    let home = IpAddr::V4(127, 0, 0, 1);

    let loopback = IpAddr::V6(String::from("::1"));
```

这些代码展示了使用枚举来存储两种不同 IP 地址的几种可能的选择。然而，事实证明存储和编码 IP 地址实在是太常见了[以致标准库提供了一个开箱即用的定义！](https://doc.rust-lang.org/std/net/enum.IpAddr.html)让我们看看标准库是如何定义 `IpAddr` 的：它正有着跟我们定义和使用的一样的枚举和变体，不过它将变体中的地址数据嵌入到了两个不同形式的结构体中，它们对不同的变体的定义是不同的：

```rust
struct Ipv4Addr {
    // --snip--
}

struct Ipv6Addr {
    // --snip--
}

enum IpAddr {
    V4(Ipv4Addr),
    V6(Ipv6Addr),
}
```

这些代码展示了可以将任意类型的数据放入枚举变体中：例如字符串、数字类型或者结构体。甚至可以包含另一个枚举！另外，标准库中的类型通常并不比你设想出来的要复杂多少。

**枚举也可以impl**



### option枚举

这一部分会分析一个 `Option` 的案例，`Option` 是标准库定义的另一个枚举。`Option` 类型应用广泛因为它编码了一个非常普遍的场景，即一个值要么有值要么没值。

例如，如果请求一个非空列表的第一项，会得到一个值，如果请求一个空的列表，就什么也不会得到。

从类型系统的角度来表达这个概念就意味着编译器需要**检查是否处理了所有应该处理**的情况，这样就可以避免在其他编程语言中非常常见的 bug。

Rust 并**没有**很多其他语言中有的空值功能。**空值**（*Null* ）是一个值，它代表没有值。在有空值的语言中，变量总是这两种状态之一：空值和非空值。

空值的问题在于当你尝试像一个非空值那样使用一个空值，会出现某种形式的错误。因为空和非空的属性无处不在，非常容易出现这类错误。

然而，空值尝试表达的概念仍然是有意义的：**空值是一个因为某种原因目前无效或缺失的值**。

问题不在于概念而在于具体的实现。为此，Rust 并没有空值，不过它确实拥有一个可以编码存在或不存在概念的枚举。这个枚举是 `Option<T>`而且它[定义于标准库中](https://doc.rust-lang.org/std/option/enum.Option.html)，如下：

```rust
enum Option<T> {
    None,
    Some(T),
}
```

`Option<T>` 枚举是如此有用以至于它甚至被包含在了 prelude 之中，无需将其显式引入作用域。

另外，它的变体也是如此：可以不需要 `Option::` 前缀来直接使用 `Some` 和 `None`。

即便如此 `Option<T>` 也仍是常规的枚举，`Some(T)` 和 `None` 仍是 `Option<T>` 的变体。

`<T>` 语法是一个我们还未讲到的 Rust 功能。它是一个泛型类型参数，第十章会更详细的讲解泛型。目前，所有你需要知道的就是 `<T>` 意味着 `Option` 枚举的 `Some` 变体可以包含任意类型的数据，同时每一个用于 `T` 位置的具体类型使得 `Option<T>` 整体作为不同的类型。

```rust
    let some_number = Some(5); //不用::就直接使用了
    let some_char = Some('e');

    let absent_number: Option<i32> = None; //需要明确指定是Option
```

`some_number` 的类型是 `Option<i32>`。`some_char` 的类型是 `Option<char>`，是不同于 `some_number` 的类型。

因为我们在 `Some` 变体中指定了值，Rust 可以推断其类型。对于 `absent_number`，Rust 需要我们**指定** `Option` 整体的类型，因为编译器只通过 `None` 值**无法推断**出 `Some` 变体保存的值的类型。

当有一个 `Some` 值时，我们就知道存在一个值，而这个值保存在 `Some` 中。当有个 `None` 值时，在某种意义上，它跟空值具有相同的意义：并没有一个有效的值。那么，`Option<T>` 为什么就比空值要好呢？

因为 `Option<T>` 和 `T`（这里的 `T` 可以是任何类型）是不同的类型，所以编译器不允许我们把 `Option<T>` 当成一个肯定有效的值来使用。(指定了T，可以把none当作某个类型使用)

例如，这段代码不能编译，因为它试图把 Option<i8> 和 i8 相加：

```rust
    let x: i8 = 5;
    let y: Option<i8> = Some(5);

    let sum = x + y;
```

Rust 不知道该如何把 `Option<i8>` 和 `i8` 相加，因为它们是不同的类型。当我们在 Rust 中拥有一个像 `i8` 这样的值时，编译器会确保它总是有效的。

我们可以放心使用它，而无需先做空值检查。只有当我们使用 `Option<i8>`（或者任何别的 `Option<T>`）时，才需要考虑值可能不存在，而编译器会确保我们在使用这个值之前处理了这种情况。

在对 `Option<T>` 进行运算之前必须将其转换为 `T`。通常这能帮助我们捕获到空值最常见的问题之一：假设某值不为空但实际上为空的情况。

消除了错误地假设一个非空值的风险，会让你对代码更加有信心。为了拥有一个可能为空的值，你必须要显式的将其放入对应类型的 `Option<T>` 中。

当使用这个值时，**必须明确的处理值为空的情况**。

只要一个值不是 `Option<T>` 类型，你就**可以**安全的认定它的值不为空。

看官方文档就知道怎么从option取了

[option的文档](https://doc.rust-lang.org/std/option/enum.Option.html)
