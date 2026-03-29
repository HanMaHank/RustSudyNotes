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



### 标量类型

​	**标量**（*scalar*）类型代表一个单独的值。Rust中有四种基本类型整型、浮点型、布尔类型和字符类型。

#### 整型

| 长度     | 有符号  | 无符号  |
| -------- | ------- | ------- |
| 8-bit    | `i8`    | `u8`    |
| 16-bit   | `i16`   | `u16`   |
| 32-bit   | `i32`   | `u32`   |
| 64-bit   | `i64`   | `u64`   |
| 128-bit  | `i128`  | `u128`  |
| 架构相关 | `isize` | `usize` |

i8 也就是从 -128 到 127 。

u8也就是从 0 到 255。

另外，`isize` 和 `usize` 类型依赖运行程序的计算机架构：64 位架构上它们是 64 位的，32 位架构上它们是 32 位的。

数字后面加上类型后缀，来显示指定数值类型如 57u8。

支持使用_来进行视觉分割 如1000等价于1_000。

Rust 的默认类型是i32。。而 `isize` 和 `usize` 主要用在对某种集合进行索引的场景中。

一般编译会检查整型溢出，发行的优化编译不会检查，如果发生溢出，会回到1。

`256` 会变成 `0`，`257` 会变成 `1`，依此类推

为了显式地处理溢出的可能性，可以使用这几类标准库提供的原始数字类型方法：

- 所有模式下都可以使用 `wrapping_*` 方法进行 wrapping，如 `wrapping_add`
- 如果 `checked_*` 方法发生溢出，则返回 `None` 值
- 用 `overflowing_*` 方法返回值和一个布尔值，表示是否出现溢出

#### 浮点

ust 的浮点数类型是 `f32` 和 `f64`，分别占 32 位和 64 位。默认类型是 `f64`，因为在现代 CPU 中，它与 `f32` 速度几乎一样，不过精度更高。所有的浮点型都是有符号的。按照 IEEE-754 标准

```rust
fn main() {
    // addition
    let sum = 5 + 10;

    // subtraction
    let difference = 95.5 - 4.3;

    // multiplication
    let product = 4 * 30;

    // division
    let quotient = 56.7 / 32.2;
    let truncated = -5 / 3; // 结果为 -1

    // remainder
    let remainder = 43 % 5;
}
```

#### 布尔类型

布尔类型有两个可能的值：`true` 和 `false`

```rust
fn main() {
    let t = true;

    let f: bool = false; // with explicit type annotation
}
```

#### 字符类型

```rust
fn main() {
    let c = 'z';
    let z: char = 'ℤ'; // with explicit type annotation
    let heart_eyed_cat = '😻';
}
```

单字符用单引号，字符串字面量用双引号

#### 复合类型

**复合类型**（*compound types*）可以把多个值组合成一个类型。Rust 有两种原生的复合类型：元组（tuple）和数组（array）。

#### 元组

元组是一种将多个不同类型的值组合成一个复合类型的通用方式。元组长度固定：一旦声明，它的大小就不能增长或缩小。

```	rust
fn main() {
    let tup: (i32, f64, u8) = (500, 6.4, 1);
}
```

使用点号（`.`）后跟值的索引来直接访问所需的元组元素

不带任何值的元组有一个特殊名字，叫做 **单元（unit）**。这种值以及其对应的类型都写作 `()`，表示空值或空的返回类型。如果一个表达式没有返回任何其他值，它就会隐式返回单元值。

#### 数组类型(Rust 的数组长度是固定的)

和元组不同，数组中的每个元素都必须具有相同类型。

数组不如 vector 类型灵活。vector 是标准库提供的一种类似数组的集合类型，它 **允许** 长度增长或缩小。

如果你不确定该用数组还是 vector，那么很可能你应该用 vector。

当你明确知道元素个数不会变化时，数组就更有用。

程序中使用月份名称。

```rust
let months = ["January", "February", "March", "April", "May", "June", "July",
              "August", "September", "October", "November", "December"];
```

可以像这样编写数组的类型：在方括号中包含每个元素的类型，后跟分号，再后跟数组元素的数量。

```rust
let a: [i32; 5] = [1, 2, 3, 4, 5];
```

```rust
let a = [3; 5]; //这种写法与 let a = [3, 3, 3, 3, 3]; 效果相同，但更简洁
```

数组是在栈（stack）上分配的一整块、大小已知且固定的内存。

```rust
fn main() {
    let a = [1, 2, 3, 4, 5];

    let first = a[0]; //这样去访问数组
    let second = a[1];
}
```

显式的数组越界可以阻止，但是无法推断的越界不能阻止如用户输入之类的。

-----

## 函数

函数在 Rust 代码中非常普遍。你已经见过语言中最重要的函数之一：`main` 函数，它是很多程序的入口点。你也见过 `fn` 关键字，它用来声明新函数。

```rust
fn main() {
    who();
    println!("无能，丧权");
}
fn who() {
    println!("公司，垃圾")
}
```

### 参数

```rust
fn main() {
    who(259);
    println!("无能，丧权");
}
fn who(num: i32) {
    println!("公司，垃圾:{}", num);
}

fn main() {
    who(259, "你是人类啊");
    println!("无能，丧权");
}
fn who(num: i32, str: &str) {
    println!("公司，垃圾:{},{}", num, str);
}
```

### 语句和表达式

Rust 是一门基于表达式（expression-based）的语言

- **语句**（*Statements*）是执行一些操作但不返回值的指令。
- **表达式**（*Expressions*）计算并产生一个值。

#### 语句

```rust
let num = 12; //使用 let 关键字创建变量并绑定一个值是一个语句
```

语句不返回值。因此，不能把 `let` 语句赋值给另一个变量

下面的列子是错误的

```rust
fn main() {
    let x = (let y = 6); //let y = 6 这条语句不会返回值，因此没有什么东西可以绑定到 x 上。
}
```

函数定义本身也是语句,但调用函数不是。

#### 表达式

表达式会计算出一个值，并且你将编写的大部分 Rust 代码是由表达式组成的。

```rust
let num = 12; //12就是一个表达式，函数调用是一个表达式。宏调用是一个表达式。
//用大括号创建的一个新的块作用域也是一个表达式
    let y = {
        let x = 3;
        x + 1 // 计算产生一个x+1的值，最后一行表达式必须省略分号，让它成为块的返回值。
    };
/*块内最后一行代码不加分号：该行的计算结果就是整个块的返回值；
块内最后一行代码加分号：该行变成 “语句”（Statement），语句没有返回值，整个块会返回 ()（空元组，Rust 中的 “无值” 标识）。*/
```

