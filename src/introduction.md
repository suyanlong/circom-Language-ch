# 简介

Circom是一种新颖的领域特定语言，用于定义可用于生成零知识证明的算术电路。Circom compiler是一个用 Rust 编写的 circom 语言编译器，可用于生成带有一组关联约束的 R1CS 文件和一个程序（用 C++ 或 WebAssembly 编写），以有效计算对电路所有线路的有效分配。它的主要特点之一circom是它的模块化，它允许程序员定义称为模板的可参数化电路，可以将其实例化以形成更大的电路。利用小型独立组件构建电路的想法使得测试、审查、审计或正式验证大型复杂circom电路变得更加容易。在这方面，circom用户可以创建自己的自定义模板或从circomLib实例化模板，这是一个公开可用的库，带有数百个电路，例如比较器、哈希函数、数字签名、二进制和十进制转换器等等。Circcomlib 向从业者和开发人员公开提供。

证明系统的实现也可以在我们的库中找到，包括用 Javascript 和 Pure Web Assembly 编写的snarkjs、用本机 Web Assembly 编写的wasmsnark 、用 C++ 和 Intel Assembly 编写的rapidSnark 。

Circcom 旨在为开发人员提供一个整体框架，通过易于使用的接口构建算术电路，并抽象化证明机制的复杂性。
