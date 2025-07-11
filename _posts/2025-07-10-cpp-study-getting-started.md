---
title: (1) C++ 学习 - Getting Started
date: 2025-07-10 18:52:00 +0800
categories: [技术]
tags: [C++]
---

## Why C++?

* 在机器视觉领域, `Python` 可以解决绝大部分的问题, 但在资源受限的场景下, `C++` 依然是绕不开的基石.

* 在工业界, 确实可能存在资源不受限的场景, 但是真正走到行业前沿, 产品必然会走上通过优化以达到尽可能高的性能的路径, 而非单纯的实现而已.

* `So, C++.`.

## Why [Learn C++](https://www.learncpp.com/)?

* `Gemini 2.5 Pro` 推荐的😂, 其实材料不是关键, 9.1 分和 9.9 分的差异很小, 关键在于坚持.

## Background

- `CPU` 只能执行 `Machine Language`, 说白了就是 `0s` 和 `1s` 的组合😂.

- 不同的 `CPU` (如 `x86`, `arm64`, `...`) 有不同的 `Machine Language`, 它们是不兼容的.

- `assembly languages` 的引入, 是为了相对好的可读性, 但是 `CPU` 无法直接执行它, 需要通过 `assembler` 操作将 `assembly languages` 转换为 `Machine Language` 再执行. 因为目的只是为了可读性, 所以在 `assembly languages` 和 `Machine Language` 是一一对应的, 因此 `assembler` 的过程很直观. 同时也就可以知道, 不同的 `CPU` (如 `x86`, `arm64`, `...`) 有不同的 `assembly languages`, 它们是不兼容的😂.

* `Machine Language` 和 `assembly languages` 的特点就是快🚀.

* 一些高级编程语言 `C/C++, C#...` 是编译型语言, 需要使用 `compiler` 将它们转换为 `executable`, 实际就是 `Machine Language`, 它们的分发不需要其它任何依赖, 且运行效率更佳. 但是需要针对不同平台编译, 因此移植复杂度相对较高. 

* 另一些高级编程语言 `Python, JavaScript` 是解释型语言, 需要使用 `interpreter` 来实时将它们的代码转换为 `Machine Language`, 因此它们的分发需要 `interpreter` 的存在, 而且往往以暴露源代码为代价. 但是有因为 `interpreter` 的存在, 同一代码各个平台都可以执行, 而且省去了编译的过程, 可以进行实时修改. 当然效率会差一些, `interpreter` 也会带来多余的资源占用.

* `C/C++` 的设计哲学: `trust the programmer`.

## C++ 开发流程

1. **Define the problem that you would lick to solve.**

    不定义问题, 就无法解决问题, 清晰地定义问题是一切的基础.

2. **Determine how you are going to solve the problem.**

    通过对问题进行思考, 梳理出一些可能的方案, 然后进行评估, 选择最合适的方案, 好的方案有如下特点:
    * 符合直觉, 没有弯弯绕绕, 简单, 容易理解.
    * 写了很棒的文档, 特别是这个方案存在一些需要特别说明的内容的话(比如假设和限制).
    * 进行了模块化, 对复用和未来的变更做了考虑.
    * 对异常进行了相对完备的处理.

3. **Write the program.**

4. **Compiling your source code.**

    在静态检查以后, 把每个 `.cpp` 文件编译为 `.o` 文件, One by one.

5. **Linking object files and libraries and creating the desired output file.**

    把 `.o` 文件, 连同引用的标准库和三方库, 最终生成一个 Executable file, 或者 library files.

6. **Testing.**

7. **Debugging.**

## 一些关于 IDE 的最佳实践

- 设置 C++ Language Stardard 为特定的版本(如 ISO C++ 17 Standard(/std:c++17)), 以保证一致的行为.
- 设置 Conformance mode 为 Yes(/permissive-), 以防止当前编译器的额外特性破坏代码在其它编译器上的一致性.
- 设置 Warning Level 为 Level4(/W4), 使用最高的警告等级, 以严格要求代码的质量.
- 设置 Treat Warnings As Errors 为 Yes(/WX), 即出现编译器警告时, 代码不会继续编译. 以严格要求代码的质量.

## The End

本节主要是一些基础性的内容.