加上分号就是语句了，不加就会有返回值

#### 具有返回值的函数

函数可以把值返回给调用它的代码。

我们不会给返回值命名，但必须在箭头（`->`）后面声明它的类型。

函数的返回值等同于函数体中最后一个表达式的值。

也可以使用 `return` 关键字并指定一个值，从函数中提前返回；

不过大多数函数都会隐式返回最后一个表达式的值。

```rust
fn main() {
    println!("{}", shit());
}

fn shit() -> i32 {
    let x = 5;
    let x = x + 1;
    x //等价写法 return x;
}
```

## 注释

// 和 /**/

## 控制流

### if语句

```rust
fn main() {
    let number = 6;

    if number % 4 == 0 {
        println!("number is divisible by 4");
    } else if number % 3 == 0 {
        println!("number is divisible by 3");
    } else if number % 2 == 0 {
        println!("number is divisible by 2");
    } else {
        println!("number is not divisible by 4, 3, or 2");
    }
}
```

条件**必须**是 `bool` 值。如果条件不是 `bool`，我们就会得到一个错误。

```rust
 let number = if condition { 5 } else { 6 }; //表达式
```

如果类型不一致，会报错。

### 循环loop、while、for

#### loop

```rust
fn main() {
    loop {
        println!("again!");
    }
}
```

一直执行除非写了break或控制台ctrl+c

可以从循环中返回值

```rust
fn main() {
    let mut counter = 0;

    let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2; //这个会返回
        }
    };

    println!("The result is {result}");
}
```

如果你在循环内部使用 `return`，也可以从中返回。不过，`break` 只会退出当前循环，而 `return` 总是会退出当前函数。

return 不是“跳出循环”，而是“直接返回当前函数”。

counter * 2这样写不支持

  - 不是“循环完全不支持”，而是 loop 只支持通过 break 值 把值返回到外层。
  - 你在循环体里单独写 counter * 2，这个值不会自动成为 loop 的结果。
  - return 则是直接返回函数，不是返回循环。

  所以本质是：在 Rust 里，循环要“产出值”必须显式 break value。

#### 循环标签

如果循环中又套了循环，那么 `break` 和 `continue` 默认只作用于当前最内层的那个循环。

加上循环标签，就可以跳出到指定循环

```rust
fn main() {
    let mut count = 0;
    'counting_up: loop {
        println!("count = {count}");
        let mut remaining = 10;

        loop {
            println!("remaining = {remaining}");
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
    println!("End count = {count}");
}
```

#### while条件循环

```rust
fn main() {
    let mut number = 3;

    while number != 0 {
        println!("{number}!");

        number -= 1;
    }

    println!("LIFTOFF!!!");
}
```

#### for循环遍历集合

```rust
fn main() {
    let a = [10, 20, 30, 40, 50];

    for element in a {
        println!("the value is: {element}");
    }
}
```

```rust
fn main() {
    for number in (1..4).rev() {
        println!("{number}!");
    }
    println!("LIFTOFF!!!");
}
```

## 所有权

**所有权**（*ownership*）是 Rust 用于如何管理内存的一组规则。

通过所有权系统管理内存，编译器在编译时会根据一系列的规则进行检查。如果违反了任何这些规则，程序都不能编译。

### 堆栈

#### 1. 栈

- **大小固定、编译期就确定**
- 存的必须是**已知大小、固定大小**的数据
- 分配**极快**（只需要移动栈顶指针）
- 访问**极快**（就在栈顶，连续内存，无需跳转）
- 栈上可以存**指向堆的指针**（指针大小固定）

#### 2. 堆

- **大小不固定、运行时才能确定**
- 可以动态扩容
- 分配**慢**（要找空闲块、要标记、要管理）
- 访问**慢**（必须通过栈上的指针**间接访问**）
- 内存不连续，CPU 缓存命中率低

### 所有权规则

1. Rust 中的每一个值都有一个 **所有者**（*owner*）。
2. 值在任一时刻有且只有一个所有者。
3. 当所有者离开作用域，这个值将被丢弃。

### String类型

字符串字面值是很方便的，不过它们并不适合使用文本的每一种场景。原因之一就是它们是不可变的。另一个原因是并非所有字符串的值都能在编写代码时就知道。(mut那个是改变了指针的指向，指向了新的变量，改变的不是原来的字符串字面量)

```rust
fn main() {
    // 这两个冒号 :: 是运算符，允许将特定的 from 函数置于 String 类型的命名空间（namespace）下，而不需要使用类似 string_from 这样的名字
    let str = String::from("Hello, world!");
    println!("{}", str);
}
```

不加 mut

栈上的值**不可变**

你**不能修改指针**

你**不能修改长度**

你**不能修改容量**

```rust
fn main() {
    let mut str = String::from("Hello, world!"); //
    str.push_str(", world!");
    println!("{}", str);
}
```

### 内存与分配

对于 `String` 类型，为了支持一个可变，可增长的文本片段，需要在堆上分配一块在编译时未知大小的内存来存放内容。这意味着：

- 必须在运行时向内存分配器（memory allocator）请求内存。
- 需要一个当我们处理完 `String` 时将内存返回给分配器的方法。

第一部分由我们完成：当调用 `String::from` 时，它的实现 (*implementation*) 请求其所需的内存。这在编程语言中是非常通用的。

第二部分实现起来就各有区别了。在有 **垃圾回收**（*garbage collector*，*GC*）的语言中，GC 记录并清除不再使用的内存

Rust 采取了一个不同的策略：内存在拥有它的变量离开作用域后就被自动释放。

```rust
    {
        let s = String::from("hello"); // 从此处起，s 是有效的

        // 使用 s
    }                                  // 此作用域已结束，
                                       // s 不再有效
```

这是一个将 `String` 需要的内存返回给分配器的很自然的位置：当 `s` 离开作用域的时候。当变量离开作用域，Rust 为我们调用一个特殊的函数。这个函数叫做 drop，在这里 `String` 的作者可以放置释放内存的代码。Rust 在结尾的 `}` 处自动调用 `drop`。

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



## 结构体

**结构体**（*struct*），或者 *structure*，是一个自定义数据类型，允许你包装和命名多个相关的值

### 定义和实例化

在大括号中，定义每一部分数据的名字和类型，我们称为 **字段**（*field*）。

```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}
```

创建一个实例需要以结构体的名字开头，接着在大括号中使用 `key: value` 键 - 值对的形式提供字段，其中 key 是字段的名字，value 是需要存储在字段中的数据值。

