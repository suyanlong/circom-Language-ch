# 作用域

Circcom 与 C 和 Rust 一样具有静态作用域。然而，我们认为信号和组件必须具有全局作用域，因此它们应该在定义它们模板的顶级块中。

```rust
pragma circom 2.0.0;

template Multiplier2 (N) {
   //Declaration of signals.
   signal input in;
   signal output out;

   //Statements.
   out <== in;
   signal input x;
   if(N > 0){
    signal output out2;
    out2 <== x;
   }
}

component main = Multiplier2(5);
```

信号out2必须在顶级块中声明。产生一个编译错误：“out2 is outside the initial scope”。

从 circom 2.1.5 开始，信号和组件现在可以在if块内声明，但前提是在编译时是已知条件。

```rust
pragma circom 2.1.5;
template A(n){
   signal input in;
   signal output outA;
   var i = 0;
   if(i < n){
    signal out <== 2;
    i = out;
   } 
   outA <== i;
}
component main = A(5);
```

在前面的示例中，i < n条件在编译时已知，然后允许声明out信号。如果条件 in < n，在编译时是未知的，我们会输出错误，因为这种情况下的声明是不允许的。

关于可见性，组件c的信号x在声明了c的模板t中也是可见的，使用符号c.x，不允许访问嵌套子组件的信号。例如，如果c是使用另一个组件d构建的，则不能从t访问d的信号。这可以在下面的代码中看到:

```rust
pragma circom 2.0.0;

template d(){
  signal output x;
  x <== 1;
}

template c(){
  signal output out2;
  out2 <== 2;
  component comp2 = d();
}

template t(){
  signal out;
  component c3 = c();
  out <== c3.comp2.x;
}

component main = t();
```

此代码会产生编译错误，因为我们无法访问组件c3的comp2。

var可以在任何块中定义，并且它的可见性会降低到像 C 或 Rust 中那样的块。
