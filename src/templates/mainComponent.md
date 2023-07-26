# main入口

为了开始执行，必须给出一个初始组件。默认情况下，该组件的名称是“main”，因此需要使用某个模板实例化组件main。

这是创建电路所需的特殊初始组件，它定义了电路的全局输入和输出信号。因此，与其他组件相比，它有一个特殊的属性：公共输入信号列表。创建main组件的语法是：

```rust
component main {public [signal_list]} = tempid(v1,...,vn);
```

其中{public [signal_list]}是可选的。未包含在列表中的模板的任何输入信号都被视为私有。

```rust
pragma circom 2.0.0;

template A(){
    signal input in1;
    signal input in2;
    signal output out;
    out <== in1 * in2;
}

component main {public [in1]}= A();
```

在此示例中，我们有两个输入信号in1和in2。让我们注意，它in1已被声明为电路的公共信号，而由于in2它没有出现在列表中，因此被视为私有信号。最后，输出信号始终被视为公共信号。

不仅在正在编译的文件中，而且在程序中包含的任何其他 circom文件中，只能定义一个main组件，否则，编译失败并显示一条消息：“Multiple main components in the project structure”
