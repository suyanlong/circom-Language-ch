# 导入

与其他代码一样，模板可以在其他文件（例如库）中找到。为了使用其他文件中的代码，我们必须使用关键字 include 以及相应的文件名（默认扩展名为 .circom）将它们包含在我们的程序中。

```rust
include "montgomery.circom";
include "mux3.circom";
include "babyjub.circom";
```

这段代码包含了circom库中的三个文件：montgomery.circom、mux3.circom、babyjub.circom。

从 circom 2.0.8 开始，可以使用-l命令选项来指示搜索要包含的文件的路径。

