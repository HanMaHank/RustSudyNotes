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

在这三个常用集合中，`HashMap` 是最不常用的，所以并没有被 prelude 自动引用。标准库中对 `HashMap` 的支持也相对较少，例如，并没有内建的构建宏。

像 vector 一样，哈希 map 将它们的数据储存在堆上，这个 `HashMap` 的键类型是 `String` 而值类型是 `i32`。类似于 vector，哈希 map 是同质的：所有的键必须是相同类型，值也必须都是相同类型。

**访问哈希map中的值**

可以通过 `get` 方法并提供对应的键来从哈希 map 中获取值

```rust
    use std::collections::HashMap;

    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);

    let team_name = String::from("Blue");
    let score = scores.get(&team_name).copied().unwrap_or(0);
```

`score` 是与蓝队分数相关的值，应为 `10`。`get` 方法返回 `Option<&V>`，如果某个键在哈希 map 中没有对应的值，`get` 会返回 `None`。程序中通过调用 `copied` 方法来获取一个 `Option<i32>` 而不是 `Option<&i32>`，接着调用 `unwrap_or` 在 `scores` 中没有该键所对应的项时将其设置为零。



Option<&i32> — 借用，生命周期绑定到 map，限制后续可变操作

Option<i32> (经过 copied) — 独立的值拷贝，不再与 map 有借用关系

**在哈希map中管理所有权**

对于像 `i32` 这样的实现了 `Copy` trait 的类型，其值可以拷贝进哈希 map。对于像 `String` 这样拥有所有权的值，其值将**被移动而哈希 map 会成为这些值的所有者**

Copy 就是"可以按位复制"的类型，或者叫"赋值时自动复制"

实现了 Copy 的基本都是"小而简单"的类型：整数、浮点数、布尔、字符、以及只包含这些类型的元组等。它们在栈上直接存储，复制成本极低，所以 Rust 让它们默认复制而不是移动。

```rust
    use std::collections::HashMap;

    let field_name = String::from("Favorite color");
    let field_value = String::from("Blue");

    let mut map = HashMap::new();
    map.insert(field_name, field_value);
    // 这里 field_name 和 field_value 不再有效，
    // 尝试使用它们看看会出现什么编译错误！
```

当 insert 调用将 field_name 和 field_value 移动到哈希 map 中后，将不能使用这两个绑定。

如果我们把对值的引用插入哈希 map，这些值本身并不会被移动进哈希 map。引用所指向的值必须至少在哈希 map 有效的那段时间里一直有效。

```rust
    let field_name = String::from("name");
    let field_value = String::from("value");
    let mut field_map = HashMap::new();
    field_map.insert(&field_name, &field_value);
    println!("{}", field_map.get(&field_name).unwrap());
    println!("{}", field_name); //如果是借用就没有问题了呢。但一般都不会这样写吧？
```

**更新哈希map**

尽管键值对的数量是可以增长的，每个唯一的键只能同时关联一个值。

当我们想要改变哈希 map 中的数据时，必须决定如何处理一个键已经有值了的情况。可以选择完全无视旧值并用新值代替旧值。可以选择保留旧值而忽略新值，并只在键**没有**对应值时增加新值。或者可以结合新旧两值。让我们看看这分别该如何实现！

**覆盖一个值**

key一样就行了

**只有key不存在时插入**

我们经常会检查某个特定的键是否已经在哈希 map 中有对应的值，然后执行如下操作：如果这个键已经存在，就让原来的值保持不变；如果这个键不存在，就插入它和它对应的值。

Hash map 为这种场景提供了一个特殊的 API，叫做 `entry`，它接收你想检查的键作为参数。

`entry` 方法的返回值是一个名为 `Entry` 的枚举，它表示一个可能存在、也可能不存在的值。

假设我们想检查黄队这个键是否已经有关联的值。如果没有，就插入值 `50`；蓝队也是同样的处理方式。使用 `entry` API 的代码如示例。

```rust
    use std::collections::HashMap;

    let mut scores = HashMap::new();
    scores.insert(String::from("Blue"), 10);

    scores.entry(String::from("Yellow")).or_insert(50);
    scores.entry(String::from("Blue")).or_insert(50);

    println!("{scores:?}");
```

**根据旧值更新一个值**

另一个常见的哈希 map 的应用场景是找到一个键对应的值并根据旧的值更新它。

```rust
    use std::collections::HashMap;

    let text = "hello world wonderful world";

    let mut map = HashMap::new();

    for word in text.split_whitespace() {
        let count = map.entry(word).or_insert(0); //返回借用可变，为啥不给option？必定有值啊！
        *count += 1;
    }

    println!("{map:?}");
```

**哈希函数**

