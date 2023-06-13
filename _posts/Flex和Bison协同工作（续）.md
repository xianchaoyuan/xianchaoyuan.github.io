flex和bison协同工作（上）和（下）已经编写出一个简易的计算器。接下来我们继续感受一下flex和bison的魅力。

**通过flex和bison来处理程序输入的一件很美好的事情就是文法的少量调整总是很简单。**想象一下，如果我们表达式语言能够处理圆括号以及注释（//）的话，它是否会变得更加有用和美好呢？为了实现这样的改进，我们仅仅需要为语法分析器和词法分析器分别添加一条和三条规则**（下面的示例中添加的都是完整的代码，便于学习，但一定要自己动手添加删除试试）**。

在语法分析器中我们定义两个新的token，OP和CP，作为开始和结束的圆括号，并且添加一条规则使一个圆括号表达式成为一个term：

calc.y

```
%{
#include <stdio.h>
%}

%token NUMBER
%token ADD SUB MUL DIV ABS
%token OP CP
%token EOL

%%
calclist: /* 空规则 */
    | calclist exp EOL { printf("= %d\n", $2); }
    ;
    
exp: factor
    | exp ADD factor { $$ = $1 + $3; }
    | exp SUB factor { $$ = $1 - $3; }
    ;
    
factor: term
    | factor MUL term { $$ = $1 * $3; }
    | factor DIV term { $$ = $1 / $3; }
    ;
    
term: NUMBER { $$ = $1; }
    | ABS term { $$ = $2 >= 0 ? $2 : -$2; }
    | OP exp CP { $$ = $2; }    // 新规则
    ;
%%

main(int argc, char **argv)
{
    yyparse();
}

yyerror(char *s)
{
    fprintf(stderr, "error: %s\n", s);
}
```
**注意在新规则中动作代码把$2 （圆括号中表达式的值）赋给了$$。 词法分析器有两条新规则来识别两个新的记号，还有一条新规则忽略两个斜线之后的任意文本。由于点号匹配除了换行符之外的任意字符，.*可以匹配掉一行剩下的部分。**

在词法分析器中添加“(”、“)”和"//"模式与动作。代码如下所示：
calc.l

```
%{
#include "calc.tab.h"
%}

%%
"+"    { return ADD; }
"-"    { return SUB; }
"*"    { return MUL; }
"/"    { return DIV; }
"|"    { return ABS; }
"("    { return OP; }
")"    { return CP; }
"//".* { }
[0-9]+ { yylval = atoi(yytext); return NUMBER; }
\n     { return EOL; }
[ \t]  { }
.      { printf("Mystery character %c\n", *yytext); }
%%
```
现在这个计算器就可以处理圆括号和注释了，修改起来是不十分方便。其实这些例子，我们即使完全通过C语言来实现它们也并没有太大的问题。现在我们分别从词法分析器和语法分析器来进行对比。

> flex：
> * 所使用的模式匹配技术非常快，与手写的词法分析器速度一致。在拥有更多模式的更复杂的词法分析器中，flex的词法分析器可能更快，因为手写的代码通常需要对每个字符做很多比较，而flex只需要一次。
> * flex版本的词法分析器永远比相应的C代码简短，这使它容易调试。

> bison:
> * 语法分析器比对应的手写语法分析器更简短也更容易调试。
> * bison可以帮助验证分析的文法是否有二义性。