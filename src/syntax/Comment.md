# 注释

在 circom 中，您可以在源代码中添加注释。这些注释行将被编译器忽略。注释可以帮助程序员阅读源代码以更好地理解它。强烈推荐向代码添加注释。

circom 2.0 中允许的注释行与其他编程语言（如 C 或 C++）类似。

## 单行注释

您可以使用以下命令在单行上编写注释：

```rust
//Using this, we can comment a line.
```

您还可以使用以下命令在代码行末尾编写注释:

```rust
template example(){
    signal input in;   //This is an input signal.
    signal output out; //This is an output signal.
}
```


## 多行注释

您可以使用和编写跨多行的注释:

```rust
/*
All these lines will be 
ignored by the compiler.
*/
```