```rust
fn main() {
    let user1 = User {
        active: true,
        username: String::from("someusername123"),
        email: String::from("someone@example.com"),
        sign_in_count: 1,
    };
}
```

如果结构体的实例是可变的，我们可以使用点号并为对应的字段赋值。(加mut)

```rust
fn main() {
    let mut user1 = User {
        active: true,
        username: String::from("someusername123"),
        email: String::from("someone@example.com"),
        sign_in_count: 1,
    };

    user1.email = String::from("anotheremail@example.com");
}
```

### 字段初始化简化写法

参数名与字段名都完全相同，我们可以使用 **字段初始化简写语法**（*field init shorthand*）来重写 `build_user`，这样其行为与之前完全相同，不过无需重复 `username` 和 `email` 了

```rust
fn build_user(email: String, username: String) -> User { 
    //参数名称和结构体内的字段名称相同可以简化赋值
    User {
        username,
        email,
        sign_in_count: 1,
    }
}

```

### 使用结构体更新语法创建实例

```rust
使用旧实例的大部分值但改变其部分值来创建一个新的结构体实例通常是很有用的。这可以通过 结构体更新语法（struct update syntax）实现。
```

```rust
fn main() {
    // --snip--

    let user2 = User {
        email: String::from("another@example.com"),
        ..user1 //..user1 必须放在最后
    };
}
//等价写法
fn main() {
    // --snip--

    let user2 = User {
        active: user1.active,
        username: user1.username,
        email: String::from("another@example.com"),
        sign_in_count: user1.sign_in_count,
    };
}

```

### 使用元组结构体创建不同的类型

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

fn main() {
    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);
}
```

元组结构体实例类似于元组，你可以将它们解构为单独的部分，也可以使用 `.` 后跟索引来访问单独的值。与元组不同的是，解构元组结构体时必须写明结构体的类型。例如，我们可以写 `let Point(x, y, z) = origin;`，将 `origin` 的值解构到名为 `x`、`y` 和 `z` 的变量中。

### 类单元结构体

没有任何字段的结构体被称为 **类单元结构体**（*unit-like structs*）

它们的行为类似于 `()`

类单元结构体在你想要在某个类型上实现 trait，但又不需要在该类型本身中存储任何数据时会很有用。

```rust
struct AlwaysEqual;

fn main() {
    let subject = AlwaysEqual;
}
```

----



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

---

### 模式匹配

 Rust 有一个叫做 `match` 的极为强大的控制流运算符，它允许我们将一个值与一系列的模式相比较，并根据相匹配的模式执行相应代码。

`match` 表达式想象成某种硬币分类器：硬币滑入有着不同大小孔洞的轨道，每一个硬币都会掉入符合它大小的孔洞。

```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

在这个例子中是 `coin` 的值。这看起来非常像 `if` 所使用的条件表达式，不过这里有一个非常大的区别：对于 `if`，表达式必须返回一个布尔值，而这里它可以是任何类型的。

### 绑定值的匹配模式

匹配分支的另一个有用的功能是可以绑定匹配的模式的部分值。 (就是匹配模式里放匹配模式)

```rust
fn main() {
    println!("Hello, world! {}",outPrintStats(Coin::Quarter(UsState::Alabama)));
}

fn outPrintStats(coin:Coin) -> u8{
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {state:?}!");
            25
        }
    }
}

#[derive(Debug)] // 这样可以立刻看到州的名称
enum UsState {
    Alabama,
    Alaska,
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}
```

### 匹配Option<T>

none的时候做什么，非none的时候做什么

```rust
    fn plus_one(x: Option<i32>) -> Option<i32> {
        match x {
            None => None,
            Some(i) => Some(i + 1),
        }
    }

    let five = Some(5);
    let six = plus_one(five);
    let none = plus_one(None);
```

这在一开始有点复杂，不过一旦习惯了，你会希望所有语言都拥有它！这一直是用户的最爱。

**`match` 还有另一方面需要讨论：这些分支必须覆盖了所有的可能性。**

### 通配符和_占位符

使用枚举，我们也可以针对少数几个特定值执行特殊操作，而对其他所有值采取默认操作。

```rust
    let dice_roll = 9;
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        _ => reroll(),  //_这个可以用other代替，效果一样，但是other叫通配，_告诉rust我们不会使用这个值
    }
		
    fn add_fancy_hat() {}
    fn remove_fancy_hat() {}
    fn reroll() {}
```

other是通配模式

当我们不想使用通配模式获取的值时，请使用 `_` ，这是一个特殊的模式，可以匹配任意值而不绑定到该值。

这告诉 Rust 我们不会使用这个值，所以 Rust 也不会警告我们存在未使用的变量。

```rust
    let dice_roll = 9;
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        other => move_player(other),
    }

    fn add_fancy_hat() {}
    fn remove_fancy_hat() {}
    fn move_player(num_spaces: u8) {}
```

### if let和let else简控制流

`if let` 语法让我们以一种不那么冗长的方式结合 `if` 和 `let`，来处理只匹配一个模式的值而忽略其他模式的情况。

```rust
    let config_max = Some(3u8);
    match config_max {
        Some(max) => println!("The maximum is configured to be {max}"),
        _ => (),
    } 
```

rust也差不多有

```rust
    let config_max = Some(3u8);
    if let Some(max) = config_max { //判断有值就输出
        println!("The maximum is configured to be {max}");
    }//想一下go语言，是不是可以直接赋值？ 赋值+判断
```

**模式匹配判断**：检查 `config_max` 是不是匹配 `Some(_)`；

**匹配成功才解构赋值**：如果是 `Some`，才把里面的值绑给 `max`；

**进入代码块执行**；

如果是 `None` → 匹配失败，直接不执行里面代码

使用 `if let` 意味着更少的输入、更少的缩进，也更少的样板代码。

这样也会失去 `match` 所强制的穷尽性检查，也就无法确保你没有遗漏某些情况。

可以认为 `if let` 是 `match` 的一个语法糖，它当值匹配某一模式时执行代码而忽略所有其他值。

可以在 `if let` 中包含一个 `else`

`else` 块中的代码与 `match` 表达式中的 `_` 分支块中的代码相同

```rust
    let mut count = 0;
    match coin {
        Coin::Quarter(state) => println!("State quarter from {state:?}!"),
        _ => count += 1,
    }
```

```rust
    let mut count = 0;
    if let Coin::Quarter(state) = coin {
        println!("State quarter from {state:?}!");
    } else {
        count += 1;
    }
```

### 使用 `let...else` 来保持在 “愉快路径”（“Happy Path”）

