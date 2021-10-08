# 抽象语法树go/ast库使用

## **推荐背景**
Go语言在编译过程中经过词法分析和语法分析之后，就到了抽象语法树的构建阶段，经历这一阶段之后，语句就真正组织成了程序代码。利用抽象语法树解析库，我们可以完成代码的自动化分析和自动化生成，因此通常用于做一些自动化的工具，例如wire。

## 使用案例
```
package main
 
import (
    "go/ast"
    "go/parser"
    "go/token"
)
var  src = `
	package main
	import "fmt"
	func main() {
    	fmt.Println("Hello, World!")
	}
`
 
func main() {
    fset := token.NewFileSet() // positions are relative to fset
    f, err := parser.ParseFile(fset, "", src, 0)
    if err != nil {
        panic(err)
    }
 
    // Print the AST.
    ast.Print(fset, f)
}
```
* fset是文件字符集用于定位ast.Node的文件位置
* parser.ParserFile的第二参数为文件名，第三参数为字符串，前者是待解析的文件路径，后者为待解析的字符串。前者优先级高于后者。第四参数为解析模式(可以使用"|"来结合多种解析模式)

**运行结果如下**
```text
     0  *ast.File {
     1  .  Package: 2:1
     2  .  Name: *ast.Ident {
     3  .  .  NamePos: 2:9
     4  .  .  Name: "main"
     5  .  }
     6  .  Decls: []ast.Decl (len = 2) {
     7  .  .  0: *ast.GenDecl {
     8  .  .  .  TokPos: 4:1
     9  .  .  .  Tok: import
    10  .  .  .  Lparen: -
    11  .  .  .  Specs: []ast.Spec (len = 1) {
    12  .  .  .  .  0: *ast.ImportSpec {
    13  .  .  .  .  .  Path: *ast.BasicLit {
    14  .  .  .  .  .  .  ValuePos: 4:8
    15  .  .  .  .  .  .  Kind: STRING
    16  .  .  .  .  .  .  Value: "\"fmt\""
    17  .  .  .  .  .  }
    18  .  .  .  .  .  EndPos: -
    19  .  .  .  .  }
    20  .  .  .  }
    21  .  .  .  Rparen: -
    22  .  .  }
    23  .  .  1: *ast.FuncDecl {
    24  .  .  .  Name: *ast.Ident {
    25  .  .  .  .  NamePos: 6:6
    26  .  .  .  .  Name: "main"
    27  .  .  .  .  Obj: *ast.Object {
    28  .  .  .  .  .  Kind: func
    29  .  .  .  .  .  Name: "main"
    30  .  .  .  .  .  Decl: *(obj @ 23)
    31  .  .  .  .  }
    32  .  .  .  }
    33  .  .  .  Type: *ast.FuncType {
    34  .  .  .  .  Func: 6:1
    35  .  .  .  .  Params: *ast.FieldList {
    36  .  .  .  .  .  Opening: 6:10
    37  .  .  .  .  .  Closing: 6:11
    38  .  .  .  .  }
    39  .  .  .  }
    40  .  .  .  Body: *ast.BlockStmt {
    41  .  .  .  .  Lbrace: 6:13
    42  .  .  .  .  List: []ast.Stmt (len = 1) {
    43  .  .  .  .  .  0: *ast.ExprStmt {
    44  .  .  .  .  .  .  X: *ast.CallExpr {
    45  .  .  .  .  .  .  .  Fun: *ast.SelectorExpr {
    46  .  .  .  .  .  .  .  .  X: *ast.Ident {
    47  .  .  .  .  .  .  .  .  .  NamePos: 7:5
    48  .  .  .  .  .  .  .  .  .  Name: "fmt"
    49  .  .  .  .  .  .  .  .  }
    50  .  .  .  .  .  .  .  .  Sel: *ast.Ident {
    51  .  .  .  .  .  .  .  .  .  NamePos: 7:9
    52  .  .  .  .  .  .  .  .  .  Name: "Println"
    53  .  .  .  .  .  .  .  .  }
    54  .  .  .  .  .  .  .  }
    55  .  .  .  .  .  .  .  Lparen: 7:16
    56  .  .  .  .  .  .  .  Args: []ast.Expr (len = 1) {
    57  .  .  .  .  .  .  .  .  0: *ast.BasicLit {
    58  .  .  .  .  .  .  .  .  .  ValuePos: 7:17
    59  .  .  .  .  .  .  .  .  .  Kind: STRING
    60  .  .  .  .  .  .  .  .  .  Value: "\"Hello World!\""
    61  .  .  .  .  .  .  .  .  }
    62  .  .  .  .  .  .  .  }
    63  .  .  .  .  .  .  .  Ellipsis: -
    64  .  .  .  .  .  .  .  Rparen: 7:31
    65  .  .  .  .  .  .  }
    66  .  .  .  .  .  }
    67  .  .  .  .  }
    68  .  .  .  .  Rbrace: 8:1
    69  .  .  .  }
    70  .  .  }
    71  .  }
    72  .  Scope: *ast.Scope {
    74  .  .  Objects: map[string]*ast.Object (len = 1) {
    74  .  .  .  "main": *(obj @ 27)
    75  .  .  }
    76  .  }
    77  .  Imports: []*ast.ImportSpec (len = 1) {
    78  .  .  0: *(obj @ 12)
    79  .  }
    80  .  Unresolved: []*ast.Ident (len = 1) {
    81  .  .  0: *(obj @ 46)
    82  .  }
    83  }

```

