+++
title = "Kaleidoscope-finally"
date = "2026-08-18"

[taxonomies]
tags = ["Kaleidoscope", "llvm"]
+++

最近终于完成了 LLVM Kaleidoscope 教程的全部内容。最后在给自己实现的 Kaleidoscope 编译器添加 Debug Info 时，遇到了一个比较奇怪的问题。

生成的 LLVM IR 执行结果完全正确，IR 本身看起来也没有明显问题，但是使用`gdb`调试时，`step`进入函数后显示的函数参数值是错误的。

最开始怀疑过：

* `DILexicalBlock` 缺失；
* `DILocation` 设置不正确；

于是使用 C 语言实现了逻辑完全相同的函数，再交给 `clang` 编译进行对比，发现 C 版本一切正常，llvm ir的指令也基本相同。
继续调试后又发现了一个更奇怪的现象：使用`break`打断点，停止时显示的函数参数一切正常。但是使用`step`进入函数时，pc停在了第一个指令，此时gdb显示的参数值是错的。

对比两种情况发现，`break`停止的位置已经是函数 prologue 执行了几条指令之后，而 `step` 则停得更靠前，基本都是第一个指令。也就是说，两种进入函数的停止位置不同，进而导致 gdb 对参数位置的判断也不同。

因为一直没找到原因，于是又换成 `lldb` 进行测试，结果 `lldb` 一切正常。

然后询问ai,说可能是“prologue_end / epilogue_begin 位置错误”。但是使用`llvm-dwarfdump --debug-line`查看，发现位置也正确。

最后没辙，逐行把 Kaleidoscope 生成的 LLVM IR 和 `clang` 编译 C 代码生成的 LLVM IR 进行对比，发现function attribute 不一样，试了一下全部拿过来，测试就发现好了。然后一个个删除，发现就是`"frame-pointer"="all"`这个属性加上就正常了。

后面又折腾了很久，想找到根本原因。但是，没找到。目前只能推测可能`llvm`和`gdb`的`step`进入函数停止的指令的定位方式可能还是存在某些不一致吧。

目前能够确认的是：
```
frame-pointer=default
    ↓
生成 RSP-based stack frame
    ↓
GDB step 进入函数时停止位置异常
    ↓
参数显示错误
```

```
frame-pointer=all
    ↓
生成 RBP-based stack frame
    ↓
GDB step 正常
    ↓
参数显示正常
```

因此目前只能推测：

`lldb`与`gdb` 在 `step`进入函数时确定停止位置的机制之间，可能存在某种不一致。导致`lldb`正常，`gdb`在某些边界条件可能有问题，而我刚好触发了。🤔