一个常见的场景是：如果某个值存在，就对它做一些操作；如果不存在，就返回一个默认值。

枚举变体里藏着数据，你想把数据取出来用。普通枚举变体没数据就不需要解构。

```rust
fn describe_state_quarter(coin: Coin) -> Option<String> {
  let Coin::Quarter(state) = coin else {
      return None;  // 匹配失败就执行这里
  };
  // 匹配成功，state 在这里可用
    if state.existed_in(1900) {
        Some(format!("{state:?} is pretty old, for America!"))
    } else {
        Some(format!("{state:?} is relatively new."))
    }
}
```

  逻辑是：
  - 如果 coin 是 Coin::Quarter(state)，就把 state 解构出来继续往下走
  - 如果不是，就执行 else 块提前返回

等价下面

```rust
fn describe_state_quarter(coin: Coin) -> Option<String> {
    let state = if let Coin::Quarter(state) = coin {
        state
    } else {
        return None;
    };

    if state.existed_in(1900) {
        Some(format!("{state:?} is pretty old, for America!"))
    } else {
        Some(format!("{state:?} is relatively new."))
    }
}
```

## 包、Crates与模块

- **包**（*Packages*）：Cargo 的一个功能，它允许你构建、测试和分享 crate。
- **Crates**：一个模块树，可以产生一个库或可执行文件。
- **模块**（*Modules*）和 **use**：允许你控制作用域和路径的私有性。
- **路径**（*path*）：一个为例如结构体、函数或模块等项命名的方式。

### 包和crates

crate 是 Rust 编译器每次处理的最小代码单位。

即使你用 `rustc` 而不是 `cargo` 来编译单个源代码文件，编译器也会把那个文件视为一个 crate。

crate 有两种形式：二进制 crate 和库 crate。**二进制 crate**（*Binary crates*）可以被编译为可执行程序，比如命令行程序或者服务端。它们必须有一个名为 `main` 函数来定义当程序被执行的时候所需要做的事情。

目前我们所创建的 crate 都是二进制 crate。

**库 crate**（*Library crates*）并没有 `main` 函数，它们也不会编译为可执行程序。

相反它们定义了可供多个项目复用的功能模块。

大多数时间 `Rustaceans` 说的 “crate” 指的都是**库 crate**，这与其他编程语言中 “library” 概念一致。

*包*（*package*）是提供一系列功能的一个或者多个 crate 的捆绑。一个包会包含一个 *Cargo.toml* 文件，阐述如何去构建这些 crate。

Cargo 实际上就是一个包，它包含了用于构建你代码的命令行工具的二进制 crate。其他项目也依赖 Cargo 库来实现与 Cargo 命令行程序一样的逻辑。

让我们来看看创建包的时候会发生什么。首先，我们输入命令 `cargo new my-project`：

```shell
$ cargo new my-project
     Created binary (application) `my-project` package
$ ls my-project
Cargo.toml
src
$ ls my-project/src
main.rs
```

Cargo 会给我们的包创建一个 ***Cargo.toml*** 文件。查看 *Cargo.toml* 的内容，会发现并没有提到 *src/main.rs*，因为 Cargo 遵循的一个**约定**：***src/main.rs* 就是一个与包同名的二进制 crate 的 crate 根**。同样的，**Cargo 知道如果包目录中包含 *src/lib.rs*，则包带有与其同名的库 crate，且 *src/lib.rs* 是 crate 根**。crate 根文件将由 Cargo 传递给 `rustc` 来实际构建库或者二进制项目。

```bash
  src/main.rs 是二进制程序的入口（编译成可执行文件）

  src/lib.rs 是库的根，不是"加入外部包"的地方，而是你自己写的库代码的入口

  外部包（依赖）是在 Cargo.toml 里声明的：

  [dependencies]
  serde = "1.0"

  然后在任何 .rs 文件里用 use 引入：

  use serde::Serialize;

  ---
  src/lib.rs 的用途是：当你的包既有可执行程序又有库时，lib.rs 放可复用的逻辑，main.rs
  调用它。或者你的包就是纯库，给别人用的。
```

#### 定义模块来控制作用域与私有性

允许你命名项的 *路径*（*paths*）；用来将路径引入作用域的 `use` 关键字；以及使项变为公有的 `pub` 关键字。我们还将讨论 `as` 关键字、外部包（external packages）和 glob 运算符（glob operator）。

#### 模块小抄

这里提供一个简单的参考，用来解释模块、路径、`use`关键词和`pub`关键词如何在编译器中工作，以及大部分开发者如何组织他们的代码。

- **从 crate 根节点开始**: 当编译一个 crate, 编译器首先在 crate 根文件（通常，对于一个库 crate 而言是 *src/lib.rs*，对于一个二进制 crate 而言是 *src/main.rs*）中寻找需要被编译的代码。
- **声明模块**: 在 crate 根文件中，你可以声明一个新模块；比如，用 `mod garden;` 声明了一个叫做 `garden` 的模块。编译器会在下列路径中寻找模块代码：
  - 内联，用大括号替换 `mod garden` 后跟的分号
  - 在文件 *src/garden.rs*
  - 在文件 *src/garden/mod.rs*
- **声明子模块**: 在除了 crate 根节点以外的任何文件中，你可以定义子模块。比如，你可能在 *src/garden.rs* 中声明 `mod vegetables;`。编译器会在以父模块命名的目录中寻找子模块代码：
  - 内联，直接在 `mod vegetables` 后方不是一个分号而是一个大括号
  - 在文件 *src/garden/vegetables.rs*
  - 在文件 *src/garden/vegetables/mod.rs*
- **模块中的代码路径**: 一旦一个模块是你 crate 的一部分，你可以在隐私规则允许的前提下，从同一个 crate 内的任意地方，通过代码路径引用该模块的代码。举例而言，一个 garden vegetables 模块下的 `Asparagus` 类型可以通过 `crate::garden::vegetables::Asparagus` 访问。
- **私有 vs 公用**: 一个模块里的代码默认对其父模块私有。为了使一个模块公用，应当在声明时使用 `pub mod` 替代 `mod`。为了使一个公用模块内部的成员公用，应当在声明前使用`pub`。
- **`use` 关键字**: 在一个作用域内，`use`关键字创建了一个项的快捷方式，用来减少长路径的重复。在任何可以引用 `crate::garden::vegetables::Asparagus` 的作用域，你可以通过 `use crate::garden::vegetables::Asparagus;` 创建一个快捷方式，然后你就可以在作用域中只写 `Asparagus` 来使用该类型。