`HashMap` 默认使用一种叫做 SipHash 的哈希函数，它可以提供对涉及哈希表[1](https://kaisery.github.io/trpl-zh-cn/ch08-03-hash-maps.html#footnote-siphash)的拒绝服务（Denial of Service, DoS）攻击的抵抗能力。不过这不是目前可用的最快哈希算法，但为了更好的安全性而接受一些性能下降，是值得的权衡。如果你分析代码后发现默认哈希函数对你的用途来说太慢，就可以通过指定不同的 hasher 来切换到其他函数。*hasher* 是一种实现了 `BuildHasher` trait 的类型。



## 错误处理

错误是软件中的常态，因此 Rust 提供了许多特性来处理某些事情出错的情况。

在很多场景下，Rust 会要求你先承认错误发生的可能性，并在代码能够通过编译之前采取一些行动。这项要求会让程序更健壮，因为它确保你会在把代码部署到生产环境之前就发现错误，并妥善处理它们。

Rust 将错误分成两大类：**可恢复的**（*recoverable*）和**不可恢复的**（*unrecoverable*）错误。对于可恢复错误，例如“文件未找到”这样的错误，我们多半只想把问题报告给用户，然后重试这次操作。不可恢复错误则总是 bug 的征兆，比如试图访问数组末尾之外的位置，因此我们希望立刻停止程序。

大多数语言并不区分这两类错误，而是使用诸如异常（exception）之类的机制以相同方式处理它们。Rust 没有异常。相反，它使用 `Result<T, E>` 类型来处理可恢复错误，使用 `panic!` 宏在程序遇到不可恢复错误时停止执行。

### 用panic处理不可恢复的错误

有时，你的代码里会发生一些糟糕的事情，而且你对此无能为力。在这种情况下，Rust 提供了 `panic!` 宏。实际中有两种方式会导致 panic：一种是执行了会让代码 panic 的操作，比如访问超出数组结尾的位置；另一种是显式调用 `panic!` 宏。这两种情况都会让程序 panic。

默认情况下，这些 panic 会打印失败信息、展开栈、清理栈数据，然后退出。你还可以通过环境变量，让 Rust 在 panic 发生时显示调用栈（call stack），以便更容易追踪 panic 的来源。

```tex
响应 panic 时的栈展开或终止
当出现 panic 时，程序默认会开始 展开（unwinding），这意味着 Rust 会回溯栈并清理它遇到的每一个函数的数据，不过这个回溯并清理的过程有很多工作。另一种选择是直接 终止（abort），这会不清理数据就退出程序。

那么程序所使用的内存需要由操作系统来清理。如果你需要项目的最终二进制文件越小越好，panic 时通过在 Cargo.toml 的 [profile] 部分增加 panic = 'abort'，可以由展开切换为终止。例如，如果你想要在 release 模式中 panic 时直接终止，可添加：

[profile.release]
panic = 'abort'
```

```rust
fn main() {
        panic!("展开！");
}

```

**使用panic!的backtrace**

**就是堆栈信息**

要获得带有这些信息的 backtrace，必须启用调试符号（debug symbols）。当像这里这样，不带 `--release` 参数运行 `cargo build` 或 `cargo run` 时，调试符号默认就是启用的。

 Rust 中的 backtrace 和其他语言里的工作方式一样：阅读 backtrace 的关键，是从最上面开始往下读，直到看到你自己写的文件。那一处就是问题开始的地方。它上面的那些行，是你的代码调用过的代码；下面的那些行，则是调用了你代码的代码。

```tex
【最上面：底层炸了 → 抛出错误】
    ↑
【中间：库函数 / 系统调用】
    ↑
【这里！⭐ 你的代码炸了 → 错误源头】
    ↑
【最下面：正常启动，调用你的代码】
```

### 用Result处理可恢复错误

大部分错误并没有严重到需要程序完全停止执行。有时函数失败的原因很容易理解并加以处理。例如，如果因为打开一个并不存在的文件而失败，此时我们可能想要创建这个文件，而不是终止进程。

```	rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

`T` 和 `E` 是泛型类型参数；第十章会详细介绍泛型。现在你需要知道的就是 `T` 代表成功时返回的 `Ok` 变体中的数据的类型，而 `E` 代表失败时返回的 `Err` 变体中的错误的类型。

因为 `Result` 有这些泛型类型参数，我们可以将 `Result` 类型和标准库中为其定义的函数用于很多不同的场景，这些情况中需要返回的成功值和失败值可能会各不相同。

```rust
use std::fs::File;

fn main() {
    let greeting_file_result = File::open("hello.txt");
}
```

`File::open` 的返回值是 `Result<T, E>`。泛型参数 `T` 会被 `File::open` 的实现放入成功返回值的类型 `std::fs::File`，这是一个文件句柄。错误返回值使用的 `E` 的类型是 `std::io::Error`。这些返回类型意味着 `File::open` 调用可能成功并返回一个可以读写的文件句柄。这个函数调用也可能会失败：例如，也许文件不存在，或者可能没有权限访问这个文件。`File::open` 函数需要一个方法在告诉我们成功与否的同时返回文件句柄或者错误信息。这些信息正好是 `Result` 枚举所代表的。

当 `File::open` 成功时，`greeting_file_result` 变量将会是一个包含文件句柄的 `Ok` 实例。当失败时，`greeting_file_result` 变量将会是一个包含了更多关于发生了何种错误的信息的 `Err` 实例。

```rust
use std::fs::File;

fn main() {
    let greeting_file_result = File::open("hello.txt");

    let greeting_file = match greeting_file_result {
        Ok(file) => file,
        Err(error) => panic!("Problem opening the file: {error:?}"),
    };
}
```

**匹配不同的错误**

不管 `File::open` 是因为什么原因失败都会 `panic!`。我们真正希望的是对不同的错误原因采取不同的行为：如果 `File::open `因为文件不存在而失败，我们希望创建这个文件并返回新文件的句柄。如果 `File::open` 因为任何其他原因失败 –例如没有打开文件的权限 –  就panic

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let greeting_file_result = File::open("hello.txt");

    let greeting_file = match greeting_file_result {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {e:?}"),
            },
            _ => {
                panic!("Problem opening the file: {error:?}");
            }
        },
    };
}
```

`ile::open` 返回的 `Err` 变体中的值类型 `io::Error`，它是一个标准库中提供的结构体。这个结构体有一个返回 `io::ErrorKind` 值的 `kind` 方法可供调用。`io::ErrorKind` 是一个标准库提供的枚举，它的变体对应 `io` 操作可能导致的不同错误类型。我们感兴趣的变体是 `ErrorKind::NotFound`，它代表尝试打开的文件并不存在。这样，`match` 就匹配完 `greeting_file_result` 了，不过对于 `error.kind()` 还有一个内层 `match`。

我们希望在内层 `match` 中检查的条件是 `error.kind()` 的返回值是否为 `ErrorKind`的 `NotFound` 变体。如果是，则通过 `File::create` 尝试创建该文件。然而因为 `File::create` 也可能会失败，还需要在内层 `match` 表达式中增加了第二个分支。当文件不能被创建，会打印出一个不同的错误信息。外层 `match` 的最后一个分支保持不变，这样对任何除了文件不存在的错误会使程序 panic。

**失败时panic的快捷方式**

`match` 能够胜任它的工作，不过它可能有点冗长并且不总是能很好的表明其意图。`Result<T, E>` 类型定义了很多辅助方法来处理各种更为特定的任务。`unwrap` 方法是一个快捷方式，其内部实现与我们`match` 表达式相同。如果 `Result` 值是变体 `Ok`，`unwrap` 会返回 `Ok` 中的值。如果 `Result` 是变体 `Err`，`unwrap` 会为我们调用 `panic!`。

```rust
use std::fs::File;

fn main() {
    let greeting_file = File::open("hello.txt").unwrap();
}
```

同样，`expect` 方法也允许我们自定义 `panic!` 的错误信息。使用 `expect` 而不是 `unwrap` 并提供一个好的错误信息可以表明你的意图并更易于追踪 panic 的根源

```rust
use std::fs::File;

fn main() {
    let greeting_file = File::open("hello.txt")
        .expect("hello.txt should be included in this project");
}
```

大部分 Rustaceans 选择 `expect` 而不是 `unwrap` 并提供更多关于为何操作期望是一直成功的上下文。



**传播错误**

当函数的实现中调用了可能会失败的操作时，除了在这个函数中处理错误外，还可以选择让调用者知道这个错误并决定该如何处理。这被称为**传播**（*propagating*）错误，这样能更好的控制代码调用，因为比起你代码所拥有的上下文，调用者可能拥有更多信息或逻辑来决定应该如何处理错误。

```rust
文件名：src/main.rs

use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let username_file_result = File::open("hello.txt"); //打开文件

    let mut username_file = match username_file_result {
        Ok(file) => file,
        Err(e) => return Err(e),
    };//判断有错吗？

    let mut username = String::new();//用来放字符串

    match username_file.read_to_string(&mut username) {
        Ok(_) => Ok(username),
        Err(e) => Err(e),
    }
}
```

这种传播错误的模式在 Rust 中太常见了，因此 Rust 提供了问号运算符 `?` 来简化这一过程。

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let mut username_file = File::open("hello.txt")?;
    let mut username = String::new();
    username_file.read_to_string(&mut username)?;
    Ok(username)
}
```

放在 `Result` 值后面的 `?`

`?`，其定义的工作方式与我们在示例 9-6 中编写的处理 `Result` 值的 `match` 表达式几乎完全相同。如果 `Result` 的值是 `Ok`，这个表达式就会返回 `Ok` 中的值，程序继续执行。如果值是 `Err`，`Err` 就会像使用了 `return` 关键字一样，作为整个函数的返回值**提前返回**，这样错误值就被传播给了调用者。

`?` 运算符消除了大量样板代码并使得函数的实现更简单。我们甚至可以在 `?` 之后直接使用链式方法调用来进一步简化代码。

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let mut username = String::new();

    File::open("hello.txt")?.read_to_string(&mut username)?;

    Ok(username)
}
```



