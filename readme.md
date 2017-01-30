# learn lua

### Lua 的特性

1. 变量名没有类型，值才有类型，变量名在运行时可与任何类型的值绑定；
2. 语言只提供唯一一种数据结构，称为（table），它混合了数组、哈希，可以用任何类型的值作为key和value。提供了一致且富有表达力的表构造语法，使得Lua很适合描述复杂的数据。
3. 函数是一等类型，支持匿名函数和正则尾递归（proper tail recursion）；
4. 支持词法定界(lexical scoping)和闭包(closure);
5. 提供thread类型和结构化的协程(coroutine)机制，在此基础上可方便实现协作试多任务；
6. 运行期间能编译字符串形式的程序文本并载入虚拟机执行；
7. 通过元表(metatable)和元方法(metamethod)提供动态元机制(dynamic metamechanism)，从而允许程序运行时根据需要改变或扩充语法设施的内定语义；
8. 能方便地利用表和动态元机制实现基于原型(prototype-based)的面向对象模型；
9. 从5.1版开始提供了完善的模块机制，从而更好地支持开发大型的应用程序；

### Lua 基础数据类型

函数type能够返回一个值或一个变量所属的类型。

```
> print(type("hello world"))
string
> print(type(print))
function
> print(type(true))
boolean
> print(type(360.0))
number
> print(type(nil))
nil
```

nil (空)

nil 是一种类型，Lua将nil用于表示“无效值”。一个变量在第一次赋值前的默认值是nil，将nil 赋予给一个全局变量就等于删除它。

```
> local num
> print(num)
nil
> num=100
> print(num)
100
> num = nil
> print(num)
nil
```

OpenResty的Lua接口还提供了一种特殊的空值，即 `ngx.null` ，用来表示不同于nil的“空值”。

boolean（布尔）

布尔类型，可选值true／false；Lua中nil和false为“假”，其他所有值均为“真”；这一点和其他编程语言有些差异。

number (数字)

Number 类型用于表示实数，和C/C++里面的double类型很类似。可以使用数学函数math.floor（向下取整）和 math.ceil(向上取整)进行取整操作。

一般地，Lua的number类型就是用双精度浮点数来实现的。值得一提的是，LuaJit支持所谓"dual-number"(双数)模式，即Luajit会根据上下文用整型来存储整数，而用双精度浮点数来存放浮点数。

另外，Luajit还支持“长长整型”的大整数（在X86_64体系机构上则是64未整数）。

string (字符串)

Lua 中有三种方式表示字符串：

1. 使用一对匹配的单引号。例如：'hello';
2. 使用一对匹配的双引号。例如："holla";
3. 字符串还可以用一种长括号（即[[]]）括起来的方式定义。我们把两个正的方括号（即[[）间插入n个等号定义为第n级正长括号。就是说，0级正的长括号写作[[，一级正的长括号写作[=[，如此等等。反的长括号也作类似定义；举个例子，4级反的长括号写作]====]。一个长字符串可以由任何一级的正的长括号开始，而由第一个碰到的同级反的长括号结束。整个词法分析过程将不受分行限制，不处理任何转义符，并且忽略掉任何不同级别的长括号。这种方式描述的字符串可以包含任何东西，当然本级别的反长括号除外。例：[[abc\nabc]]，里面的“\n”不会被转义。

另外，Lua的字符串是不可改变的值，不能像在C语言中那样直接修改字符串的某个字符，而是根据修改要求来创建一个新的字符串。Lua也不能通过下标来访问字符串的某个字符。想了解更多关于字符串的操作，请看string库章节。

在Lua实现中，Lua字符串一般都会经历一个“内化”(intern)的过程，即两个完全一样的Lua字符串在Lua虚拟机中只会存储一份。每一个Lua字符串在创建时都会插入到Lua虚拟机内部的一个全局的哈希表中。这意味着

1. 创建相同的Lua字符串并不会引入新的动态内存分配操作，所以相对便宜（但仍有全局哈希表查询但开销）；
2. 内容相同的Lua字符串不会占用多份存储空间；
3. 已经创建好的Lua字符串直接进行想等性比较时是`0(1)`时间度的开销，而不是通常见到的`0(n)`.

talbe(表)

Table类型实现来一种抽象的“关联数组”。“关联数组”是一种具有特殊索引方式的数组，索引通常是字符串（string）或者number类型，但也可以是除`nil`以外的任意类型的值。

在内部实现上，table通常实现为一个哈希表、一个数组、或者两者的混合。具体的实现为何种形式，动态依赖于具体的table的键分布特点。

想了解更多关于table的操作，请查看Table库章节。

function (函数)

在Lua中，函数也是一种数据类型，函数可以存储在变量中，可以通过参数传递给其他函数，还可以作为其他函数的返回值。

有名函数的定义本质上是匿名函数对变量的赋值。为说明这一点，考虑

```
function foo()
end
```

等价于

```
foo = function ()
end
```

类似地，

```
local function foo()
end
```

等价于

```
local foo = function ()
end
```