这里我们创建一个名为`backyard`的二进制 crate 来说明这些规则。该 crate 的路径同样命名为`backyard`，该路径包含了这些文件和目录：

```bash
backyard
├── Cargo.lock
├── Cargo.toml
└── src
    ├── garden
    │   └── vegetables.rs
    ├── garden.rs
    └── main.rs
```

这个例子中的 crate 根文件是 *src/main.rs*，该文件包含了：

```rust
use crate::garden::vegetables::Asparagus;

pub mod garden;

fn main() {
    let plant = Asparagus {};
    println!("I'm growing {plant:?}!");
}
```

`pub mod garden;` 行告诉编译器将 *src/garden.rs* 中发现的代码包含进来：

文件名：src/garden.rs

```rust
pub mod vegetables;
```

在此处，`pub mod vegetables;` 意味着在 *src/garden/vegetables.rs* 中的代码也应该被包含。这些代码是：

```rust
#[derive(Debug)]
pub struct Asparagus {}
```

#### 在模块中对相关代码分组

**模块**让我们可以将一个 crate 中的代码进行分组，以提高可读性与重用性。因为一个模块中的代码默认是私有的，所以还可以利用模块控制项的**私有性**（*privacy*）。**私有项是不可为外部使用的内在详细实现**。我们也可以将**模块和它其中的项标记为公开的**，这样，外部代码就可以使用并依赖于它们。

```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}

        fn seat_at_table() {}
    }

    mod serving {
        fn take_order() {}

        fn serve_order() {}

        fn take_payment() {}
    }
}
```

  mod 就是命名空间，用来组织代码，和文件系统目录一个道理：

  crate          ← 根（src/lib.rs 或 src/main.rs）
  └── front_of_house
      ├── hosting
      └── serving

  几个关键点：

  - mod 可以嵌套，形成树状结构
  - 整棵树的根隐式叫 crate
  - 模块里可以放函数、结构体、枚举、常量、其他模块

### 引用模块树中项的路径

为了向 Rust 指示在模块树中从何处查找某个项，我们使用路径，就像在文件系统中使用路径一样。

路径有两种形式：

- **绝对路径**（*absolute path*）是以 crate 根（root）开头的完整路径；对于外部 crate 的代码，是以 crate 名开头的绝对路径，对于当前 crate 的代码，则以字面值 `crate` 开头。
- **相对路径**（*relative path*）从当前模块开始，以 `self`、`super` 或当前模块中的某个标识符开头。

绝对路径和相对路径都后跟一个或多个由双冒号（`::`）分割的标识符。

希望调用 `add_to_waitlist` 函数。这相当于在问：`add_to_waitlist` 函数的路径是什么？

同级的没有公私有问题

文件名：src/lib.rs

```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // 绝对路径
    crate::front_of_house::hosting::add_to_waitlist();

    // 相对路径
    front_of_house::hosting::add_to_waitlist();
}
```

以上代码编译失败，因为不是pub

```rust
mod front_of_house {
    pub mod hosting {
        fn add_to_waitlist() {}
    }
}
```

然后模块公有，不代表函数公有

子模块对父模块是私有的，但同级或父级代码可以访问子模块的私有内容

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}
```

#### super开始的相对路径

可以通过在路径的开头使用 `super` ，**从父模块开始构建相对路径**，而不是从当前模块或者 crate 根开始。这类似以 `..` 语法开始一个文件系统路径。使用 `super` 允许我们引用父模块中的已知项，这使得当模块与父模块关联的很紧密，但某天父模块可能要移动到模块树的其它位置时重新组织模块树变得更容易。

```rust
fn deliver_order() {}

mod back_of_house {
    fn fix_incorrect_order() {
        cook_order();
        super::deliver_order();
    }

    fn cook_order() {}
}
```

#### 创建公有的结构体和枚举

我们还可以使用 `pub` 来设计公有的结构体和枚举，不过关于在结构体和枚举上使用 `pub` 还有一些额外的细节需要注意。如果我们在一个结构体定义的前面使用了 `pub`，这个结构体会变成公有的，但是这个**结构体的字段仍然是私有的**。我们可以根据情况决定每个字段是否公有。

```rust
mod back_of_house {
    pub struct Breakfast {
        pub toast: String,
        seasonal_fruit: String,
    }

    impl Breakfast {
        pub fn summer(toast: &str) -> Breakfast {
            Breakfast {
                toast: String::from(toast),
                seasonal_fruit: String::from("peaches"),
            }
        }
    }
}

pub fn eat_at_restaurant() {
    // 在夏天订购一个黑麦土司作为早餐
    let mut meal = back_of_house::Breakfast::summer("Rye");
    // 改变主意更换想要面包的类型
    meal.toast = String::from("Wheat");
    println!("I'd like {} toast please", meal.toast);

    // 如果取消下一行的注释代码不能编译；
    // 不允许查看或修改早餐附带的季节水果
    // meal.seasonal_fruit = String::from("blueberries");
}
```

```rust
mod back_of_house { //公有枚举
    pub enum Appetizer {
        Soup,
        Salad,
    }
}

pub fn eat_at_restaurant() {
    let order1 = back_of_house::Appetizer::Soup;
    let order2 = back_of_house::Appetizer::Salad;
}
```



## 使用Use关键字将路径引入作用域

不得不编写路径来调用函数显得繁琐且重复。

无论我们选择 `add_to_waitlist` 函数的绝对路径还是相对路径，每次我们想要调用 `add_to_waitlist` 时，都必须指定`front_of_house` 和 `hosting`

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting; //import差不多

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```

增加 `use` 和路径类似于在文件系统中创建软连接（符号连接，symbolic link）。通过在 crate 根增加 `use crate::front_of_house::hosting`，现在 `hosting` 在作用域中就是有效的名称了，如同 `hosting` 模块被定义于 crate 根一样。通过 `use` 引入作用域的路径也会检查私有性，同其它路径一样。

### 创建惯用的use路径

你可能会比较疑惑，为什么我们是指定 `use crate::front_of_house::hosting`，然后在 `eat_at_restaurant` 中调用 `hosting::add_to_waitlist` ，而不是通过指定一直到 `add_to_waitlist` 函数的 `use` 路径来得到相同的结果，就是说，为什么不直接定到函数，而是定位到mod。

因为可以清晰地表明函数不是在**本地定义**的，同时使完整路径的重复度最小化。

另一方面，使用 `use` 引入结构体、枚举和其他项时，习惯是指定它们的**完整路径**。

```rust
use std::collections::HashMap;

fn main() {
    let mut map = HashMap::new();
    map.insert(1, 2);
}
```