```rust
use std::fs;
use std::io;

fn read_username_from_file() -> Result<String, io::Error> {
    fs::read_to_string("hello.txt")
}
```

将文件读取到一个字符串是相当常见的操作，所以标准库提供了名为 `fs::read_to_string` 的函数，它会打开文件、新建一个 `String`、读取文件的内容，并将内容放入 `String`，接着返回它。

**哪里可以使用？运算符**,想将错误给父处理

`?` 运算符只能被用于返回值与 `?` 作用的值相兼容的函数。因为 `?` 运算符被定义为从函数中提早返回一个值，这与示例 9-6 中的 `match` 表达式有着完全相同的工作方式。示例 9-6 中 `match` 作用于一个 `Result` 值，提早返回的分支返回了一个 `Err(e)` 值。函数的返回值必须是 `Result` 才能与这个 `return` 相兼容。

```rust
use std::fs::File;

fn main() {
    let greeting_file = File::open("hello.txt")?; //得到报错
}
```

这段代码打开一个文件，这可能会失败。`?` 运算符作用于 `File::open` 返回的 `Result` 值，不过 `main` 函数的**返回类型**是 `()` 而不是 `Result`。

 `?` 也可用于 `Option<T>` 值。如同对 `Result` 使用 `?` 一样，只能在返回 `Option` 的函数中对 `Option` 使用 `?`。在 `Option<T>` 上调用 `?` 运算符的行为与 `Result<T, E>` 类似：如果值是 `None`，此时 `None` 会从函数中提前返回。如果值是 `Some`，`Some` 中的值作为表达式的返回值同时函数继续。

