
- [The Multics PL/1 language](https://multicians.org/pl1.html)
- [Programming language](https://en.wikipedia.org/wiki/Programming_language)
- [Operating system](https://en.wikipedia.org/wiki/Operating_system)
- [System programming language](https://en.wikipedia.org/wiki/System_programming_language)
- [B5500 ref man](http://www.bitsavers.org/pdf/burroughs/B5000_5500_5700/1021326_B5500_RefMan_May67.pdf)
- [Burroughs MCP](https://en.wikipedia.org/wiki/Burroughs_MCP)



## 理解rust的ownership

###  [栈（Stack）与堆（Heap）](https://kaisery.github.io/trpl-zh-cn/ch04-01-what-is-ownership.html#栈stack与堆heap)

在很多语言中，你并不需要经常考虑到栈与堆。不过在像 Rust 这样的系统编程语言中，值是位于栈上还是堆上在更大程度上影响了语言的行为以及为何必须做出这样的抉择。

跟踪哪部分代码正在使用堆上的哪些数据，最大限度的减少堆上的重复数据的数量，以及清理堆上不再使用的数据确保不会耗尽空间，这些问题正是所有权系统要处理的。一旦理解了所有权，你就不需要经常考虑栈和堆了，不过明白了所有权的存在就是为了管理堆数据，能够帮助解释为什么所有权要以这种方式工作。

###  [所有权规则](https://kaisery.github.io/trpl-zh-cn/ch04-01-what-is-ownership.html#所有权规则)

> 1. Rust 中的每一个值都有一个被称为其 **所有者**（*owner*）的变量。
> 2. 值有且只有一个所有者。
> 3. 当所有者（变量）离开作用域，这个值将被丢弃。



### 变量和其有效的作用域

**作用域**是一个**项（item）**在程序中有效的范围。当 变量`s` **进入作用域** 时，它就是有效的。这一直持续到它 **离开作用域** 为止。

### [内存与分配](https://kaisery.github.io/trpl-zh-cn/ch04-01-what-is-ownership.html#内存与分配)

对于 `String` 类型，为了支持一个可变，可增长的文本片段，需要在堆上分配一块在编译时未知大小的内存来存放内容。这意味着：

- 必须在运行时向操作系统请求内存。
- 需要一个当我们处理完 `String` 时将内存返回给操作系统的方法。

Rust 采取了一个不同的策略：内存在拥有它的变量离开作用域后就被自动释放。即当变量离开作用域，Rust 为我们调用一个特殊的函数。这个函数叫做 `drop`，在这里（如在表示离开作用域的 `}` 处）可以放置释放内存的代码。

### 拷贝方式

```rust
let x = 5;
let y = x;
```

Rust 有一个叫做 `Copy` trait 的特殊注解，可以用在类似整型这样的存储在栈上的类型上。如果一个类型拥有 `Copy` trait，一个旧的变量在将其赋值给其他变量后仍然可用。Rust 不允许自身或其任何部分实现了 `Drop` trait 的类型使用 `Copy` trait。

那么什么类型是 `Copy` 的呢？可以查看给定类型的文档来确认，不过作为一个通用的规则，任何简单标量值的组合可以是 `Copy` 的，不需要分配内存或某种形式资源的类型是 `Copy` 的。如下是一些 `Copy` 的类型：

- 所有整数类型，比如 `u32`。
- 布尔类型，`bool`，它的值是 `true` 和 `false`。
- 所有浮点数类型，比如 `f64`。
- 字符类型，`char`。
- 元组，当且仅当其包含的类型也都是 `Copy` 的时候。比如，`(i32, i32)` 是 `Copy` 的，但 `(i32, String)` 就不是。

### 移动方式

```rust
let s1 = String::from("hello");
let s2 = s1;
```

**移动**：s1被 **浅拷贝**（*shallow copy*）（拷贝指针、长度和容量而不拷贝数据）给了s2，且使s1无效。

### 克隆方式

```rust
let s1 = String::from("hello");
let s2 = s1.clone();

println!("s1 = {}, s2 = {}", s1, s2);
```

**克隆（clone）**： **深拷贝**（*deep copy*）

### [所有权与函数](https://kaisery.github.io/trpl-zh-cn/ch04-01-what-is-ownership.html#所有权与函数)

将值传递给函数在语义上与给变量赋值相似。向函数传递值可能会移动或者复制，就像赋值语句一样。

```rust
fn main() {
    let s = String::from("hello");  // s 进入作用域

    takes_ownership(s);             // s 的值移动到函数里 ...
                                    // ... 所以到这里不再有效

    let x = 5;                      // x 进入作用域

    makes_copy(x);                  // x 应该移动函数里，
                                    // 但 i32 是 Copy 的，所以在后面可继续使用 x

} // 这里, x 先移出了作用域，然后是 s。但因为 s 的值已被移走，
  // 所以不会有特殊操作

fn takes_ownership(some_string: String) { // some_string 进入作用域
    println!("{}", some_string);
} // 这里，some_string 移出作用域并调用 `drop` 方法。占用的内存被释放

fn makes_copy(some_integer: i32) { // some_integer 进入作用域
    println!("{}", some_integer);
} // 这里，some_integer 移出作用域。不会有特殊操作
```

返回值也可以转移所有权。

```rust
fn main() {
    let s1 = gives_ownership();         // gives_ownership 将返回值
                                        // 移给 s1

    let s2 = String::from("hello");     // s2 进入作用域

    let s3 = takes_and_gives_back(s2);  // s2 被移动到
                                        // takes_and_gives_back 中, 
                                        // 它也将返回值移给 s3
} // 这里, s3 移出作用域并被丢弃。s2 也移出作用域，但已被移走，
  // 所以什么也不会发生。s1 移出作用域并被丢弃

fn gives_ownership() -> String {             // gives_ownership 将返回值移动给
                                             // 调用它的函数

    let some_string = String::from("hello"); // some_string 进入作用域.

    some_string                              // 返回 some_string 并移出给调用的函数
}

// takes_and_gives_back 将传入字符串并返回该值
fn takes_and_gives_back(a_string: String) -> String { // a_string 进入作用域

    a_string  // 返回 a_string 并移出给调用的函数
}
```

变量的所有权总是遵循相同的模式：将值赋给另一个变量时移动它。当持有堆中数据值的变量离开作用域时，其值将通过 `drop` 被清理掉，除非数据被移动为另一个变量所有。

### [引用与借用](https://kaisery.github.io/trpl-zh-cn/ch04-02-references-and-borrowing.html#引用与借用)

 **引用**：& 符号就是 **引用**，它们允许你使用值但不获取其所有权。可以把引用看成是指向变量的指针，变量是指向值的“指针”。当引用离开作用域后并不丢弃它指向的数据，因为它没有所有权。个不可变引用是可以同时存在。

借用：函数参数是引用类型的称为**借用**（*borrowing*）。被调用的函数**借用**了调用函数发过来的**引用**。

```rust
fn main() {
    let s1 = String::from("hello");

    let len = calculate_length(&s1);

    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize {
    s.len()
}
```

### [可变引用](https://kaisery.github.io/trpl-zh-cn/ch04-02-references-and-borrowing.html#可变引用)

可变引用有一个很大的限制：在特定作用域中的特定数据有且只有一个可变引用。

```rust
fn main() {
    let mut s = String::from("hello");

    change(&mut s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

这个限制的好处是 Rust 可以在编译时就避免数据竞争。**数据竞争**（*data race*）类似于竞态条件，它可由这三个行为造成：

- 两个或更多指针同时访问同一数据。
- 至少有一个指针被用来写入数据。
- 没有同步数据访问的机制。

可以使用大括号来创建一个新的作用域，以允许拥有多个可变引用，只是不能 **同时** 拥有。

[悬垂引用（Dangling References）](https://kaisery.github.io/trpl-zh-cn/ch04-02-references-and-borrowing.html#悬垂引用dangling-references)

**悬垂指针**（*dangling pointer*）：是其指向的内存可能已经被分配给其它持有者或已经被释放。

Rust 中编译器确保引用永远也不会变成悬垂状态：当你拥有一些数据的引用，编译器确保**数据**不会在其引用**之前离开**作用域。

### [引用的规则](https://kaisery.github.io/trpl-zh-cn/ch04-02-references-and-borrowing.html#引用的规则)

- 在任意给定时间，**要么** 只能有一个可变引用，**要么** 只能有多个不可变引用。
- 引用必须总是有效的。

### [生命周期与引用有效性](https://kaisery.github.io/trpl-zh-cn/ch10-03-lifetime-syntax.html#生命周期与引用有效性)

Rust 中的每一个引用都有其 **生命周期**（*lifetime*），也就是引用保持有效的作用域。生命周期的主要目标是避免悬垂引用，它会导致程序引用了非预期引用的数据。Rust 编译器有一个 **借用检查器**（*borrow checker*），它比较作用域来确保所有的借用都是有效的。

```rust
{
    let r;                // ---------+-- 'a
                          //          |
    {                     //          |
        let x = 5;        // -+-- 'b  |
        r = &x;           //  |       |
    }                     // -+       |
                          //          |
    println!("r: {}", r); //          |
}                         // ---------+
```

这里将 `r` 的生命周期标记为 `'a` 并将 `x` 的生命周期标记为 `'b`。如你所见，内部的 `'b` 块要比外部的生命周期 `'a` 小得多。在编译时，Rust 比较这两个生命周期的大小，并发现 `r` 拥有生命周期 `'a`，不过它引用了一个拥有生命周期 `'b` 的对象。程序被拒绝编译，因为生命周期 `'b` 比生命周期 `'a` 要小：被引用的对象比它的引用者存在的时间更短。

```rust
&i32        // 引用
&'a i32     // 带有显式生命周期的引用
&'a mut i32 // 带有显式生命周期的可变引用

fn main() {
    let string1 = String::from("long string is long");
    let result;
    {
        let string2 = String::from("xyz");
        result = longest(string1.as_str(), string2.as_str());
    }
    println!("The longest string is {}", result);
}

fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

error[E0597]: `string2` does not live long enough
  --> src/main.rs:15:5
   |
14 |         result = longest(string1.as_str(), string2.as_str());
   |                                            ------- borrow occurs here
15 |     }
   |     ^ `string2` dropped here while still borrowed
16 |     println!("The longest string is {}", result);
17 | }
   | - borrowed value needs to live until here
```

错误表明为了保证 `println!` 中的 `result` 是有效的，`string2` 需要直到外部作用域结束都是有效的。Rust 知道这些是因为（`longest`）函数的参数和返回值都使用了相同的生命周期参数 `'a`。

当从函数返回一个引用，返回值的生命周期参数需要与一个参数的生命周期参数相匹配。如果返回的引用 **没有** 指向任何一个参数，那么唯一的可能就是它指向一个函数内部创建的值，它将会是一个悬垂引用，因为它将会在函数结束时离开作用域。尝试考虑这个并不能编译的 `longest` 函数实现：

```rust
fn longest<'a>(x: &str, y: &str) -> &'a str {
    let result = String::from("really long string");
    result.as_str()
}
```

出现的问题是 `result` 在 `longest` 函数的结尾将离开作用域并被清理，而我们尝试从函数返回一个 `result` 的引用。无法指定生命周期参数来改变悬垂引用，而且 Rust 也不允许我们创建一个悬垂引用。在这种情况，最好的解决方案是返回一个有所有权的数据类型而不是一个引用，这样函数调用者就需要负责清理这个值了。

生命周期注解并不改变任何引用的生命周期的长短。与当函数签名中指定了泛型类型参数后就可以接受任何类型一样，当指定了泛型生命周期后函数也能接受任何生命周期的引用。生命周期注解描述了多个引用生命周期相互的关系，而不影响其生命周期。

### [结构体定义中的生命周期注解](https://kaisery.github.io/trpl-zh-cn/ch10-03-lifetime-syntax.html#结构体定义中的生命周期注解)

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}
fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.')
        .next()
        .expect("Could not find a '.'");
    let i = ImportantExcerpt { part: first_sentence };
}
```

这个结构体有一个字段，`part`，它存放了一个字符串 slice，这是一个引用。类似于泛型参数类型，必须在结构体名称后面的尖括号中声明泛型生命周期参数，以便在结构体定义中使用生命周期参数。这个注解意味着 `ImportantExcerpt` 的实例不能比其 `part` 字段中的引用存在的更久。

### [生命周期省略（Lifetime Elision）](https://kaisery.github.io/trpl-zh-cn/ch10-03-lifetime-syntax.html#生命周期省略lifetime-elision)

函数或方法的参数的生命周期被称为 **输入生命周期**（*input lifetimes*），而返回值的生命周期被称为 **输出生命周期**（*output lifetimes*）。

被编码进 Rust 引用分析的模式被称为 **生命周期省略规则**（*lifetime elision rules*）。这并不是需要程序员遵守的规则；这些规则是一系列特定的场景，此时编译器会考虑，如果代码符合这些场景，就无需明确指定生命周期。

- **规则1**: 每一个是引用的参数都有它自己的生命周期参数。换句话说就是，有一个引用参数的函数有一个生命周期参数：`fn foo<'a>(x: &'a i32)`，有两个引用参数的函数有两个不同的生命周期参数，`fn foo<'a, 'b>(x: &'a i32, y: &'b i32)`，依此类推。
- **规则2**：如果只有一个输入生命周期参数，那么它被赋予所有输出生命周期参数：`fn foo<'a>(x: &'a i32) -> &'a i32`。
- **规则3**：是如果方法有多个输入生命周期参数，不过其中之一因为方法的缘故为 `&self` 或 `&mut self`，那么 `self` 的生命周期被赋给所有输出生命周期参数。

