# 函数

在circom中，函数定义了通用的抽象代码片段，可以执行一些计算以获得要返回的值或表达式。

```rust
function funid ( param1, ... , paramn ) {

 .....

 return x;
}
```

函数计算数值（或数组）值或表达式。函数可以是递归的。考虑circom 库中的下一个函数。

```rust
/*
    This function calculates the number of extra bits 
    in the output to do the full sum.
 */

function nbits(a) {
    var n = 1;
    var r = 0;
    while (n-1<a) {
        r++;
        n *= 2;
    }
    return r;
}
```

函数不能声明信号或生成约束（如果需要，请使用模板）。下面这个个函数会生成错误消息：“Template operator found”。

```rust
function nbits(a) {
    signal input in; //This is not allowed.
    var n = 1;
    var r = 0;
    while (n-1<a) {
        r++;
        n *= 2;
    }
    r === a; //This is also not allowed.
    return r;
}
```

通常，可以有许多返回语句，但是每个执行跟踪必须以返回语句结束(否则将产生编译错误)。return语句的执行将控制返回给函数的调用者。

```rust
function example(N){
     if(N >= 0){ return 1;}
//   else{ return 0;}
}
```

编译example函数会产生下一条错误消息：“In example there are paths without return”。
