### 函数参数何时通过值传递？

与C系列中的所有语言一样，Go中的所有内容都按值传递。也就是说，函数总是获取正在传递的东西的副本，就好像有一个赋值语句将值赋给参数。例如，将`int`值传递给函数会生成一个副本`int`，并且传递指针值会生成指针的副本，但不会生成它指向的数据。（有关如何影响方法接收器的讨论，请参阅[后面的部分](http://docs.studygolang.com/doc/faq#methods_on_values_or_pointers)。）

映射和切片值的行为类似于指针：它们是包含指向底层映射或切片数据的指针的描述符。复制地图或切片值不会复制它指向的数据。复制接口值会生成存储在接口值中的事物的副本。如果接口值包含结构，则复制接口值会生成结构的副本。如果接口值包含指针，则复制接口值会生成指针的副本，但同样不会指向它指向的数据。

请注意，此讨论是关于操作的语义。只要优化不改变语义，实际实现可以应用优化以避免复制。

### 为什么T和* T有不同的方法集？

正如[Go规范](http://docs.studygolang.com/ref/spec#Types)所说，类型的方法集`T`包含所有具有接收器类型的方法`T`，而相应指针类型`*T`的方法包含所有带接收器`*T`或 方法的方法`T`。这意味着方法集`*T` 包括`T`但不反过来。

之所以出现这种区别是因为如果接口值包含指针`*T`，则方法调用可以通过取消引用指针来获取值，但是如果接口值包含值`T`，则方法调用无法获得指针的安全方法。（这样做会允许方法修改接口内部值的内容，这是语言规范所不允许的。）

即使在编译器可以将值的地址传递给方法的情况下，如果方法修改了值，则更改将在调用者中丢失。作为一个例子，如果该`Write`方法 [`bytes.Buffer`](http://docs.studygolang.com/pkg/bytes/#Buffer) 中使用的值接收器，而不是一个指针，这样的代码：

```
var buf bytes.Buffer 
io.Copy（buf，os.Stdin）
```

将标准输入复制到*副本*中`buf`，而不是*复制*到`buf`自身中。这几乎不是理想的行为。