编译器采用三条规则来判断引用何时不需要明确的注解。第一条规则适用于输入生命周期，后两条规则适用于输出生命周期。如果编译器检查完这三条规则后仍然存在没有计算出生命周期的引用，编译器将会停止并生成错误。这些规则适用于 `fn` 定义，以及 `impl` 块。

### [方法定义中的生命周期注解](https://kaisery.github.io/trpl-zh-cn/ch10-03-lifetime-syntax.html#方法定义中的生命周期注解)

（实现方法时）结构体字段的生命周期必须总是在 `impl` 关键字之后声明并在结构体名称之后被使用，因为这些生命周期是结构体类型的一部分。

```rust
impl<'a> ImportantExcerpt<'a> {
    fn level(&self) -> i32 {
        3
    }
}
```

### [静态生命周期](https://kaisery.github.io/trpl-zh-cn/ch10-03-lifetime-syntax.html#静态生命周期)

`'static`，其生命周期**能够**存活于整个程序期间。所有的字符串字面值都拥有 `'static` 生命周期，我们也可以选择像下面这样标注出来：

```rust
let s: &'static str = "I have a static lifetime.";
```

### [结合泛型类型参数、trait bounds 和生命周期](https://kaisery.github.io/trpl-zh-cn/ch10-03-lifetime-syntax.html#结合泛型类型参数trait-bounds-和生命周期)

