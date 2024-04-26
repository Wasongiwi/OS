# Note02

1. ```rust
   fn main() {
       let s1 = String::from("hello,");
       let s2 = String::from("world!");
       // 在下句中，s1的所有权被转移走了，因此后面不能再使用s1
       let s3 = s1 + &s2;
       assert_eq!(s3,"hello,world!");
       // 下面的语句如果去掉注释，就会报错
       // println!("{}",s1);
   }
   //不转移所有权
   fn main() {
       let s1 = "hello";
       let s2 = String::from("rust");
       let s = format!("{} {}!", s1, s2);
       println!("{}", s);
   }
   
   ```

   * 第一个字符串的所有权会被转移（移动）到 `+` 运算符的实现中，因为 `+` 运算符的实现需要获取第一个字符串的所有权。
   * 第二个字符串的引用（`&str`）会被传递给 `+` 运算符的实现，因为 `+` 运算符的实现需要使用第二个字符串的内容来完成字符串连接操作。
   * 所以，在 `let s3 = s1 + &s2;` 这一行中，`s1` 的所有权被转移给了 `+` 运算符的实现，而 `&s2` 是 `s2` 的引用，可以被借用给 `+` 运算符的实现使用，而 `s2` 本身并没有被转移。
   * 通过将第二个字符串的引用传递给 `+` 运算符的实现，我们可以避免将第二个字符串的所有权转移给 `+` 运算符，从而在 `+` 运算完成后仍然可以使用第二个字符串。

2. 就字符串字面值来说，我们在编译时就知道其内容，最终字面值文本被直接硬编码进可执行文件中，这使得字符串字面值快速且高效，这主要得益于字符串字面值的不可变性。不幸的是，我们不能为了获得这种性能，而把每一个在编译时大小未知的文本都放进内存中（你也做不到！），因为有的字符串是在程序运行的过程中动态生成的。

3. 对于 `String` 类型，为了支持一个可变、可增长的文本片段，需要在堆上分配一块在编译时未知大小的内存来存放内容，这些都是在程序运行时完成的：

   - 首先向操作系统请求内存来存放 `String` 对象：由 `String::from` 完成，它创建了一个全新的 `String`
   - 在使用完成后，将内存释放，归还给操作系统
4. 变量在离开作用域后，就自动释放其占用的内存：自动调用 `drop` 函数

   ```rust
   {
       let s = String::from("hello"); // 从此处起，s 是有效的
   
       // 使用 s
   }                                  // 此作用域已结束，
                                      // s 不再有效，内存被释放
   ```

5. 数组

   ```rust
   #[derive(Debug)]
    struct File {
      name: String,
      data: Vec<u8>,
    }
   
    fn main() {
      let f1 = File {
        name: String::from("f1.txt"),
        data: Vec::new(),
      };
   
      let f1_name = &f1.name;
      let f1_length = &f1.data.len();
   
      println!("{:?}", f1);
      println!("{} is {} bytes long", f1_name, f1_length);
    }
    
   //File { name: "f1.txt", data: [] }
   //f1.txt is 0 bytes long
   ```

   * `File` 结构体两个字段 `name` 和 `data` 分别拥有底层两个 `[u8]` 数组的所有权(`String` 类型的底层也是 `[u8]` 数组)，通过 `ptr` 指针指向底层数组的内存地址，这里你可以把 `ptr` 指针理解为 Rust 中的引用类型。
   * 侧面印证了：**把结构体中具有所有权的字段转移出去后，将无法再访问该字段，但是可以正常访问其它的字段**。
   * 如果你想在结构体中使用一个引用，就必须加上生命周期，否则就会报错

6. 枚举：Rust决定抛弃 `null`，而改为使用 `Option` 枚举变量：

   1. `Option` 枚举包含两个成员，一个成员表示含有值：`Some(T)`, 另一个表示没有值：`None`，定义如下：其中 `T` 是泛型参数，`Some(T)`表示该枚举成员的数据类型是 `T`，换句话说，`Some` 可以包含任何类型的数据。

      ```rust
      enum Option<T> {
          Some(T),
          None,
      }
      while let Some(top) = stack.pop() {
          println!("{}", top);
      }
      ```
7. 数组 

   ```rust
   fn main() {
     // 编译器自动推导出one的类型
     let one             = [1, 2, 3];
     // 显式类型标注
     let two: [u8; 3]    = [1, 2, 3];
     let blank1          = [0; 3];
     let blank2: [u8; 3] = [0; 3];
   
     // arrays是一个二维数组，其中每一个元素都是一个数组，元素类型是[u8; 3]
     let arrays: [[u8; 3]; 4]  = [one, two, blank1, blank2];
   
     // 借用arrays的元素用作循环中
     for a in &arrays {
       print!("{:?}: ", a);
       // 将a变成一个迭代器，用于循环
       // 你也可以直接用for n in a {}来进行循环
       for n in a.iter() {
         print!("\t{} + 10 = {}", n, n+10);
       }
   
       let mut sum = 0;
       // 0..a.len,是一个 Rust 的语法糖，其实就等于一个数组，元素是从0,1,2一直增加到到a.len-1
       for i in 0..a.len() {
         sum += a[i];
       }
       println!("\t({:?} = {})", a, sum);
     }
   }
   ```

8. 循环控制

   1. for

      ```rust
      // 第一种
      let collection = [1, 2, 3, 4, 5];
      for i in 0..collection.len() {
        let item = collection[i];
        // ...
      }
      
      // 第二种
      for item in collection {
      
      }
      
      ```

   2. | 使用方法                      | 等价使用方式                                      | 所有权     |
      | ----------------------------- | ------------------------------------------------- | ---------- |
      | `for item in collection`      | `for item in IntoIterator::into_iter(collection)` | 转移所有权 |
      | `for item in &collection`     | `for item in collection.iter()`                   | 不可变借用 |
      | `for item in &mut collection` | `for item in collection.iter_mut()`               | 可变借用   |

9. 模式匹配

   1. match

      ```
      enum Direction {
          East,
          West,
          North,
          South,
      }
      
      fn main() {
          let dire = Direction::South;
          match dire {
              Direction::East => println!("East"),
              Direction::North | Direction::South => {
                  println!("South or North");
              },
              _ => println!("West"),
          };
      }
      ```

      * 通过将 `_` 其放置于其他分支后，`_` 将会匹配所有遗漏的值。`()` 表示返回**单元类型**与所有分支返回值的类型相同，所以当匹配到 `_` 后，什么也不会发生。