## ast.File内容
```
type File struct {
	Doc        *CommentGroup   // associated documentation; or nil
	Package    token.Pos       // position of "package" keyword
	Name       *Ident          // package name
	Decls      []Decl          // top-level declarations; or nil
	Scope      *Scope          // package scope (this file only)
	Imports    []*ImportSpec   // imports in this file
	Unresolved []*Ident        // unresolved identifiers in this file
	Comments   []*CommentGroup // list of all comments in the source file
}
```
其中Decls成员表示的就是文件中的顶级声明。接下来我们主要是关注它的内容。
### 包声明
```
  Package: 2:1
  Name: *ast.Ident {
  .  NamePos: 2:9
  .  Name: "main"
  }
```
对应文件中的 "package main"语句，记录了语句的位置以及包名(main)字符串的位置信息。
### 引入声明
```
	0: *ast.GenDecl {
 .  TokPos: 4:1
 .  Tok: import
 .  Lparen: -
 .  Specs: []ast.Spec (len = 1) {
 .  .  0: *ast.ImportSpec {
 .  .  .  Path: *ast.BasicLit {
 .  .  .  .  ValuePos: 4:8
 .  .  .  .  Kind: STRING
 .  .  .  .  Value: "\"fmt\""
 .  .  .  }
 .  .  .  EndPos: -
 .  .  }
 .  }
 .  Rparen: -
 }
```
在ast.GenDecl中记录了import语句的位置信息。Specs为一个ast.Spec的数组，记录了每一个import的包名及位置信息。
### 函数声明
```
	*ast.FuncDecl{
		Name:*ast.Index{...} 
		Type:*ast.FuncType{
			Params:*ast.FieldList{...}
			Results: *ast.FieldList{...}
		},
		Body:*ast.BlockStmt{...}
	}
```
* ast.FuncDecl标明定义的是一个函数。
* Name记录函数名
* Type是函数类型,其中Params表示参数信息，这里为空；Results表示返回值信息，这里也为空。
* Body则为函数体信息。

### 表达式
```
List: []ast.Stmt (len = 1) {
.  0: *ast.ExprStmt {
.  .  X: *ast.CallExpr {
.  .  .  Fun: *ast.SelectorExpr {
.  .  .  .  X: *ast.Ident {
.  .  .  .  .  NamePos: 7:5
.  .  .  .  .  Name: "fmt"
.  .  .  .  }
.  .  .  .  Sel: *ast.Ident {
.  .  .  .  .  NamePos: 7:9
.  .  .  .  .  Name: "Println"
.  .  .  .  }
.  .  .  }
.  .  .  Lparen: 7:16
.  .  .  Args: []ast.Expr (len = 1
.  .  .  .  0: *ast.BasicLit {
.  .  .  .  .  ValuePos: 7:17
.  .  .  .  .  Kind: STRING
.  .  .  .  .  Value: "\"Hello Wor
.  .  .  .  }
.  .  .  }
.  .  .  Ellipsis: -
.  .  .  Rparen: 7:31
.  .  }
.  }
}
Rbrace: 8:1
```
函数体中的表达式描述了函数的内容，例子中的fmt.Println("hello world")。

ast.CallExpr表示函数调用，其中SelectorExpr描述了调用函数的包名及函数名，Args则描述了参数信息。

## 遍历AST树
ast库提供了可以深度优先遍历AST的方法：func Inspect(node Node, f func(Node) bool)。 其中node为根节点，f为处理节点的方法。
```
ast.Inspect(f, func(n ast.Node) bool {
		var s string
		switch x := n.(type) {
		case *ast.BasicLit:
			s = x.Value
		case *ast.Ident:
			s = x.Name
		}
		if s != "" {
			fmt.Printf("%s:\t%s\n", fset.Position(n.Pos()), s)
		}
		return true
	})
```
此函数遍历f(ast.File)节点打印所有的标识符和文字。

更多相关类型，可以通过命令 `go doc ast  |grep "^type .*  struct"`查看。

## 进阶使用
利用AST对go文件进行分析，我们可以实现代码的自动生成，其中包括以下几个常见使用领域：

1. 代码注入: wire使用AST实现构造函数代码生成。
2. DeepCopy: 结合AST生成结构体的深拷贝函数代码
3. 对象赋值: 在领域编程中，常常需要在不同的领域对象中进行数据转换，利用AST的解析结果，可以自动生成指定领域对象间的转换函数文件。
   
## 小结
抽象语法树的生成属于程序编译流程中的一员，利用AST及其相关库提供到方法，我们可以很方便的解析一个go文件，把文件内容结构化，以便做进一步的分析和使用。AST广泛应用于代码自动生成的功能中，例如`go generate`命令，wire工具等等。其中不少企业也会在开源库中，使用Comment的特殊格式，来自定义框架代码的自动生成命令。

## 参考连接
https://www.cnblogs.com/double12gzh/p/13632267.html
https://cloud.tencent.com/developer/section/1142075

## 实验工具
https://astexplorer.net

https://greyireland.gitee.io/goast-viewer