```rust
fn last_char_of_first_line(text: &str) -> Option<char> {
    text.lines().next()?.chars().last()
}

text.lines()    // 把文本按行切分
     .next()?   // 取第一行，如果没有行就直接返回 None
     .chars()   // 把第一行切成字符
     .last()    // 取最后一个字符
```

目前为止，我们所使用的所有 `main` 函数都返回 `()`。`main` 函数是特殊的因为它是可执行程序的入口点和退出点，为了使程序能正常工作，其可以返回的类型是有限制的。

幸运的是 `main` 函数也可以返回 `Result<(), E>`，示例 9-12 中的代码来自示例 9-10 不过修改了 `main` 的返回值为 `Result<(), Box<dyn Error>>` 并在结尾增加了一个 `Ok(())` 作为返回值。这段代码就可以编译了。

```rust
use std::error::Error;
use std::fs::File;

fn main() -> Result<(), Box<dyn Error>> {
    let greeting_file = File::open("hello.txt")?;

    Ok(())
}
```

`Box<dyn Error>` 类型是一个**trait 对象**（*trait object*）

现在可以把 `Box<dyn Error>` 理解为“任何类型的错误”。在返回错误类型 `Box<dyn Error>` 的 `main` 函数中，对 `Result` 使用 `?` 是被允许的，因为它允许任何 `Err` 值提前返回。即便 `main` 函数体现在只会返回 `std::io::Error` 错误类型，通过指定 `Box<dyn Error>`，这个签名仍然是正确的；即使以后在 `main` 函数体中加入返回其他错误类型的代码，这个函数签名依然保持正确。

当 `main` 函数返回 `Result<(), E>`，如果 `main` 返回 `Ok(())` 可执行程序会以 `0` 值退出，而如果 `main` 返回 `Err` 值则会以非零值退出；成功退出的程序会返回整数 `0`，运行错误的程序会返回非 `0` 的整数。

