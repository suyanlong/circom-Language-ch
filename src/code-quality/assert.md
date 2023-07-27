# 断言

## assert(bool_expression)

该声明介绍了要检查的条件。在这里，我们根据bool_expression在编译时是否未知来区分两种情况：

* 如果断言语句依赖于仅具有已知条件的控制流（请参阅Unknowns）并且bool_expression已知（例如，如果它仅依赖于模板参数或字段常量的值），则在编译时评估断言。如果评估结果为 false，则编译失败。考虑下一段代码：

```rust
template A(n) {
  signal input in;
  assert(n>0);
  in * in === n;
}

component main = A(0);

```


这里，可以在编译期间评估断言，并且评估结果为假。因此，编译结束时会抛出错误error[T3001]: False assertreach。如果主要组件被定义为component main = A(2)，则编译正确完成。

* 否则，编译器会在最终见证生成代码中添加一个断言，该断言必须在见证生成期间得到满足。在下面的示例中，如果in作为参数传递的用于生成见证的输入不满足断言，则不会生成见证。

```rust
template Translate(n) {
  signal input in;  
  assert(in<=254);
  . . .
}
```

试想，当使用 === 引入类似 in * in === n; 这样的约束时，在见证生成代码中会自动添加一个 assert。在这种情况下类似于，assert(in * in == n)。