这种习惯用法背后没有什么硬性要求：它只是一种惯例，人们已经习惯了以这种方式阅读和编写 Rust 代码。

这个习惯用法有一个例外，那就是我们想使用 `use` 语句将两个具有相同名称的项带入作用域，因为 Rust 不允许这样做。

用这种用法还可以避免同名项冲突，因为可以通过父模块区分。

### as关键字

避免同名项冲突也可以使用as关键字

```rust
use std::fmt::Result;
use std::io::Result as IoResult; //使用as避免冲突

fn function1() -> Result {
    // --snip--
}

fn function2() -> IoResult<()> {
    // --snip--
}
```

### 使用pub use重导出名称

使用 `use` 关键字，将某个名称导入当前作用域后，该名称对此作用域之外还是私有的。

若要让作用域之外的代码能够像在当前作用域中一样使用该名称，可以将 `pub` 与 `use` 组合使用。

这种技术被称为**重导出**（*re-exporting*），因为在把某个项目导入当前作用域的同时，也将其暴露给其他作用域。

```rust
mod front_of_house { //模块是私有的
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub use crate::front_of_house::hosting; //但想导出这个模块 就用这种

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```

在这个修改之前，外部代码需要使用路径 `restaurant::front_of_house::hosting::add_to_waitlist()` 来调用 `add_to_waitlist` 函数，并且还需要将 `front_of_house` 模块标记为 `pub`。现在这个 `pub use` 从根模块重导出了 `hosting` 模块，外部代码现在可以使用路径 `restaurant::hosting::add_to_waitlist`。

### 使用外部包

我们编写了一个猜猜看游戏。那个项目使用了一个外部包 `rand` 来生成随机数。为了在项目中使用 `rand`，在 *Cargo.toml* 中加入了如下行：

```yaml
rand = "0.8.5"
```

