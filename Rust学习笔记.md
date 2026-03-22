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