```rust
use std::fmt::Display;

fn longest_with_an_announcement<'a, T>(x: &'a str, y: &'a str, ann: T) -> &'a str
    where T: Display
{
    println!("Announcement! {}", ann);
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

这个是那个返回两个字符串 slice 中较长者的 `longest` 函数，不过带有一个额外的参数 `ann`。`ann` 的类型是泛型 `T`，它可以被放入任何实现了 `where` 从句中指定的 `Display` trait 的类型。这个额外的参数会在函数比较字符串 slice 的长度之前被打印出来，这也就是为什么 `Display` trait bound 是必须的。因为生命周期也是泛型，所以生命周期参数 `'a` 和泛型类型参数 `T` 都位于函数名后的同一尖括号列表中。




===================================
## RUST与C的对比



### Q1
下面的代码片段是否有漏洞？如果有，请指出
```
#define SIZE 6
uint8_t* pointer = (uint8_t*) malloc(SIZE); 
for(int i = 0 ; i < SIZE ; ++i) {
    pointer[i] = i; 
}

```

A1：

空指针引用（NULL Dereference）。如果开发者忘记了检查所返回的指针是否正确性，就可能会导致空指针引用。
```
let my_var: u32 = 42;
let my_ref: &u32 = &my_var; // <-- This is a reference. References ALWAYS point to valid data!
let my_var2 = *my_ref; // <-- An example for a Dereference. 
```
在Rust中，Rust处理这类指针错误的方式非常极端，在“安全”代码中粗暴简单地禁用所有裸指针。此外在“安全”代码中，Rust还取消了空值。
不过不用担心，Rust中存在一个优雅的替代方案——引用和借贷的方式。本质上来说，这些引用（references）还是那些老指针，但有了生命周期（Lifetimes）和借贷（Borrowing）规则，系统就能确保代码的安全性。


