# 未知值

由于在约束生成期间接受的表达式最多只能是二次的，因此在编译时对未知值的使用，施加了某些检查和条件。

在circom中，常数值和模板参数总是被认为是已知的，而信号总是被认为是未知的。

仅依赖于已知的表达式被视为已知，而依赖于某些未知的表达式被视为未知。

```rust
pragma circom 2.0.0;

template A(n1, n2){ // known
   signal input in1; // unknown
   signal input in2; // unknown
   var x = 0; // known
   var y = n1; // known
   var z = in1; // unknown
}

component main = A(1, 2);
```

在上面的代码中，模板参数n1、n2和常量值0被认为是已知的，因此，变量x和y也被认为是已知的。

同时，信号in1,in2被认为是未知的。因此，该变量z也被认为是未知的。

## 数组

带有数组访问的约束必须是已知的访问位置。

```rust
pragma circom 2.0.0;

template A(n){
   signal input in;
   signal output out;
   var array[n];

   out <== array[in];
   // Error: Non-quadratic constraint was detected statically, using unknown index will cause the constraint to be non-quadratic
}

component main = A(10);
```

在上面的代码中，使用已知大小的值定义数组n（因为模板参数始终被认为是已知的），而约束设置为依赖于未知位置处的数组元素in（因为信号始终被认为是未知的）。

数组还必须定义为已知大小。

```rust
pragma circom 2.0.0;

template A(){
   signal input in;
   var array[in];
   // Error: The length of every array must known during the constraint generation phase
}

component main = A();
```

在上面的代码中，数组被定义为具有未知大小的值in（因为信号始终被认为是未知的）。

## 控制流

如果if-else或for-loop块具有未知的条件，则该块被认为是未知的并且在其内部不能生成约束。因此，约束只能在已知条件的控制流中产生。

以 if-then 语句为例：

```rust
pragma circom 2.0.0;

template A(){
   signal input in;
   signal output out;

   if (in < 0){
   // Error: There are constraints depending on the value of the condition and it can be unknown during the constraint generation phase
       out <== 0;
   }
}

component main = A();
```

在上面的代码中，在 if-then 语句中定义了一个约束，其中的比较条件涉及未知值in（因为信号始终被认为是未知的）。

同样，以for循环为例：

```rust
pragma circom 2.0.0;

template A(){
   signal input in;
   signal output out;

   for (var i = 0; i < in; i++){
   // Error: There are constraints depending on the value of the condition and it can be unknown during the constraint generation phase
       out <== i;
   }
}

component main = A();
```

在上面的代码中，在 for 循环中定义了一个约束，其中计数条件为未知值in（因为信号始终被认为是未知的）。

有关其他详细信息，请参阅[控制流](../controlFlow.md)。
