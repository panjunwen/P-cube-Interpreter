# 基于解析器组合子的简单解释器

一个玩具级程序语言解释器，包含词法分析、语法分析、抽象语法树生成以及解释执行等模块；此外，基于解析器组合子，实现一个通用的不确定自顶向下语法分析器。使用Python语言编写。

### 通用语法分析器

为了简明的表达解析器组合子的设计思想，同时与编译原理课程理论结合，设计了一个较为通用的不确定的自顶向下语法分析器（general_parser.py，得益于解析器组合子和运算符重载，50行代码不到即可实现，有作弊之嫌）。
文法规则的书写与编译原理课程中类似，下面是对表达式文法的描述：

```S -> E;E -> LI(T, VT('+'));T -> LI(F, VT('*'));F -> VT('i') | VT('n')
```
其中非终结符直接用“变量名”表示，VT('x')表示终结符x，只能用单个字符表示终结符，规则LI(x, 终结符VT) 表示一个由终结符VT分割的任意多个x形成的列表，此外可用+，|表示“连接”和“或”，括号可改变结合性。e.g.上述第二条规则等价于 E -> E + T | T。特别地，首条规则的左部必须为S，规则满足"自顶向下"顺序。
对于含左递归的文法，不需要手工消除左递归，只需要按上述规则书写成“列表”形式；对于含左公因子的文法，由于解析器本身支持回溯，也不需要修改文法(实践中还是要避免)。

```
> python general_parser.py grammer 'i+i*n+i*n'
解析成功：('+', ('+', 'i', ('*', 'i', 'n')), ('*', 'i', 'n'))
```

含有左公因子的例子：

```
python general_parser.py grammer2 xay
语法规则如下: 
S -> VT('x') + A + VT('y');
A -> (VT('a') + VT('b')) | VT('a')
解析成功：(('x', 'a'), 'y')

```

此外，对于“相互递归调用”的文法（一些个人见解，可能不太准确），e.g. A -> B | a; B  -> bA，布尔表达式中的not表达式就是这样，表达式文法中“因子”中的“(expression)”也会造成这种现象。这个“通用语法分析器”要求文法描述有一定的顺序，反映到实际的代码中，变量无法在定义前被调用（undefined），而Python正好不支持“惰性求值”，所以在解释器的构造中要使用一个“Lazy”组合子，以达到延迟求值的目的，而这个Lazy组合子之所以能解决问题，依赖于Python的函数定义不存在“顺序性”，具体参见组合子库。

上面说了一大堆，要表达的只有一句：这个“通用语法分析器”不支持“相互递归调用”的文法，因为它写起来太不优雅。但还是要举个例子，来简单说明下如何用这个组合子库来表达这种文法。

```
S = A()
A = lambda: Lazy(B) | VT('a')
B = lambda: VT('b') + Lazy(A)
```

### 语言解释器

语法规则结合了Python和Pascal，语法分析生成语法树，执行时递归计算节点值，实现了函数、流程控制、列表、输出等简单功能。具体请看下面的代码示例。

完善方向：添加语法错误提示和错误恢复功能（参考“装配脑袋/施凡”老师的文章），对左公因子文法进行处理避免回溯，达到“工具级”水平；进一步的，构造LR项目集族的自动机，实现自底向上的LR分析。

测试:

```
> python imp.py factorial.imp 
120

> python imp.py fib.imp
13

> python imp.py hello.imp -m verbose 
15
Final global variable values:
list: [1, 2, 3, 4, 5]
sum_of_list: <function sum_of_list('list')>

```
代码示例：

```
# 求和
list = [1, 2, 3, 4, 5];
def sum_of_list(list):
    sum = 0;
    for i in list do
        sum = sum + i
    end;
    return sum
end;
print sum_of_list(list)

# 斐波那契
def fib(n):
    if n <= 2 then
        return 1
    else
        return fib(n-1) + fib(n-2)
    end
end;
print fib(7)

# 阶乘
foo = 5;
p = 1;
n = 1000;
def factorial(n):
    while n > 0 do
      p = p * n;
      n = n - 1
    end
end;
x = factorial(foo);
print p
```

文件说明：

```
├── README.md
├── combinators.py		组合子库
├── factorial.imp			测试用例，阶乘
├── fib.imp				测试用例，斐波那契数列
├── general_parser.py		通用语法分析器
├── grammer				通用语法分析器测试用文法规则
├── hello.imp				测试用例，列表（数组）值累加
├── imp.py				解释器入口
├── imp_ast.py			抽象语法树及解释执行
├── imp_lexer.py			词法分析
├── imp_parser.py			语法分析
├── imp_test.py			一些单元测试内容
```

感谢以下几位博主所做的工作：

https://jayconrod.com/tags/imp

http://www.cnblogs.com/Ninputer/archive/2011/06/26/2090645.html

http://www.cnblogs.com/huxi/archive/2011/06/18/2084316.html