### Q2
下面的代码片段是否有漏洞？如果有，请指出
```
uint8_t* pointer = (uint8_t*) malloc(SIZE);

...

if (err) {
  abort = 1;
  free(pointer);
}

...

if (abort) {
  logError("operation aborted before commit", pointer);
}

```
A2：

释放内存后再使用（Use After Free）,会产生严重的漏洞，导致黑客随意操控你的代码。
```
fn foobar() {
    let foo = Hashmap::new();
^  
|  
|   {
|   let bar = Vec::new();
|   ^
|   |
|   | 
|   |
|   |
|   V
|   } // `bar` will be freed once we get here
|  
V  
} // `foo` will be freed once we get here
```
在Rust中，Rust也使用资源获取即初始化（Resource Acquisition Is Initialization）的方式，这意味着每个变量在超出范围后都一定会被释放，因此在“安全的”Rust代码中，永远不必担心释放内存的事情。
但Rust不满足于此，它更进一步，直接禁止用户访问被释放的内存。这一点通过Ownership规则实现，
在Rust中，变量有一个所有权（Ownership）属性，owner有权随意调用所属的数据，也可以在有限的lifetime内借出数据（即Borrowing）。

此外，数据只能有一个owner，这样一来，通过RAII规则，owner的范围指定了何时释放数据。最后，ownership还可以被“转移”，当开发者将ownership分配给另一个不同的变量时，ownership就会转移。



