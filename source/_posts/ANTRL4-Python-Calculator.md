---
title: 用 ANTLR4 和 python 十多行代码写一个计算器
date: 2020-04-16 18:53:48
tags:
	- ANTLR4
	- Python
	- 编译原理
---

## background

用 ANTLR4 和 Python 写了个简单的计算器，代码在这里👉 [keyi6/calculANTLR-python3](https://github.com/keyi6/calculANTLR-python3)。

本文可以是一个 ANTLR4 的 [Hello World](https://wizardforcel.gitbooks.io/antlr4-short-course/content/getting-started.html) 的进阶。

如果很想使用 C++，可以参考这个 repo [empirical-soft/calculANTLR](https://github.com/empirical-soft/calculANTLR)。

## introduction

ANTLR4(ANother Tool for Language Recognition) 是一款功能强大的语法分析器生成器，可以用来读取、处理、执行和转换结构化文本或二进制文件。它被广泛应用于学术界和工业界构建各种语言、工具和框架。

![pic](https://wizardforcel.gitbooks.io/antlr4-short-course/content/images/basic-data-flow.png)

ANTLR 有一套自己规定的定义语法的语法（⚠️禁止套娃），后缀为 `.g` 或者 `.g4`。通过这个文件，ANTLR 可以生成对应的 parser 和 lexer。并且 ANTLR 支持很多语言，所以用好 ANTLR 就能简单愉快的写一个编译器前端！

## 实现思路

[安装流程](https://www.antlr.org/)不在此处赘述。

### step1. 编写 `Calculantlr.g4`

```ANTLR
grammar Calculantlr;

// non-terminals expressed as context-free grammar (BNF)
expr:	left=expr op=('*'|'/') right=expr  # OpExpr
    |	left=expr op=('+'|'-') right=expr  # OpExpr
    |	atom=INT                           # AtomExpr
    |	'(' expr ')'                       # ParenExpr
    ;

// tokens expressed as regular expressions
INT : [0-9]+ ;
```



### step2. 用 ANTLR 来生成  parser 和 lexer

```bash
antlr4 -Dlanguage=Python3 Calculantlr.g4 -visitor -o dist 
```

其中 `-Dlanguage=Python3`指定生成语言是 python；`-visitor` 是用来遍历 AST 用的，`-o` 指定输出的目录。

这一步会生成下面几个文件👇

```
dist
├── Calculantlr.interp
├── Calculantlr.tokens
├── CalculantlrLexer.interp
├── CalculantlrLexer.py
├── CalculantlrLexer.tokens
├── CalculantlrListener.py
├── CalculantlrParser.py
└── CalculantlrVisitor.py
```



### step3. 如何使用生成的  parser 和 lexer

看官方关于 python 的文档只有[这么一点点](https://github.com/antlr/antlr4/blob/master/doc/python-target.md#how-do-i-run-the-generated-lexer-andor-parser)，不过依葫芦画瓢，这四句就建好了一个 AST 树。

```python
from antlr4 import *
from dist.CalculantlrLexer import CalculantlrLexer
from dist.CalculantlrParser import CalculantlrParser

#...

# lexing
lexer = CalculantlrLexer(input_stream)
stream = CommonTokenStream(lexer)

# parsing
parser = CalculantlrParser(stream)
tree = parser.expr()
```



### step4. 通过 Visitor 来遍历 AST

Visitor 的具体介绍可以看[官方文档](https://wizardforcel.gitbooks.io/antlr4-short-course/content/calculator-visitor.html)。看 `dist/CalculantlrVisitor.py`，这个 `CalculantlrVisitor` 继承了 `ParseTreeVisitor`，并且有：

- `visitAtomExpr`
- `visitParenExpr`
- `visitOpExpr`

这个几个函数，分别对应了 `Calculantlr.g4` 的那几条规则（命名都一样）。

为了计算结果，我写了一个 `CalcVisitor` 继承自 `CalculantlrVisitor`，然后重写上面的三个函数，来实现表达式计算的功能。



#### `visitAtomExpr`

这个函数对应的👇这条语法。

```ANTLR
expr:	...
    |	atom=INT                           # AtomExpr
```

要计算 AST 这个节点的值很简单，直接返回就好了。

```python
def visitAtomExpr(self, ctx:CalculantlrParser.AtomExprContext):
    return int(ctx.getText())
```



#### `visitParenExpr`

这个函数对应的👇这条语法。

```ANTLR
expr:	...
    |	'(' expr ')'                       # ParenExpr
```

要计算 AST 这个节点的值也很简单，不管括号，直接返回 `expr` 的值就好了。

```python
def visitParenExpr(self, ctx:CalculantlrParser.ParenExprContext):
    return self.visit(ctx.expr())
```



#### `visitOpExpr`

这个函数对应的👇这两条语法。

```ANTLR
expr:	left=expr op=('*'|'/') right=expr  # OpExpr
    |	left=expr op=('+'|'-') right=expr  # OpExpr
    ...
```

思路是，计算左边，计算右边，然后计算自己并返回。

```python
def visitOpExpr(self, ctx:CalculantlrParser.OpExprContext):
    l = self.visit(ctx.left)
    r = self.visit(ctx.right)

    op = ctx.op.text
    if op == '+': return l + r
    elif op == '-': return l - r
    elif op == '*': return l * r
    elif op == '/': return l / r # 这里需要做个不被 0 除的保护
```



现在 `CalcVisitor` 这个类就写完了。这样调用，就能获得表达式的值了！

```python
visitor = CalcVisitor()
ans = visitor.visit(tree)
```



### step5 大功告成！

![demo](https://raw.githubusercontent.com/keyi6/calculANTLR-python3/master/.github/demo.gif)



## 写在后面

这是一个非常简单的 demo，ANTLR  除了做编译器前端之外还有很多很多作用。之前在知乎看到一个律师答主用 ANTLR 做一些法律文件（半结构化文本）的提取，GitHub 也有很多有趣的项目，例如 [bramp/js-sequence-diagrams](https://github.com/bramp/js-sequence-diagrams) 做了一个时序图语言来方便画图，等等。
