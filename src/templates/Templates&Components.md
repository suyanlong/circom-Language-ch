# 模板和组件

## 模板
在 Circcom 中创建通用电路的机制就是所谓的模板。

它们通常是在使用模板时必须实例化的某些值的参数。模板的实例化是一个新的电路对象，它可以用来组成其他电路，从而作为更大电路的一部分。由于模板通过实例化来定义电路，因此它们有自己的信号。

```rust
template tempid ( param_1, ... , param_n ) {
 signal input a;
 signal output b;

 .....

}
```

模板不能包含本地函数或模板定义。

在定义该值的同一模板内为输入信号赋值也会生成错误“Exception caused by invalid assignment”，如下一个示例所示。

```rust
pragma circom 2.0.0;

template wrong (N) {
 signal input a;
 signal output b;
 a <== N;
}

component main = wrong(1);

```

模板的实例化是使用关键字组件并提供必要的参数来完成的。

```rust
component c = tempid(v1,...,vn);
```

参数的值在编译时应该是已知的常量。下一个代码会生成以下编译错误消息：“Every component instantiation must be resolved during the constraint generation phase”。

```rust

pragma circom 2.0.0;

template A(N1,N2){
   signal input in;
   signal output out; 
   out <== N1 * in * N2;
}


template wrong (N) {
 signal input a;
 signal output b;
 component c = A(a,N); 
}

component main {public [a]} = wrong(1);
```

## 成分

一个组件定义了一个算术电路，因此它接收 N 个输入信号并产生 M 个输出信号和 K 个中间信号。此外，它还可以产生一组约束。

为了访问组件的输入或输出信号，我们将使用点表示法。组件外部看不到其他信号。

```rust
c.a <== y*z-1;
var x;
x = c.b;
```

在所有输入信号都分配给具体值之前，不会触发组件实例化。因此，实例化可能会被延迟，因此组件创建指令并不意味着组件对象的执行，而是意味着当所有输入都设置好后将完成实例化过程的创建。仅当设置了所有输入时才能使用组件的输出信号，否则会生成编译器错误。例如，下面的代码会导致错误：

```rust
pragma circom 2.0.0;

template Internal() {
   signal input in[2];
   signal output out;
   out <== in[0]*in[1];
}

template Main() {
   signal input in[2];
   signal output out;
   component c = Internal ();
   c.in[0] <== in[0];
   c.out ==> out;  // c.in[1] is not assigned yet
   c.in[1] <== in[1];  // this line should be placed before calling c.out
}

component main = Main();
```

组件是不可变的（如信号）。可以首先声明组件，然后在第二步中初始化组件。如果有多个初始化指令（在不同的执行路径中），它们都需要是同一模板的实例化（可能具有不同的参数值）。

```rust
template A(N){
   signal input in;
   signal output out;
   out <== in;
}

template C(N){
   signal output out;
   out <== N;
}
template B(N){
  signal output out;
  component a;
  if(N > 0){
     a = A(N);
  }
  else{
     a = A(0);
  }
}

component main = B(1);
```

如果将指令a = a(0);替换为a = C(0)，则编译失败，并显示下一条错误消息:“Assignee and assigned types do not match”。

我们可以按照之前给出的大小限制来定义**组件数组**。此外，不允许在组件数组的定义中进行初始化，并且只能逐个组件进行实例化，访问数组的位置。数组中的所有组件都必须是同一模板的实例，如下一个示例所示。

```rust
template MultiAND(n) {
    signal input in[n];
    signal output out;
    component and;
    component ands[2];
    var i;
    if (n==1) {
        out <== in[0];
    } else if (n==2) {
          and = AND();
        and.a <== in[0];
        and.b <== in[1];
        out <== and.out;
    } else {
        and = AND();
        var n1 = n\2;
        var n2 = n-n\2;
        ands[0] = MultiAND(n1);
        ands[1] = MultiAND(n2);
        for (i=0; i<n1; i++) ands[0].in[i] <== in[i];
        for (i=0; i<n2; i++) ands[1].in[i] <== in[n1+i];
        and.a <== ands[0].out;
        and.b <== ands[1].out;
        out <== and.out;
    }
}
```

当组件是独立的（输入不依赖于彼此的输出）时，这些部分的计算可以使用标签并行完成parallel，如下一行所示。

```rust
template parallel NameTemplate(...){...}
```

如果使用此标签，生成的 C++ 文件将包含计算见证的并行代码。在处理大型电路时，并行化变得尤为重要。

请注意，前面的并行性是在模板级别声明的。有时，声明每个组件的并行性可能很有用。从版本 2.0.8 开始，并行标签也可以在组件级别使用，并行标签在调用模板之前指示。

```rust
component comp = parallel NameTemplate(...){...}
```

用例的一个真实示例是汇总代码中的以下代码：

```rust
component rollupTx[nTx];
for (i = 0; i < nTx; i++) {
        rollupTx[i] = parallel RollupTx(nLevels, maxFeeTx);
}
```

需要再次强调的是，这种并行性只能在 C++ 见证生成器中利用。

## 自定义模板

从2.0.6版本开始，该语言允许定义一种新类型的模板，即自定义模板。这种新结构的工作方式与标准模板类似：它们的声明方式类似，只需在;custom之后的声明中添加关键字即可。template并以完全相同的方式实例化。即Example定义一个自定义模板，然后实例化如下：

```rust
pragma circom 2.0.6; // note that custom templates are only allowed since version 2.0.6
pragma custom_templates;

template custom Example() {
   // custom template's code
}

template UsingExample() {
   component example = Example(); // instantiation of the custom template
}
```

然而，它们的计算编码方式与标准模板的编码方式不同。snarkjs将在稍后阶段处理每个定义的自定义模板的使用，以生成和验证 zk 证明，而不是生成 r1cs 约束，在本例中使用 PLONK 方案（并使用自定义模板的定义作为 PLONK 的自定义门，请参阅这里如何）。有关自定义模板的定义和用法的信息将导出到文件中.r1cs（请参阅此处的第 4 节和第 5 节）。这意味着自定义模板不能在其主体内引入任何约束，也不能声明任何子组件。

