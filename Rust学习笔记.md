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