`main` 函数也可以返回任何实现了 [`std::process::Termination` trait](https://doc.rust-lang.org/std/process/trait.Termination.html) 的类型，它包含了一个返回 `ExitCode` 的 `report` 函数。

### 要不要panic！

如果代码 panic，就没有恢复的可能。你可以选择对任何错误场景都调用 `panic!`，不管是否有可能恢复，不过这样就是你代替调用者决定了这是不可恢复的。

选择返回 `Result` 值的话，就将选择权交给了调用者，而不是代替他们做出决定。

因此**返回 `Result`** 是定义可能会失败的函数的一个**好的默认选择**。

**当你比编译器知道更多时**

当你有某些其他逻辑来确保 `Result` 一定会是 `Ok` 值，而编译器又无法理解这套逻辑时，调用 `expect` 也是合适的。

如果你通过人工检查代码，能够确保绝不会出现 `Err` 变体，那么调用 `expect` 完全可以接受，同时最好在 `expect` 的提示文本里记录下你为什么认为这里永远不会出现 `Err`。

```rust
    use std::net::IpAddr;

    let home: IpAddr = "127.0.0.1"
        .parse()
        .expect("Hardcoded IP address should be valid");
```

 `expect` 中写明这个 IP 地址是硬编码的这一假设，也会提醒我们：如果将来需要从其他来源获取 IP 地址，就该把 `expect` 改成更合适的错误处理代码。

**错误处理知道准则**

在当有可能会导致有害状态（bad state）的情况下建议使用 `panic!` —— 在这里，**有害状态**（*bad state*）是指当一些假设、保证、协议或不可变性被打破的状态，例如无效的值、自相矛盾的值或者被传递了不存在的值

- 有害状态是非预期的行为，与偶尔会发生的行为相对，比如用户输入了错误格式的数据。
- 在此之后代码的运行依赖于不处于这种有害状态，而不是在每一步都检查是否有问题。
- 没有可行的手段来将有害状态信息编码进所使用的类型中的情况。我们会在第十八章[“将状态和行为编码为类型”](https://kaisery.github.io/trpl-zh-cn/ch18-03-oo-design-patterns.html#将状态和行为编码为类型)部分通过一个例子来说明我们的意思。
- 如果你正在调用不受你控制的外部代码，并且它返回了一个你无法修复的无效状态，那么 `panic!` 往往是合适的。

然而当错误预期会出现时，返回 `Result` 仍要比调用 `panic!` 更为合适。这样的例子包括解析器接收到格式错误的数据，或者 HTTP 请求返回了一个表明触发了限流的状态。在这些例子中，应该通过返回 `Result` 来表明失败预期是可能的，而调用者就必须决定该如何处理这个问题。

虽然在所有函数中都拥有许多错误检查是冗长而烦人的。幸运的是，可以利用 Rust 的类型系统（以及编译器的类型检查）为你进行很多检查。

如果函数有一个特定类型的参数，可以在知晓编译器已经确保其拥有一个有效值的前提下进行你的代码逻辑。例如，如果你使用了一个并不是 `Option` 的类型，则程序期望它是**有值**的并且不是**空值**。你的代码无需处理 `Some` 和 `None` 这两种情况，它只会有一种情况就是绝对会有一个值。尝试向函数传递空值的代码甚至根本不能编译，所以你的函数在运行时没有必要判空。另外一个例子是使用像 `u32` 这样的无符号整型，也会确保它永远不为负。

```rust
pub struct Guess {
    value: i32,
}

impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 || value > 100 { //new的时候范围不在直接抛出异常
            panic!("Guess value must be between 1 and 100, got {value}.");
        }

        Guess { value }
    }

    pub fn value(&self) -> i32 {
        self.value
    }
}

```

Rust 的错误处理功能被设计为帮助你编写更加健壮的代码。`panic!` 宏代表一个程序无法处理的状态，并停止执行而不是使用无效或不正确的值继续处理。Rust 类型系统的 `Result` 枚举代表操作可能会在一种可以恢复的情况下失败。可以使用 `Result` 来告诉代码调用者他需要处理潜在的成功或失败。在适当的场景使用 `panic!` 和 `Result` 将会使你的代码在面对不可避免的错误时显得更加可靠。



## 泛型、trait和生命周期

每种编程语言都有一些工具，用来高效处理概念上的重复。在 Rust 中，这样的工具之一就是**泛型**（*generics*）：它是具体类型或其他属性的抽象占位符。我们可以描述泛型的行为，或者它们与其他泛型之间的关系，而不需要在编译和运行代码时就提前知道它们具体会被什么替换。

函数可以像接收未知值那样，接收某种泛型类型的参数，而不是像 `i32` 或 `String` 这样的具体类型，从而让同一段代码可以作用于多种具体值。

我们会回顾如何通过提取函数来减少代码重复。然后，我们会使用同样的技巧，把两个只在参数类型上不同的函数变成一个泛型函数。我们也会解释如何在结构体和枚举定义中使用泛型类型。

然后，你会学到如何使用 **trait** 以泛型的方式定义行为。你可以把 trait 和泛型类型组合起来，把某个泛型类型限制为只接受具有特定行为的类型，而不是任意类型。

最后，我们会讨论 **生命周期**（*lifetimes*）：它是一类向编译器提供“引用之间如何相互关联”信息的泛型。生命周期让我们能够向编译器提供足够的信息，使它在更多场景下也能确认引用是有效的。

### 提取函数来减少重复

泛型允许我们使用一个可以代表多种类型的占位符来替换特定类型，以此来减少代码冗余。

```rust
fn largest(list: &[i32]) -> &i32 {
    let mut largest = &list[0];

    for item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!("The largest number is {result}");

    let number_list = vec![102, 34, 6000, 89, 54, 2, 43, 8];

    let result = largest(&number_list);
    println!("The largest number is {result}");
}
```

### 泛型数据类型

我们使用泛型来为函数签名或结构体之类的项创建定义，这样它们就可以配合多种不同的具体数据类型使用。

```rust
fn largest<T>(list: &[T]) -> &T {
    let mut largest = &list[0];

    for item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!("The largest number is {result}");

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest(&char_list);
    println!("The largest char is {result}");
}
```

如果现在就编译这段代码会出现错误

帮助信息里提到了 `std::cmp::PartialOrd`。要想比较得实现比较的trait

```rust
fn largest<T: std::cmp::PartialOrd>(list: &[T]) -> &T { //这样就可以运行了
    let mut largest = &list[0];

    for item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!("The largest number is {result}");

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest(&char_list);
    println!("The largest char is {result}");
}
```



### 结构体定义中的泛型

我们也可以使用 `<>` 语法来定义结构体，让一个或多个字段使用泛型类型参数。

```rust
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let integer = Point { x: 5, y: 10 };
    let float = Point { x: 1.0, y: 4.0 };
}
```

```rust
struct Point<T, U> {
    x: T,
    y: U,
}

fn main() {
    let both_integer = Point { x: 5, y: 10 };
    let both_float = Point { x: 1.0, y: 4.0 };
    let integer_and_float = Point { x: 5, y: 4.0 };
}
```

### 枚举定义中泛型

和结构体类似，枚举也可以在成员中存放泛型数据类型。

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

`Result` 枚举有两个泛型类型，`T` 和 `E`。`Result` 有两个成员：`Ok`，它存放一个类型 `T` 的值，而 `Err` 则存放一个类型 `E` 的值。这个定义使得 `Result` 枚举能很方便的表达任何可能成功（返回 `T` 类型的值）也可能失败（返回 `E` 类型的值）的操作。

### 方法定义中的泛型

我们可以像第五章那样为结构体和枚举实现方法，并在这些方法定义中使用泛型。

```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x //返回我自己吗
    }
}

fn main() {
    let p = Point { x: 5, y: 10 };

    println!("p.x = {}", p.x());
}
```

这里在 `Point<T>` 上定义了一个叫做 `x` 的方法用于返回字段 `x` 中数据的引用。

注意必须在 `impl` 后面声明 `T`，这样就可以在 `Point<T>` 上实现的方法中使用 `T` 了。通过在 `impl` 之后声明泛型 `T`，Rust 就知道 `Point` 的尖括号中的类型是泛型而不是具体类型。

定义方法时也可以为泛型指定限制（constraint）。

以选择为 `Point<f32>` 实例实现方法，而不是为泛型 `Point` 实例。

```rust
impl Point<f32> { //就是f32才有这个方法
    fn distance_from_origin(&self) -> f32 { 
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}
```

```rust
struct Point<X1, Y1> {
    x: X1,
    y: Y1,
}

impl<X1, Y1> Point<X1, Y1> {
    fn mixup<X2, Y2>(self, other: Point<X2, Y2>) -> Point<X1, Y2> {
        Point {
            x: self.x,
            y: other.y,
        }
    }
}

fn main() {
    let p1 = Point { x: 5, y: 10.4 };
    let p2 = Point { x: "Hello", y: 'c' };

    let p3 = p1.mixup(p2);

    println!("p3.x = {}, p3.y = {}", p3.x, p3.y);
}
```

### 泛型的代码开销

你可能会好奇，使用泛型类型参数是否会带来运行时开销。好消息是：使用泛型不会让程序比使用具体类型运行得更慢。

Rust 通过在编译时对泛型代码进行**单态化**（*monomorphization*）来实现这一点。单态化就是把泛型代码转换成具体代码的过程，方法是用编译时实际用到的具体类型去填充泛型代码。

在这个过程中，编译器所做的工作正好与示例 10-5 中我们创建泛型函数的步骤相反。编译器寻找所有泛型代码被调用的位置并使用泛型代码针对具体类型生成代码。

代码是泛型，编译后就不是了。

## Trait:定义共同的行为

*trait* 定义了某个特定类型拥有可能与其他类型共享的功能。可以通过 trait 以一种抽象的方式定义共同行为。可以使用 *trait bounds* 指定泛型是任何拥有特定行为的类型。

*trait* 类似于其他语言中的常被称为 **接口**（*interfaces*）的功能，虽然有一些不同。

**定义trait**
一个类型的行为由其可供调用的方法构成。如果可以对不同类型调用相同的方法的话，这些类型就可以共享相同的行为了。trait 定义是一种将方法签名组合起来的方法，目的是定义一个实现某些目的所必需的行为的集合。

例如，这里有多个存放了不同类型和属性文本的结构体：结构体 `NewsArticle` 用于存放发生于世界各地的新闻故事，而结构体 `SocialPost` 最多只能存放 280 个字符的内容，以及指示该帖子是新发布的、转发的还是对另一条帖子的回复的元数据。

我们想要创建一个名为 `aggregator` 的多媒体聚合库用来显示可能储存在 `NewsArticle` 或 `SocialPost` 实例中的数据摘要。为了实现功能，每个结构体都要能够获取摘要，这样的话就可以调用实例的 `summarize` 方法来请求摘要。

```rust
pub trait Summary { //每一个结构体都需要有的方法
    fn summarize(&self) -> String;
}
```

这里使用 `trait` 关键字来声明一个 trait，后面是 trait 的名字，在这个例子中是 `Summary`。我们也声明 `trait` 为 `pub` 以便依赖这个 crate 的其它 crate 也可以使用这个 trait。

在方法签名后跟分号，而不是在大括号中提供其实现。接着每一个实现这个 trait 的类型都需要提供其自定义行为的方法体，编译器也会确保任何实现 `Summary` trait 的类型都拥有与这个签名的定义完全一致的 `summarize` 方法。

trait 体中可以有多个方法：一行一个方法签名且都以分号结尾。

**为类型实现trait**

现在我们定义了 `Summary` trait 的签名，接着就可以在多媒体聚合库中实现这个类型了。

```rust
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

//实现这个接口为了...
impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

pub struct SocialPost {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub repost: bool,
}

impl Summary for SocialPost {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}
```

在类型上实现 trait 类似于实现常规方法。区别在于 `impl` 关键字之后，我们提供需要实现 trait 的名称，接着是 `for` 和需要实现 trait 的类型的名称。

在 `impl` 块中，使用 trait 定义中的方法签名，不过不再后跟分号，而是需要在大括号中编写函数体来为特定类型实现 trait 方法所拥有的行为。

现在库在 `NewsArticle` 和 `SocialPost` 上实现了`Summary` trait，crate 的用户可以像调用常规方法一样调用 `NewsArticle` 和 `SocialPost` 实例的 trait 方法了。

唯一的区别是 trait 必须和类型一起引入作用域以便使用额外的 trait 方法。

```rust
use aggregator::{SocialPost, Summary};

fn main() {
    let post = SocialPost {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        repost: false,
    };

    println!("1 new post: {}", post.summarize());
}
```

### 使用默认实现

有时为 trait 中的某些或全部方法提供默认的行为，而不是在每个类型的每个实现中都定义自己的行为是很有用的。

```rust
pub trait Summary {
    fn summarize(&self) -> String {
        String::from("(Read more...)")
    }
}
```

如果想要对 `NewsArticle` 实例使用这个默认实现，可以通过 `impl Summary for NewsArticle {}` 指定一个空的 `impl` 块。

虽然我们不再直接为 `NewsArticle` 定义 `summarize` 方法了，但是我们提供了一个默认实现并且指定 `NewsArticle` 实现 `Summary` trait。因此，我们仍然可以对 `NewsArticle` 实例调用 `summarize` 方法

```rust
    let article = NewsArticle {
        headline: String::from("Penguins win the Stanley Cup Championship!"),
        location: String::from("Pittsburgh, PA, USA"),
        author: String::from("Iceburgh"),
        content: String::from(
            "The Pittsburgh Penguins once again are the best \
             hockey team in the NHL.",
        ),
    };

    println!("New article available! {}", article.summarize());
```

### 使用trait作为参数

知道了如何定义 trait 和在类型上实现这些 trait 之后，我们可以探索一下如何使用 trait 来接受多种不同类型的参数。

```rust
pub fn notify(item: &impl Summary) { //能传递进来的只有实现了这个的，否则不能编译
    println!("Breaking news! {}", item.summarize());
}
```

**Trait Bound 语法**

`impl Trait` 语法更直观，但它实际上是更长形式的 *trait bound* 语法的语法糖。

```rust
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
```

这种更冗长的写法与上一节的示例等价，但更为冗长。trait bound 与泛型参数声明在一起，位于尖括号中的冒号后面。

`impl Trait` 很方便，适用于短小的例子。更长的 trait bound 则适用于更复杂的场景。例如，可以获取两个实现了 `Summary` 的参数。使用 `impl Trait` 的语法看起来像这样：

```rust
pub fn notify(item1: &impl Summary, item2: &impl Summary) {
```

这适用于 `item1` 和 `item2` 可以是不同类型的情况，只要它们都实现了 `Summary`。不过，如果我们希望强制两个参数必须具有**相同类型**，就必须使用 trait bound

```rust
pub fn notify<T: Summary>(item1: &T, item2: &T) {
```

泛型 `T` 被指定为 `item1` 和 `item2` 的参数限制，如此传递给参数 `item1` 和 `item2` 值的具体类型必须一致。

**通过+语法指定多个Trait bound**

我们也可以指定多个 trait bound。假设我们希望 `notify` 在 `item` 上既能使用格式化显示，又能使用 `summarize` 方法：在 `notify` 的定义中，指定 `item` 必须同时实现 `Display` 和 `Summary` 两个 trait。

```rust
pub fn notify(item: &(impl Summary + Display)) { //必须两个都实现
```

```rust
pub fn notify<T: Summary + Display>(item: &T) {
```

**通过where简化trait bound**

然而，使用过多的 trait bound 也有缺点。每个泛型有其自己的 trait bound，所以有多个泛型参数的函数在名称和参数列表之间会有很长的 trait bound 信息，这使得函数签名难以阅读。

```rust
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {
```

```rust
fn some_function<T, U>(t: &T, u: &U) -> i32
where
    T: Display + Clone,
    U: Clone + Debug,
{
```

这个函数签名就显得不那么杂乱，函数名、参数列表和返回值类型都离得很近，看起来跟没有那么多 trait bounds 的函数很像。

### 返回实现了trait的类型

```rust
fn returns_summarizable() -> impl Summary {
    SocialPost {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        repost: false,
    }
}
```

通过使用 `impl Summary` 作为返回值类型，我们指定了 `returns_summarizable` 函数返回某个实现了 `Summary` trait 的类型，但是不确定其具体的类型。在这个例子中 `returns_summarizable` 返回了一个 `SocialPost`，不过调用方并不知情。

返回一个只是指定了需要实现的 trait 的类型的能力在闭包和迭代器场景十分的有用

不过这只适用于返回单一类型的情况。例如，这段代码的返回值类型指定为返回 `impl Summary`，但是返回了 `NewsArticle` 或 `SocialPost` 就行不通

以下报错 //只能返回一种类型，这里多种了，和java倒是不一样

```rust
fn returns_summarizable(switch: bool) -> impl Summary {
    if switch {
        NewsArticle {
            headline: String::from(
                "Penguins win the Stanley Cup Championship!",
            ),
            location: String::from("Pittsburgh, PA, USA"),
            author: String::from("Iceburgh"),
            content: String::from(
                "The Pittsburgh Penguins once again are the best \
                 hockey team in the NHL.",
            ),
        }
    } else {
        SocialPost {
            username: String::from("horse_ebooks"),
            content: String::from(
                "of course, as you probably already know, people",
            ),
            reply: false,
            repost: false,
        }
    }
}
```

**使用trait bound有条件地实现方法**

通过使用带有 trait bound 的泛型参数的 `impl` 块，可以有条件地只为那些实现了特定 trait 的类型实现方法。

```rust
use std::fmt::Display;

struct Pair<T> {
    x: T,
    y: T,
}

impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Self { x, y }
    }
}

impl<T: Display + PartialOrd> Pair<T> { //只给满足条件的 T 实现方法
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}
```

也可以对任何实现了特定 trait 的类型有条件地实现 trait。对任何满足特定 trait bound 的类型实现 trait 被称为 *blanket implementations*，它们被广泛的用于 Rust 标准库中。

标准库为任何实现了 `Display` trait 的类型实现了 `ToString` trait。这个 `impl` 块看起来像这样：

```rust
impl<T: Display> ToString for T {
    // --snip--
}
```

因为标准库有了这些 blanket implementation，我们可以对任何实现了 `Display` trait 的类型调用由 `ToString` 定义的 `to_string` 方法

``` rust
let s = 3.to_string();
```

```rust
use std::fmt::Display;

#[derive(Debug, Clone)]  // 👈 在这里加！直接获得 Clone
struct Human<T> {
    name: String,
    age: u8,
    time: T,
}
impl<T> Human<T> {
    fn new(name: String, age: u8, time: T) -> Self {
        Self { name, age, time }
    }
}

trait Summary {
    fn summarize(&self) -> String;
}

impl<T> Summary for Human<T> {
    fn summarize(&self) -> String {
        format!("{}: {}", self.name, self.age)
    }
}

impl<T: Display> Display for Human<T> {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        write!(f, "{}({})", self.name, self.age)
    }
}

fn main() {
    let h = Human::new("John".to_string(), 15, 21);
    let c =  Human::new("Hank".to_string(), 25, 31);
    println!("{}", h.summarize());
    some_function(&h,&c);
}

fn some_function<T: Display + Clone, U: Clone + Summary>(t: &T, u: &U) {
    println!("{}", u.summarize());
}
```

## 生命周期确保引用有效

生命周期是另一类我们已经使用过的泛型。不同于确保类型有期望的行为，生命周期用于保证引用在我们需要的整个期间内都是有效的。

Rust 中的每一个引用都有其**生命周期**（*lifetime*），也就是引用保持有效的作用域。大部分时候生命周期是隐含并可以推断的，正如大部分时候类型也是可以推断的一样。

类似于当因为有多种可能类型的时候必须注明类型，也会出现引用的生命周期以一些不同方式相关联的情况，所以 Rust 需要我们使用泛型生命周期参数来注明它们的关系，这样就能确保运行时实际使用的引用绝对是有效的。

### 悬垂引用

生命周期的主要目标是避免**悬垂引用**（*dangling references*），后者会导致程序引用了非预期引用的数据。

```rust
fn main() {
    let r;

    {
        let x = 5;
        r = &x; //我X已死，勿念
    }

    println!("r: {r}");
}
```

外部作用域声明了一个没有初值的变量 `r`，而内部作用域声明了一个初值为 `5` 的变量`x`。在内部作用域中，我们尝试将 `r` 的值设置为一个 `x` 的引用。接着在内部作用域结束后，尝试打印出 `r` 的值。这段代码不能编译因为 `r` 引用的值在尝试使用之前就离开了作用域。

变量 `x` 并没有 “存在的足够久”。其原因是 `x` 在到达第 7 行内部作用域结束时就离开了作用域。不过 `r` 在外部作用域仍是有效的；作用域越大我们就说它 “存在的越久”。如果 Rust 允许这段代码工作，`r` 将会引用在 `x` 离开作用域时被释放的内存，这时尝试对 `r` 做任何操作都不能正常工作。那么 Rust 是如何决定这段代码是不被允许的呢？这得益于借用检查器。

Rust 编译器有一个**借用检查器**（*borrow checker*），它比较作用域来确保所有的借用都是有效的。

```rust
fn main() {
    let r;                // ---------+-- 'a
                          //          |
    {                     //          |
        let x = 5;        // -+-- 'b  |
        r = &x;           //  |       |
    }                     // -+       |
                          //          |
    println!("r: {r}");   //          |
}                         // ---------+
```

这里将 `r` 的生命周期标记为 `'a` 并将 `x` 的生命周期标记为 `'b`。如你所见，内部的 `'b` 块要比外部的生命周期 `'a` 小得多。在编译时，Rust 比较这两个生命周期的大小，并发现 `r` 拥有生命周期 `'a`，不过它引用了一个拥有生命周期 `'b` 的对象。程序被拒绝编译，因为生命周期 `'b` 比生命周期 `'a` 要小：被引用的对象比它的引用者存在的时间更短。

让我们看看并没有产生悬垂引用且可以正确编译的例子：

```rust
fn main() {
    let x = 5;            // ----------+-- 'b
                          //           |
    let r = &x;           // --+-- 'a  |
                          //   |       |
    println!("r: {r}");   //   |       |
                          // --+       |
}                         // ----------+
```

这里 `x` 拥有生命周期 `'b`，比 `'a` 要大。这就意味着 `r` 可以引用 `x`：Rust 知道 `r` 中的引用在 `x` 有效的时候也总是有效的。

### 函数中的泛型生命周期

让我们来编写一个返回两个字符串 slice 中较长者的函数。这个函数获取两个字符串 slice 并返回一个字符串 slice。

```rust
fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let result = longest(string1.as_str(), string2);
    println!("The longest string is {result}");
}
```

注意这个函数获取作为引用的字符串 slice，而不是字符串，因为我们不希望 `longest` 函数获取参数的所有权

如果尝试像示例 10-20 中那样实现 `longest` 函数，它并不能编译：

```rust
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() { x } else { y } //不知道会返回哪个
}
```

相应地会出现如下有关生命周期的错误：

```tex
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0106]: missing lifetime specifier
 --> src/main.rs:9:33
  |
9 | fn longest(x: &str, y: &str) -> &str {
  |               ----     ----     ^ expected named lifetime parameter
  |
  = help: this function's return type contains a borrowed value, but the signature does not say whether it is borrowed from `x` or `y`
help: consider introducing a named lifetime parameter
  |
9 | fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
  |           ++++     ++          ++          ++

For more information about this error, try `rustc --explain E0106`.
error: could not compile `chapter10` (bin "chapter10") due to 1 previous error

```

提示文本揭示了返回值需要一个泛型生命周期参数，因为 Rust 并不知道将要返回的引用是指向 `x` 或 `y`。事实上我们也不知道，因为函数体中 `if` 块返回一个 `x` 的引用而 `else` 块返回一个 `y` 的引用！

当我们定义这个函数的时候，并不知道传递给函数的具体值，所以也不知道到底是 `if` 还是 `else` 会被执行。我们也不知道传入的引用的具体生命周期，所以也就不能像示例 10-17 和 10-18 那样通过观察作用域来确定返回的引用是否总是有效。借用检查器自身同样也无法确定，因为它不知道 `x` 和 `y` 的生命周期是如何与返回值的生命周期相关联的。为了修复这个错误，我们将增加泛型生命周期参数来定义引用间的关系以便借用检查器可以进行分析。

### 生命周期注解语法

