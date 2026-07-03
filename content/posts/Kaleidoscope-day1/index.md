+++
title = "Kaleidoscope-day1"
date = "2026-07-03"

[taxonomies]
tags = ["Kaleidoscope", "llvm"]
+++

阅读了Kaleidoscope llvm教程的第一二章，大概初步了解了语言的部分词法和语法定义。


总结一下, 不完整的定义：
```
S: definition | external | expr | ';'
definition: 'def' prototype expr
external: 'extern' prototype 
prototype: IDENTIFIER '(' IDENTIFIER? (',' IDENTIFIER)* ')'
expr: primary | binaryExpr
primary: NUMBER | '(' expr ')' | IDENTIFIER ('(' expr? (',' expr)* ')')?
binaryExpr: primary ops expr
IDENTIFIER:  LETTER (LETTER | DIGIT)*
NUMBER: DIGIT+  ( '.' DIGIT+)?
LETTER:  [a-zA-Z_]
DIGIT: [0-9]
```