在 *Cargo.toml* 中加入 `rand` 依赖告诉了 Cargo 要从 [crates.io](https://crates.io/) 下载 `rand` 和其依赖，并使其可在项目代码中使用。

为了将 `rand` 定义引入项目包的作用域，我们加入一行 `use` 起始的包名，它以 `rand` 包名开头并列出了需要引入作用域的项。我们曾将 `Rng` **trait** 引入作用域并调用了 `rand::thread_rng` 函数：

```rust
use rand::Rng;

fn main() {
    let secret_number = rand::thread_rng().gen_range(1..=100);
}
```

[crates.io](https://crates.io/) 上有很多 Rust 社区成员发布的包，将其引入你自己的项目都需要一道相同的步骤：在 *Cargo.toml* 列出它们并通过 `use` 将其中定义的项引入项目包的作用域中。

注意 `std` 标准库对于你的包来说也是外部 crate。因为标准库随 Rust 语言一同分发，无需修改 *Cargo.toml* 来引入 `std`，不过需要通过 `use` 将标准库中定义的项引入项目包的作用域中来引用它们

```rust
use std::collections::HashMap;
```

这是一个以标准库 crate 名 `std` 开头的绝对路径。

### 使用嵌套路径来清理大量的use列表

当需要引入很多定义于相同包或相同模块的项时，为每一项单独列出一行会占用源码大量的垂直空间。

例如猜猜看章节示例 2-4 中有两行 `use` 语句都从 `std` 引入项到作用域：

```rust
// --snip--
use std::cmp::Ordering;
use std::io;
// --snip--
```

相反，我们可以使用嵌套路径将相同的项在一行中引入作用域。这么做需要指定路径的相同部分，接着是两个冒号，接着是大括号中的各自不同的路径部分

```rust
// --snip--
use std::{cmp::Ordering, io};
// --snip--
```

**前缀相同，括号**

在较大的程序中，使用嵌套路径从相同包或模块中引入很多项，可以显著减少所需的独立 `use` 语句的数量！

```rust
use std::io;
use std::io::Write;

use std::io::{self, Write};
```

### 使用glob运算符导入项

如果希望将一个路径下**所有**公有项引入作用域，可以指定路径后跟 `*` glob 运算符:

```rust
use std::collections::*;
```

这个 `use` 语句将 `std::collections` 中定义的所有公有项引入当前作用域。使用 glob 运算符时请多加小心！Glob 会使得我们难以推导作用域中有什么名称和它们是在何处定义的。

glob 运算符经常用于测试模块 `tests` 中，这时会将所有内容引入作用域；

## 将模块拆分成多个文件

到目前为止，本章的所有示例都在**一个文件中定义了多个模块**。当模块变大时，你可能想把它们的定义移到单独的文件中，以便让代码更容易浏览。

就是利用pub use

文件一

```rust
pub mod front_of_house;

pub use crate::front_of_house::hosting; //库crate

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```

文件二

```rust
pub mod hosting {
    pub fn add_to_waitlist() {}
}
```

Rust 允许你把一个包拆分成多个 crate，再把一个 crate 拆分成多个模块，这样你就能在一个模块中引用另一个模块里定义的项。你既可以使用绝对路径，也可以使用相对路径。还可以用 `use` 语句把路径引入作用域，这样在同一作用域内多次使用该项时就能写更短的路径。模块代码默认是私有的，不过你可以加上 `pub` 关键字，把定义公开出去。

## 常见集合

- **向量**（*vector*）允许你把数量可变的值一个挨一个地存放起来。
- **字符串**（*string*）是字符的集合。此前我们已经提到过 `String` 类型，不过本章会更深入地讨论它。
- **哈希映射**（*hash map*）允许你把某个值与特定的键关联起来。它是更通用的数据结构 *map* 的一种具体实现。

### vector

第一种集合类型是 `Vec<T>`，也被称为 *vector*。vector 允许你在单个数据结构中存放多个值，并把这些值在内存中彼此相邻地排列起来。vector 只能**存储相同类型**的值。当你有一组项目要处理时，它就很有用，例如文件中的文本行，或者购物车中商品的价格。

**新建**

```rust
 let v: Vec<i32> = Vec::new();
```

我们加了一个**类型注解**。因为还没有往这个 vector 里插入任何值，Rust 并不知道我们打算存储什么类型的元素。这一点很重要。vector 是使用泛型实现的；

更常见的情况是，我们会用**初始值**创建 `Vec<T>`，而 Rust 会推断出你想存储的值的类型，所以很少需要写这种类型注解。Rust 还很贴心地提供了 **`vec!` 宏**，它会创建一个新的 vector，并把你提供的值放进去。

```rust
    let v = vec![1, 2, 3];
    let v = Vec::from([1, 2, 3, 4, 5]);
```

**添加**

```rust
    let mut v = Vec::new();

    v.push(5);
    v.push(6);
    v.push(7);
    v.push(8);
```

和任何变量一样，如果想修改它的值，就必须像第三章讲过的那样，使用 `mut` 关键字让它变成可变的。

**读**

```rust
    let v = vec![1, 2, 3, 4, 5];

    let third: &i32 = &v[2]; // 索引语法
    println!("The third element is {third}");

    let third: Option<&i32> = v.get(2); //get方法
    match third {
        Some(third) => println!("The third element is {third}"),
        None => println!("There is no third element."),
    }
```

Rust 提供这两种引用元素的方式，是为了让你可以选择：当尝试使用超出已有元素范围的索引值时，程序该如何表现。

假设我们有一个包含 5 个元素的 vector，然后尝试分别用这两种技术访问索引 100 处的元素，看看会发生什么。

```rust
    let v = vec![1, 2, 3, 4, 5];

    let does_not_exist = &v[100];
    let does_not_exist = v.get(100);
```

运行这段代码时，第一种 `[]` 方法会让程序 panic，因为它引用了一个不存在的元素。 直接崩溃。

当传给 `get` 方法的索引超出了 vector 的范围时，它不会 panic，而是返回 `None`。

当程序拿到了一个有效引用后，借用检查器会应用所有权和借用规则（第四章讲过），来确保这个对 vector 内容的引用以及其他任何引用都保持有效。

在同一作用域中，不能同时拥有可变引用和不可变引用。

我们持有了 vector 第一个元素的不可变引用，然后又尝试在 vector 末尾添加一个元素。

```rust
    let mut v = vec![1, 2, 3, 4, 5];

    let first = &v[0];

    v.push(6);

    println!("The first element is: {first}");
```

编译器直接报错。

那么为什么？！！

为什么对第一个元素的引用，会在乎 vector 末尾发生的变化呢？

这是由 vector 的工作方式决定的。因为 vector 会把值彼此相邻地存放在内存中，所以如果末尾追加一个新元素，而当前存放位置又没有足够空间容纳所有元素，**程序就可能需要分配一块新内存**，并把**旧元素复制**到新空间里去。

**复制！！！！请求内存，复制**



**遍历**

```rust
    let v = vec![100, 32, 57];
    for i in &v {
        println!("{i}");
    }
```



```rust
    let mut v = vec![100, 32, 57];
    for i in &mut v {
        *i += 50;
    }
```

#### 使用枚举来存储多种类型

```rust
    enum SpreadsheetCell {
        Int(i32),
        Float(f64),
        Text(String),
    }

    let row = vec![
        SpreadsheetCell::Int(3),
        SpreadsheetCell::Text(String::from("blue")),
        SpreadsheetCell::Float(10.12),
    ];
```

用结构体的话每个元素都得有所有字段。

如果在编写程序时，你并不知道运行时究竟会有哪些类型需要存进 vector，那么这种枚举技巧就不适用了。相反，你可以使用 trait 对象。

**丢弃vector时也会丢弃其所有元素**

```rust
    {
        let v = vec![1, 2, 3, 4];

        // 使用 v
    } // <- 在这里 v 离开作用域并被释放
```

### 使用字符串存储UTF-8文本

字符串是新晋 Rustacean 们通常会被困住的领域，这是由于三方面理由的结合：Rust 倾向于确保暴露出可能的错误，字符串是比很多程序员所想象的要更为复杂的数据结构，以及 UTF-8。所有这些要素结合起来对于来自其他语言背景的程序员就可能显得很困难了。

在集合章节中讨论字符串的原因是，字符串就是作为**字节的集合**外加一些方法实现的，当这些字节被解释为文本时，这些方法提供了实用的功能。

我们会讲到 `String` 中那些任何集合类型都有的操作，比如创建、更新和读取。也会讨论 `String` 与其他集合不一样的地方，例如索引 `String` 是很复杂的，由于人和计算机理解 `String` 数据方式的不同。

**定义字符串**

定义一下**字符串**这一术语的具体意义。Rust 的核心语言中只有一种字符串类型，字符串 slice `str`，它通常以被借用的形式出现，`&str`。

**字符串 slices**：它们是一些对储存在别处的 UTF-8 编码字符串数据的引用。

由于字符串字面值被储存在程序的二进制输出中，因此它们也是字符串 slices。

字符串（`String`）类型由 Rust 标准库提供，而不是编入核心语言，它是一种可增长、可变、可拥有、UTF-8 编码的字符串类型。

当 Rustaceans 提及 Rust 中的 “字符串 “时，他们可能指的是 `String` 或 string slice `&str` 类型，而不仅仅是其中一种类型。

虽然本节主要讨论 `String`，但这两种类型在 Rust 的标准库中都有大量使用，而且 `String` 和 字符串 slices 都是 UTF-8 编码的。

**新建字符串**

很多 `Vec<T>` 上可用的操作在 `String` 中同样可用，事实上 `String` 被实现为一个带有一些额外保证、限制和功能的字节 vector 的封装。

```rust
   let mut s = String::new();
```

这新建了一个叫做 `s` 的空的字符串，接着我们可以向其中加载数据。通常字符串会有初始数据，因为我们希望一开始就有这个字符串。为此，可以使用 **`to_string` 方法**，它能用于任何实现了 `Display` trait 的类型，比如字符串字面值。

```rust
fn main() {
    let data = "FuckMe";
    let mut s = data.to_string();
    s.push_str("\r\n");
    println!("{}", s);
}
```

也可以使用 `String::from` 函数来从字符串字面值创建 `String`。

```rust
let s = String::from("initial contents");
```

因为字符串应用广泛，这里有很多不同的用于字符串的通用 API 可供选择。其中一些可能看起来多余，不过都有其用武之地！在这个例子中，`String::from` 和 `.to_string` 最终做了完全相同的工作，所以如何选择就是代码风格与可读性的问题了。

记住字符串是 UTF-8 编码的，所以可以包含任何经过正确编码的数据

**更新字符串**

`String` 的大小可以增加，其内容也可以改变，就像可以放入更多数据来改变 `Vec` 的内容一样。另外，可以方便的使用 **`+` 运算符**或 `format!` 宏来拼接 `String` 值。

可以通过 `push_str` 方法来附加字符串 slice，从而使 `String` 变长

```rust
    let mut s = String::from("foo");
    s.push_str("bar");
```

`push` 方法被定义为获取**一个单独的字符**作为参数，并附加到 `String` 中。

```rust
    let mut s = String::from("lo");
    s.push('l');
```

```rust
   let s1 = String::from("Hello, ");
    let s2 = String::from("world!");
    let s3 = s1 + &s2; // 注意 s1 被移动了，不能继续使用
```

执行完这些代码之后，字符串 `s3` 将会包含 `Hello, world!`。`s1` 在相加后不再有效的原因，和使用 `s2` 的引用的原因，与使用 `+` 运算符时调用的函数签名有关。`+` 运算符使用了 `add` 函数，这个函数签名看起来像这样：

```rust
fn add(self, s: &str) -> String {
```

在标准库中你会发现，`add` 的定义使用了泛型和关联类型。在这里我们替换为了具体类型，这也正是当使用 `String` 值调用这个方法会发生的。

编译器可以把 `&String` 参数强制转换成 `&str`。

Rust 会使用一种叫做 **deref 强制转换**（*deref coercion*）的机制，这里会把 `&s2` 转换成 `&s2[..]`

可以发现签名中 `add` 获取了 `self` 的所有权，因为 `self` **没有**使用 `&`。

`s1` 的所有权将被移动到 `add` 调用中，之后就不再有效。所以虽然 `let s3 = s1 + &s2;` 看起来就像它会复制两个字符串并创建一个新的字符串，而**实际上这个语句会获取 `s1` 的所有权**，附加上从 `s2` 中拷贝的内容，并返回结果的所有权。换句话说，它看起来好像生成了很多拷贝，不过实际上并没有：**这个实现比拷贝要更高效**。

如果想要级联多个字符串，`+` 运算符的行为就显得笨重了：

```rust
    let s1 = String::from("tic");
    let s2 = String::from("tac");
    let s3 = String::from("toe");

    let s = s1 + "-" + &s2 + "-" + &s3;
```

这时 `s` 的内容会是 `tic-tac-toe`。在有这么多 `+` 和 `"` 字符的情况下，很难理解具体发生了什么。对于更为复杂的字符串链接，可以使用 `format!` 宏：

```rust
    let s1 = String::from("tic");
    let s2 = String::from("tac");
    let s3 = String::from("toe");

    let s = format!("{s1}-{s2}-{s3}"); //使用的是引用
```

这些代码也会将 `s` 设置为 `tic-tac-toe`。`format!` 与 `println!` 的工作原理相同，不过不同于将输出打印到屏幕上，它返回一个带有结果内容的 `String`。这个版本就好理解的多，宏 `format!` 生成的代码使用**引用**因此不会获取任何参数的所有权。

**一般用 format!**



Rust **不允许使用索引**获取 `String` 字符的原因是，索引操作预期总是需要常数时间（O(1)）。但是对于 `String` 不可能保证这样的性能，因为 Rust 必须从开头到索引位置遍历来确定有多少有效的字符。

#### 字符串slice

索引字符串通常是一个坏点子，因为字符串索引应该返回的类型是不明确的：字节值、字符、字形簇或者字符串 slice。因此，如果你真的希望使用索引创建字符串 slice 时，Rust 会要求你更明确一些。为了更明确索引并表明你需要一个字符串 slice，相比使用 `[]` 和单个值的索引，可以使用 `[]` 和一个 range 来创建含特定字节的字符串 slice：

```rust
let hello = "Здравствуйте";

let s = &hello[0..4];
```

这里，`s` 会是一个 `&str`，它包含字符串的头四个字节。早些时候，我们提到了这些字母都是两个字节长的，所以这意味着 `s` 将会是 `Зд`。

如果尝试用类似 **`&hello[0..1]` 的方式对字符的部分字节进行 slice**，Rust 会在运行时 panic，就跟访问 vector 中的无效索引时一样。

**遍历字符串**

操作字符串每一部分的最好的方法是明确表示需要字符还是字节。对于**单独的 Unicode 标量值使用 `chars` 方法**。对 “Зд” 调用 `chars` 方法会将其分开并返回两个 `char` 类型的值，接着就可以遍历其结果来访问每一个元素了：

```rust
for c in "Зд".chars() {
    println!("{c}");
}
```

这些代码会打印出如下内容：

```bash
З
д
```

用chars遍历就可以取出字符串里的内容了

你要字符就用 .chars()，要字节就用 .bytes()。不像其他语言默认给你一种，然后在边界情况出错。

chars() 是 str 上的方法，返回一个迭代器，按 Unicode 标量值（scalar value）逐个产出 char 类型。

char 类型本身**固定占 4 字节内存**，但它能表示的值包括所有 Unicode 字符——不管是 ASCII 还是中文。

  - char 类型大小：固定 4 字节
  - UTF-8 编码中 A 占的空间：1 字节

  - A → 1 字节（ASCII）
  - 我 → 3 字节
  - 是 → 3 字节
  - " → 1 字节（ASCII 双引号，如果算的话）

#### 处理字符串的复杂性

总而言之，字符串是复杂的。不同的编程语言会选择不同的方式，把这种复杂性呈现给程序员。Rust 选择把正确处理 `String` 数据作为所有 Rust 程序的默认行为，这意味着程序员必须在一开始就更多地思考如何处理 UTF-8 数据。这种权衡比其他编程语言更直接地暴露了字符串的复杂性，但它能避免你在开发周期的后期再去处理那些涉及非 ASCII 字符的错误。

好消息是，标准库围绕 `String` 和 `&str` 构建了很多功能，来帮助我们正确处理这些复杂场景。请务必查看相关文档，了解一些有用的方法，例如用 `contains` 搜索字符串，或用 `replace` 把字符串的一部分替换成另一段字符串。

### 使用Hash Map存储键值对

最后介绍的常用集合类型是**哈希 map**（*hash map*）。`HashMap<K, V>` 类型储存了一个键类型 `K` 对应一个值类型 `V` 的映射。它通过一个**哈希函数**（*hashing function*）来实现映射，决定如何将键和值放入内存中。很多编程语言支持这种数据结构，不过通常有不同的名字：**哈希**、**map**、**对象**、**哈希表**、**字典**或者**关联数组**。

哈希 map 可以用于需要任何类型作为键来寻找数据的情况，而不是像 vector 那样通过索引。例如，在一个游戏中，你可以将每个团队的分数记录到哈希 map 中，其中键是队伍的名字而值是每个队伍的分数。给出一个队名，就能检索到该队的得分。

**新建**

```rust
    let mut scores = HashMap::new();
    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Red"), 50);
    let teams = vec![String::from("Blue"), String::from("Red")];
    println!("{:?}",teams)
```



