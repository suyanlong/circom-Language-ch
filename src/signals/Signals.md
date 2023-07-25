# 信号和变量

使用circom构建的算术电路对包含Z/pZ域中的信号进行操作。信号可以使用标识符命名，也可以存储在数组中并使用关键字signal进行声明。信号可以定义为输入或输出，否则被视为中间信号。

```rust
signal input in;
signal output out[N];
signal inter;
```

这个小例子声明了一个带有标识符的输入信号in、一个带有标识符的输出信号的N维数组out以及一个带有标识符的中间信号inter。

信号始终被认为是私有的。只有在定义主要组件时，通过提供公共输入信号列表，程序员才能区分公共信号和私有信号。

```rust
pragma circom 2.0.0;

template Multiplier2(){
   //Declaration of signals
   signal input in1;
   signal input in2;
   signal output out;
   out <== in1 * in2;
}

component main {public [in1,in2]} = Multiplier2();
```

从circom2.0.4开始，还允许在声明后立即初始化中间和输出信号。那么，前面的例子可以重写如下：

```rust
pragma circom 2.0.0;

template Multiplier2(){
   //Declaration of signals
   signal input in1;
   signal input in2;
   signal output out <== in1 * in2;
}

component main {public [in1,in2]} = Multiplier2();
```

此示例将主要组件的输入信号in1和声明为公共信号。in2

在这种情况下，主组件的所有输出信号都是公共的（并且不能设为私有），如果没有另外声明，则主组件的输入信号是私有的，使用上面的关键字public。其余信号都是私有的，不能公开。

因此，从程序员的角度来看，只有公共输入和输出信号从电路外部可见，因此不能访问中间信号。

```rust
pragma circom 2.0.0;

template A(){
   signal input in;
   signal outA; //We do not declare it as output.
   outA <== in;
}

template B(){
   //Declaration of signals
   signal output out;
   component comp = A();
   out <== comp.outA;
}

component main = B();
```

此代码会产生编译错误，因为signal outA未声明为输出信号，因此无法访问它并将其分配给signal out。

信号是不可变的，这意味着一旦为它们分配了值，该值就不能再更改。因此，如果一个信号被分配两次，就会产生编译错误。这可以在下一个示例中看到，其中信号out被分配两次，从而产生编译错误。

```rust
pragma circom 2.0.0;

template A(){
   signal input in;
   signal output outA; 
   outA <== in;
}

template B(){
   //Declaration of signals
   signal output out;
   out <== 0;
   component comp = A();
   comp.in <== 0;
   out <== comp.outA;
}

component main = B();
```

在编译时，信号的内容始终被视为未知（请参阅Unknowns），即使已经为其分配了常量。这样做的原因是为了提供一个精确的定义哪些结构是允许的，哪些结构是不允许的，而不依赖于编译器检测信号是否始终具有常量值的能力。

```rust
pragma circom 2.0.0;

template A(){
   signal input in;
   signal output outA; 
   var i = 0; var out = 0;
   while (i < in){
    out++; i++;
   }
   outA <== out;
}

template B(){
 component a = A();
 a.in <== 3;
}

component main = B();
```

此示例会产生编译错误，因为signal的值outA取决于signal的值in，尽管该值是常量3。

信号只能使用操作<--或<==（请参阅基本运算符）与左侧的信号和-->或==>（请参阅基本运算符）与右侧的信号进行分配。安全选项是<==和==>，因为它们分配值的同时还生成约束。一般来说，使用<--and是危险的，只有当分配的表达式不能包含在约束中时才应使用，如下例所示。-->

```rust
out[k] <-- (in >> k) & 1;
```