### Q3：
下面的代码片段是否有漏洞？如果有，请指出
```
uint8_t* get_dangling_pointer(void) {
    uint8_t array[4] = {0};
    return &array[0];
}
```

A3：
返回悬空指针（Dangling Pointers）。C语言老手都知道，向stack-bound变量返回指针很糟糕， 返回的指针会指向未定义内存。虽然这类错误多见于新手，一旦习惯堆栈规则和调用惯例，就很难出现这类错误了。

在Rust中

事实证明，Rust的lifetime check不仅适用于本地定义变量，也适用于返回值。

与C语言不同，在返回reference时，Rust的编译器会确保相关内容可有效调用，也就是说，编译器会核实返回的reference有效。即Rust的reference总是指向有效内存。
```
fn get_dangling_pointer() -> &u8 {
    let array = [0; 4];
    &array[0]
}

// main.rs:1:30: 1:33 error: missing lifetime specifier [E0106]
// main.rs:1 fn get_dangling_pointer() -> &u8 {
//    
```

#### Q4：
下面的代码片段是否有漏洞？如果有，请指出
```
void print_out_of_bounds(void) {
    uint8_t array[4] = {0};
    printf("%u\r\n", array[4]);
}

```

A4:

超出访问范围（Out Of Bounds Access）
另一个常见问题就是在访问时，访问了没有权限的内存，多半情况就是所访问的数组，其索引超出范围。这种情况也出现在读写操作中，访问超限内存会导致可执行文件出现严重的漏洞，这些漏洞可能会给黑客操作你的代码大开方便之门。

在Rust中，在这种情况下，Rust利用运行时检查以减少这种不必要的行为，非常方便。
fn print_panics() {
    let array = [0; 4];
    println!("{}", array[4]);
}

// thread '<main>' panicked at 'index out of bounds: the len is 4 but the index is 4', main.rs:3



总结：

如果你曾经编写过任何规模大小的程序，你可能会遇到各种错误。你在编码时产生的微小的错误会导致你的程序执行失败。程序越复杂，发生错误的概率越高！

为了修复和防止错误，有很多方法可供程序员使用。其中一个是在运行程序之前确定程序的正确性：静态类型。这种技术是设计编程语言的一部分，并且可以防止简单的错误，例如尝试使用字符串作为整数，或者比较类型不同的对象，例如 Car 和 Book。

我的个人观点是，编程语言及其实现应该尽可能地捕获程序员所犯的错误，从而使得他们能构建更好更安全的软件。虽然静态类型使得语言更加复杂和难以学习，但它为程序员提供了一个安全的机制，我相信这是非常值得的。

### 参考：
- 用Rust解决C语言的隐患  https://blog.csdn.net/qiansg123/article/details/80127743
- Rust 语言如何帮助你防止bug   https://www.oschina.net/translate/how-rust-helps-you-prevent-bugs?print
- Reddit讨论：CppCon 2017 - 在Facebook上反复出现的 C++ bug https://cloud.tencent.com/developer/article/1489383
- C++工程师的Rust迁移之路 系列 https://zhuanlan.zhihu.com/p/75385189

