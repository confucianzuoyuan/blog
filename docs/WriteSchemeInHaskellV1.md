# 使用$Haskell$编写$Scheme$解释器

## 概论

大部分网络上的$Haskell$教程会使用一种与语言参考手册类似的方式来进行教学。他们会告诉你语法和一些语言结构，然后让你在交互式命令行里写一些简单的函数。而如何用函数式的方式来写一个有用的程序的问题却被留在了最后面或是直接被忽略了。

而我们会用一种完全不同的方式。你会从使用和解析命令行参数开始，然后写一个能够实现$R5RS \; Scheme$标准的像模像样的子集的$Scheme$解释器。你将会学习到$Haskell$的$I/O$，可变状态，动态类型，错误处理以及其他相关的解析功能。而当你完成这个教程的时候，你就会对$Haskell$和$Scheme$相当熟悉了。

这个教程主要面向两大读者群：

- 已经掌握$Lisp$或$Scheme$并且想要学习$Haskell$的人
- 虽然不懂任何编程语言，但是有大量背景知识并且对计算机非常熟悉的人

第二种读者会发现这个任务有些困难，因为在这里省略了很多$Scheme$以及其他通用的编程概念从而把教程的重点放在$Haskell$上面。$SICP$或者$The \; Little \; Schemer$之类的书会对你很有帮助。

而那些使用像C，Java或者Python这样的基于过程或面向对象语言的用户需要注意了：你需要忘掉大部分你已经熟悉的编程知识。Haskell与上述的语言完全不同，并且要求你用一种完全不一样的方式来进行思考。最好以白板的状态来开始这个教程并且不要尝试将Haskell与命令式语言进行比较，因为很多你以为你熟悉的概念（classes，functions，return）在Haskell里有完全不同的含义。

由于每一课都建立在之前代码的基础上，所以你最好按顺序来学习课程。

这个教程假定你用ghc作为你的Haskell编译器。代码在Hugs或者其他编译器里或许也能运行但并没有被完全测试过，也许你还需要下载一些额外的库来支持他们。

## 第一步：编译然后运行程序

首先，你需要安装一个ghc。在Linux上，它常常被预先安装好了或者能够通过apt-get或者yum命令来轻松搞定。你也可以从官网直接下载它。不过除非你确信你想从源码去编译它，否则下载一个二进制包就可以了。像安装其他的软件包一样下载和安装它既可。这个教程是在Linux下完成的，但是只要你会使用相关的命令行操作，它在Windows或是Mac环境下也一样能工作。

对UNIX或者Windows Emacs用户来说，这里有一个很棒的Emacs mode，包括了语法高亮和自动缩进的功能。Windows用户则能够直接使用记事本或者其他文本编辑器：Haskell的语法对记事本相当友好，尽管你仍要小心处理缩进。Eclipse用户建议使用eclipsefp插件。

现在，是时候写你的第一个Haskell程序了。这个程序将通过命令行读入一个名字然后输出一个问候语句。建立一个以 ".hs” 结尾的文件并输入下列代码。当心缩进，否则你的程序可能没法通过编译。

```haskell
module Main where  
import System.Environment  

main :: IO ()  
main = do args <- getArgs  
          putStrLn ("Hello, " ++ args !! 0)
```

我们来看下这段代码。前两行表示我们讲创建一个名叫Main的模块，并且导入了System这个模块。所有的Haskell程序都会从Main模块里的一个叫做main函数的地方开始运行。你可以在这个模块中导入其他的模块，但是如果没有了它，编译器就无法生成可执行文件供用户运行。此外Haskell是大小写的敏感的：模块名称需要是大写开头的，而函数声明则必须是非大写开头的。

`main :: IO ()`这行是函数的类型声明：它表示`main`函数的类型是`IO ()`，是一个返回`Unit`类型`()`的IO操作。一个`Unit`类型仅会包含一个值，而`()`，它表示什么也没有。类型声明在$Haskell$里是可选的：编译器能够自动的识别它们，当你的声明和编译器自动识别发生冲突的时候它则会报错。在这个教程中，为了更清晰的说明，所有的类型都是显式声明的。而你在家里跟着做的时候，你可能更愿意选择忽略，因为在编写这个程序的时候其实并不太需要去在意它们。

$IO$类型是$Monad$类型类的一个实例，$Monad$是一种抽象的概念，如果满足以下这两个条件，那我们就会说这样的值是$Monad$：

1. 这个值包含了一些特定类型的附加信息；
2. 大多数函数不需要去关心这些附加的信息。

在这个例子里，

1. 这个附加的信息就是将被执行的$IO$操作；
2. 而这个附加信息的值是不存在的，表示成 `()` 。

`IO[String]` 和 `IO()` 都同样属于 `IO Monad` 类型，但它们有着不同的基本类型。它们作用于 `(` 或是传递 `)` 不同类型的值， `[String]` 和 `()` 。

那些包含了附加信息的值则被称作$Monadic$。

$Monadic$值常被称作“操作”，因为最容易的思考$IO \; Monad$用途的方法就是把它当做一系列可能会影响外界世界的动作。这一系列动作会传递一些基础的数值，然后在这个过程中每个动作都会对这些值进行影响。

$Haskell$是一个函数式的语言：与给出计算机一系列指令从而让它执行不同，你需要给$Haskell$一系列定义来告诉它每一个函数来如何处理。这些定义会将各种动作和函数组合在一起。而编译器会识别出将它们组织在一起的执行方式。

要写出这样一个定义，你首先需要建立一个等式。等式的左边是一个名称，可能还会带有若干个与变量绑定的模式(后面会解释)。右边的话，会给出一些由其他定义组合而成的式子，从而告诉计算机如何遇到该定义时如何进行计算。这些等式就和一般的代数表达式一样：你总是可以在程序中用等式右边的部分来替代左边的名字，并且得到与之前相同的结果。这种行为被称作 **引用透明** ，而这种性质使得$Haskell$代码比其它的语言更加易于理解。

那我们应该怎么定义我们的`main`函数呢？我们知道它必须是一个能够从命令行读入参数，然后从打印出一些输出，最终返回`()`（空值）的`IO ()`操作。

这里有两种方法创建一个$IO$操作：

1. 使用`return`函数提升一个普通值进入$IO \; Monad$。
2. 连接两个已经存在的$IO$操作。

因为我们接下来要做两件事情，所以我们选择第二种方法。我们通过内建函数`getArgs`读入命令行参数并把它们存入一个字符串列表。而内建函数`putStrLn`则能够读入一个字符串然后将它输出到终端。

我们使用一个`do`代码块来连接这两个操作。一个`do`代码块包括很多行，所有的行按照第一个非空白字符在`do`后面排列，并且每行都可能是如下两种形式之一：

1. $name \leftarrow action1$
2. $action2$

第一种形式将$action1$的结果和$name$绑定，从而你可以在下一个操作中使用它。例如，如果有$action1$的类型是`IO [String]`(一个会返回一个字符串列表的$IO$操作，就和$getArgs$一样)，那$name$就会在接下来的一系列操作里和这个返回的字符串列表通过绑定操作符`>>=`绑定在一起。第二种情况仅仅执行这个$action2$，并通过`>>`操作符同下一行连结在一起。绑定操作符在处理不同$Monad$的情况下有不同的语义：在$IO \; Monad$中，它会连续执行所有的操作，然后对外部世界产生这些操作带来的副作用。由于这个绑定符号的语义依赖你具体使用的$Monad$类型，所以你并不能在同一个`do`代码块里把不同类型的$Monad$类型的操作糅杂在一起—在这里只有$IO \; Monad$是可用的（在同一个管道中）。

当然，这些操作可能自己会调用其他函数或是复杂的表达式，然后继续传递它们的计算结果（通过调用`return`或是其他最终调用了`return`的函数）。

在这个例子里，我们首先取出参数列表中的第一个元素`(args !! 0)`，然后把它拼接到字符串"Hello,"的后面（"Hello," ++），最后把结果传给`putStrLn`。

就这样，一个包含了之前所说的读取和打印操作的新的操作就这样创建完毕并存到了`main`这个返回值为`IO ()`的标识符中。这样$Haskell$系统就能够识别并运行它了。

$Haskell$中，字符串即是字符的列表形式，所以你可以对它使用任何的列表函数或是操作符。以下是一个完整的标准操作符列表和它们对应的优先级：

![](haskell-cheatsheet.png)

接下来编译和运行这个程序：

```bash
$ ghc -o hello_you --make listing2.hs
$ ./hello_you Jonathan
Hello, Jonathan
```

### 习题

1. 修改程序，让它能够从命令行读取两个参数然后打印出一条包含它们的信息。
2. 修改程序，让它能够使用输入的参数进行简单的四则运算，建议使用`read`来将字符串转化成数字类型，并用`show`来进行相反的操作。对各种不同的动作都操练一番。
3. `getLine`是一个从命令行读取一行输入信息然后返回字符串的$IO$操作。修改程序，让它能够提示需要一个名字并读取这个名字而不是像之前那样直接从命令行传入参数，最后打印它。

## 解析

### 一个简单的解析器

现在，让我们试着写一个非常简单的解析器。我们会用到$Parsec$库。

添加一行到导入模块的部分：

```haskell
import Text.ParserCombinators.Parsec hiding (spaces)
import  System.Environment
```

这样我们就可以使用$Parsec$库中的函数了，除了一个等下会和我们自己定义的函数名冲突的`spaces`函数。

现在让我们定义一个能够识别出$Scheme$中允许的符号的解析器：

```haskell
symbol :: Parser Char
symbol = oneOf "!#$%&|*+-/:<=>?@^_~"
```

这又是一个$Monad$的例子：在这里，被隐藏的 **额外信息** 包括在输入流中的位置，回溯记录以及$First$和$Follow$集等。$Parsec$会替我们解决这个问题。而我们只需要去调用$Parsec$库中的函数`oneOf`，它就会替我们将传递给它的字符串中的任意一个识别出来。$Parsec$库提供了一些内置的解析器：例如`letter`和`digit`函数。正如你将看到的，你可以将基本的函数组合成更加复杂的解析器。

让我们定义一个调用解析器并且处理可能的错误的函数：

```haskell
readExpr :: String -> String
readExpr input = case parse symbol "lisp" input of 
    Left err -> "No match: " ++ show err 
    Right val -> "Found value"
```

正如你从类型签名看到的一样，`readExpr`是一个将`String`转化成`String`的函数`(->)`。我们把传入的参数命名为`input`，然后把它和我们之前定义的名叫`symbol`的解析器一起传递给`parse`函数。传递的第二个参数是我们给输入定义的名称，这会在显示错误信息的时候用到。`parse`会返回一个被解析的返回值或者一个错误，因此我们是需要处理错误情况的。根据标准的$Haskell$编程规约，$Parsec$返回一个`Either`类型，用它的`Left`构造器表示错误并且用`Right`构造器来表示普通的值。

我们使用一个`case...of`的语句来对`parse`的各种可能的返回值进行匹配。如果我们得到一个`Left`值（错误），那我们就把这个`error`绑定给变量`err`然后在开头加上字符串`"No match "`然后返回。如果我们得到一个`Right`值，我们把它绑定给`val`，然后无视它并返回一个`"Found value"`字符串。

我们可以看到使用`case...of`来进行模式匹配的例子，之后我们会继续看到很多类似的做法的。

最后，我们需要修改我们的`main`函数来调用`readExpr`并且打印出结果：

```haskell
main :: IO ()
main = do 
    (expr:_) <- getArgs 
    putStrLn (readExpr expr)
```

为了编译并运行程序，需要在命令行指定`--make`参数，否则就会爆出链接错误。举个例子：

```bash
$ ghc --make -o simple_parser listing3.1.hs
$ ./simple_parser $
Found value
$ ./simple_parser a
No match: "lisp" (line 1, column 1):
unexpected "a"
```

### 空格

接下来，我们会对我们的解析器添加一系列改动来使它能够渐渐识别出我们给出的更加复杂的表达式。现在的解析器在遇到空白的时候就会卡住了：

```bash
$ ./simple_parser "   %"
No match: "lisp" (line 1, column 1):
unexpected " "
```

让我们来修正这个问题，并且忽略掉输入中的空格符。

首先，我们定义一个能够辨认出任意数量空格的解析器。顺便，这也是我们之前在导入$Parsec$模块的时候添加了`hiding (spaces)`的原因：模块中已经有一个`spaces`
函数了，但却不大符合我们的要求。

```haskell
spaces :: Parser ()
spaces = skipMany1 space
```

就像函数一样，操作也能传递给其他操作。在这里我们把`Parser`操作`space`传递给`Parser`操作`skipMany1`，来获取到一个能够解析一个或者多个空格的解析器。

现在，我们来编辑一下我们的解析函数：

```haskell
readExpr input = case parse (spaces >> symbol) "lisp" input of
    Left err -> "No match: " ++ show err
    Right val -> "Found value"
```

我们在第二课里简单看过一点关于`>>`（"bind"）操作符的内容，并且提到我们是把它放在`do`代码块中的每行的行尾来起到连接作用的。这里，我们显式的使用它来将我们的空格解析器和之前的符合解析器组合起来。然而，相比$IO \; Monad$绑定在`Parser`中有着完全不同的语义。

对于$Parser \; Monad$来说，绑定意味着“尝试匹配第一个解析器，然后用剩下的输入尝试匹配第二个，如果任意一次匹配失败的话，就返回失败”。总的来说，绑定在具体的$Monad$中会起到不同的效果；它被用作一种通用的组织计算的方式，所以能够适应各种不同的情况。你可以阅读对应的文档来判断出它到底会干什么。

编译并且运行代码。请注意我们这里的`spaces`函数是基于`skipMany1`定义的，它不会再像之前那样能够识别出单个的字符。因此你必须放一些空格在输入字符的前面。看下现在代码是如何运作的：

```bash
$ ghc -package parsec -o simple_parser [../code/listing3.2.hs listing3.2.hs]
$ ./simple_parser "   %"
Found value
$ ./simple_parser %
No match: "lisp" (line 1, column 1):
unexpected "%"
expecting space
$ ./simple_parser "   abc"
No match: "lisp" (line 1, column 4):
unexpected "a"
expecting space 
```

### 返回值

现在，我们的解析器还并不能做些什么—它仅仅是告诉我们一个给定的字符串是否能够被识别。现在，我们想让它能够做更多的事情：我们希望它能够将输入的字符串转换成一个特定的数据结构并让我们可以容易的遍历它。在这一节，我们将学习如何定义一个数据类型，并且修改我们的解析器让它能够返回该数据类型。

首先，我们来定义一个包含所有各种$Lisp$值的数据类型：

```haskell
data LispVal = Atom String
             | List [LispVal]
             | DottedList [LispVal] LispVal
             | Number Integer
             | String String
             | Bool Bool
```

这是一个代数数据类型的例子：它定义了一组`LispVal`类型的变量可能存储的值。每一个可能性（通过“|”符号分割的构造器）包含了一个代表构造器的标识符和这个构造器能够接受的一系列数据类型。在这里，一个`LispVal`可能是：

1. `Atom`。存储了一个用来命名它的字符串
2. `List`。其中存储了一组其他`LispVal`（$Haskell$列表用方括号表示），也被称为$Proper \; List$。
3. `DottedList`。对应$Scheme$中的`(a b . c)`形式。也被称作$Improper List$。存储了除最后一个元素以外的所有元素，然后再把最后一个元素额外存储起来。
4. `Number`。包含一个$Haskell$数字。
5. `String`。包含一个$Haskell$字符串。
6. `Bool`。包含一个$Haskell$布尔值。

构造器和类型使用的是不同的命名空间，所以你同时将一个类型名和构造器都定义成`String`，并没有问题。唯一要注意的是，它们都需要以大写字母开头。

接下来，我们来添加一些解析函数来返回对应的不同类型。一个字符串总是一个以双引号开头，然后接着一串不包含双引号的字符，最终以另一个双引号结束：

```haskell
parseString :: Parser LispVal
parseString = do
                char '"'
                x <- many (noneOf "\\"")
                char '"'
                return $ String x
```

我们再次使用`do`表达式而不是`>>`操作符来组织代码。只是因为我们需要获取解析得到的值（`many (noneOf "\\"")`的返回值）并且同时使用一些其他的解析操作。总的来说，对于不返回值得操作，使用`>>`符号，对于你需要立刻将返回的值传递到下一个操作的情况，使用`>>=`，其余的情况则用`do`代码块比较好。

当我们完成解析并从`many`函数中获取$Haskell$字符串时，我们调用了`String`构造器（`LispVal`数据类型）来把它转化成一个`LispVal`类型的值。每一个在代数数据类型中的构造器都能够像函数一样将传递给它的参数转化成它对应的类型。构造器还能够在模式匹配中作为左手边的匹配表达式进行使用；我们会在第三课里尝试将解析器返回的结果分别用`Either`类型的两种构造器进行匹配。

接着我们使用内置的`return`函数将我们的`LispVal`值$lift$成一个$Parser \; Monad$。注意，`do`代码块中的每行都必须有同样的类型，然而由于我们的`String`构造器的返回结果是`LispVal`类型，因此我们要利用`return`帮助将它封装成一个`Parser`操作并且在不消费任何输入的情况下直接将内部的值进行返回。这样我们的整个`parserString`操作就能够得到`Parser LispVal`类型的返回值了。

`$`符号是一个中缀函数调用符：它和我们直接使用`return (String x)`的作用一样，但是`$`是右结合的，并且运行的优先级较低，这样让我们能够省略掉一些原来需要写的括号。由于`$`是一个操作符，你可以像使用函数那样使用它做任何事情：传递它，部分调用等。在这个方面，它和`Lisp`中的`apply`函数功能一致。

现在继续来看$Scheme$的变量。一个`atom`是一个字母或者符号，跟着若干个字母，数字或者符号：

```haskell
parseAtom :: Parser LispVal
parseAtom = do 
              first <- letter <|> symbol
              rest <- many (letter <|> digit <|> symbol)
              let atom = first:rest
              return $ case atom of 
                         "#t" -> Bool True
                         "#f" -> Bool False
                         _    -> Atom atom
```

这里我们来看下另一个$Parsec$的组合运算符：选择运算符`<|>`。它会让我们首先尝试第一个解析器，如果它失败了，然后尝试第二个。如果任意一个成功了，那就会返回成功解析出得值。第一个解析器必须在它消费掉任何输入前失败返回：我们待会儿来看看如何用它来实现回溯。

一旦我们读到第一个字符和并成功读完剩下的部分，我们需要把它们放在一起组成一个`atom`。`let`声明定义了一个新的变量`atom`。我们使用列表连接符`:`来连接它们。和`:`相对应的，我们使用连接符`++`像这样来连接列表`[first] ++ rest`；`first`只是一个字符，我们可以用方括号包围它来将它转换成一个单元素的列表。

然后我们使用一个`case`表达式来尝试将字符串匹配成`true`和`false`，从而判断到底是应该创建和返回哪种`LispVal`类型。下划线符号`_`是一个可读性的技巧：目标会不断尝试匹配`case`块中的值直到遇到`_`（或者在此之前就因为某些异常失败了从而导致整个匹配失败）并作为一个通配符返回。因此如果代码运行到`_`条件下，它总是会匹配并且返回一个`atom`值。

最后，我们再为数字创建一个解析器。这里会展示更多的方法来处理$monadic$值：

```haskell
parseNumber :: Parser LispVal
parseNumber = liftM (Number . read) $ many1 digit
```

从右往左看会让你很容易理解这个表达式，因为函数调用符（`$`）和函数组合符（`.`）都是右结合的。结合器`many1`会匹配目标的一个或者多个传递给它的参数，这里我们会匹配到一个或者多个数字。我们会用返回的字符串来构建出一个数字的`LispVal`类型，不过这里我们貌似有一些类型上的匹配问题。因此首先，我们用内建的`read`函数来将字符串转化为数字。然后我们再把数字传递给`Number`构造器得到一个`LispVal`类型的值。我们用函数组合符创建出一个将右边参数的调用结果传递给左边参数的函数，因此我们就这样将两个函数调用结合起来了。

不幸的是，`many1 digit`的返回值是一个`Parser String`，所以我们的经过结合的`Number . Read`函数仍然不能直接对它进行操作。我们需要一种告诉它只操作$Monad$里的值的方法，然后再把处理后的结果返回给`Parser LispVal`。而标准库中的`liftM`函数刚好能帮助我们，所以我们对我们的函数`Number . Read`使用`liftM`，然后使用结果对`Parser`进行调用。

我们需要在程序顶部导入$Monad$模块来使用`liftM`函数：

```haskell
import Control.Monad
```

这种不断进行函数组合，函数调用和函数传递的编程风格在$Haskell$代码中是非常常见的。这会让你能够在一行中表达出非常复杂的逻辑，并把中间的阶段分解成其它可以用各种方式结合起来的函数。不幸的是，这表明你需要常常从右向左阅读$Haskell$代码并且注意跟踪它们的类型。在后面的教程中我们会看到更多的例子，所以你应该会马上能适应这种方式。

创建一个能够接受字符串，数字或是`Atom`的解析器：

```haskell
parseExpr :: Parser LispVal
parseExpr = parseAtom
         <|> parseString
         <|> parseNumber
```

编辑`readExpr`函数让它调用我们的新解析器：

```haskell
readExpr :: String -> String
readExpr input = case parse parseExpr "lisp" input of
    Left err -> "No match: " ++ show err
    Right _ -> "Found value"
```

编译并运行代码，你就能发现它接受任意的数字，字符串或者符号并且能够拒绝其他的情况了：

```bash
$ ghc -package parsec -o simple_parser [.../code/listing3.3.hs listing3.3.hs]
$ ./simple_parser "\\"this is a string\\""
Found value
$ ./simple_parser 25
Found value
$ ./simple_parser symbol
Found value
$ ./simple_parser (symbol)
bash: syntax error near unexpected token `symbol'
$ ./simple_parser "(symbol)"
No match: "lisp" (line 1, column 1):
unexpected "("
expecting letter, "\\"" or digit
```

### 习题

1. 重写`parseNumber`函数，不允许使用`liftM`，尝试
   1. 使用`do`代码块
   2. 显式的运用`>>=`操作符来进行连接
2. 我们的字符串并不太符合$R5RS$规范，因为它们不支持在字符串里使用转义之后的引号。修改`parseString`函数让`\"`表示一个引号字符而不是整个字符串的结束。你可能需要用一个新的解析器操作来替换`noneOf "\""`从而让它能接受非引号字符或者一个转义符号之后的引号字符。
3. 修改程序，让它支持`\n`、`\r`、`\t`、`\\`以及其它你希望转义的字符。
4. 修改`parseNumber`让它提供$Scheme$标准中对不同进制的支持。`readOct`和`readHex`函数或许会对你很有用。
5. 给`LispVal`增加一个字符构造器，然后为$R5RS$标准中定义的字符创造一个解析器。
6. 给`LispVal`增加一个浮点数构造器来支持$R5RS$中的小数相关的语法。参考$Haskell$中的`readFloat`函数。
7. 增加数据类型和解析器从而支持$Scheme$中的$full \; numeric \; tower$。$Haskell$已经有内建类型来表示其中的部分内容，你可以通过`Prelude`模块确认。至于其它，你可以通过定义复合类型的方法来表示它们。例如，一个分数可以用分子和分母表示而一个复数可以用实部和虚部来表示（每一部分都是一个实数）。

### 递归解析：列表和引号

接下来，给我们的解释器添加更多的解析器。从$Lisp$的知名括号列表开始：

```haskell
parseList :: Parser LispVal
parseList = liftM List $ sepBy parseExpr spaces
```

和`parserNumber`类似的，首先解析一系列由空格分隔开的表达式（`sepBy parseExpr spaces`），然后在$Parser \; Monad$内部调用构造符将它们组成一个`List`。注意我们能够把`parseExpr`直接传递给`sepBy`，尽管它是一个我们自己写的操作。

`dotted-list`的解析器稍微会复杂一点，不过仍然只是需要使用我们已经熟悉的概念：

```haskell
parseDottedList :: Parser LispVal
parseDottedList = do
    head <- endBy parseExpr spaces
    tail <- char '.' >> spaces >> parseExpr
    return $ DottedList head tail
```

注意我们是怎么使用`>>`把一系列的`Parser`操作连接起来并且`do`代码块中运用它的。表达式`char '.' >> spaces`返回一个`Parser ()`，然后通过与`parseExpr`结合产生一个`Parser LispVal`类型，完全和我们在`do`代码块中需要的类型一致。

```haskell
parseQuoted :: Parser LispVal
parseQuoted = do
    char '\\''
    x <- parseExpr
    return $ List [Atom "quote", x]
```

大部分都是我们已经熟悉了的内容了：这段程序读取一个单个的引号字符，读取一个表达式然后把它绑定给`x`，然后返回`(quote x)`，来表达一个$Scheme$符号。`Atom`构造器就像一个普通函数一样：你传递一个需要封装的字符串给它，然后它返回给你一个`LispVal`类型的值。你可以对这个`LispVal`做任何你一般情况下能做的事情，比如把它放入一个列表里。

最后，编辑`parseExpr`函数来把我们的新解析器添加进去：

```haskell
parseExpr :: Parser LispVal
parseExpr = parseAtom
         <|> parseString
         <|> parseNumber
         <|> parseQuoted
         <|> do char '('
                x <- try parseList <|> parseDottedList
                char ')'
                return x
```

这里演示了最后一个$Parsec$的功能：回溯。`parseList`和`parseDottedList`直到某个特定的位置都能够识别出相同的字符串；这打破了一个选择不能在出错前消费任何输入的前提。而`try`连接器试图运行某个的解析器，但是如果解析失败了，它会回滚到上一个状态。这让你在不影响其它分支的前提下对目标进行各种操作。

编译然后运行：

```bash
$ ghc -package parsec -o simple_parser [../code/listing3.4.hs listing3.4.hs]
$ ./simple_parser "(a test)"
Found value
$ ./simple_parser "(a (nested) test)"
Found value
$ ./simple_parser "(a (dotted . list) test)"
Found value
$ ./simple_parser "(a '(quoted (dotted . list)) test)"
Found value
$ ./simple_parser "(a '(imbalanced parens)"
No match: "lisp" (line 1, column 24):
unexpected end of input
expecting space or ")"
```

注意我们可以在`parseExpr`里任意深入的嵌套我们的解析器。这样，我们通过一些简单的定义就能够完全的让程序阅读$Lisp$代码了。这就是递归的威力。

### 习题

1. 添加`backquote`语法糖的支持：$Scheme$标准详述了它应该怎样展开成（`quasiquote/unquote`）。
2. 添加`vectors`的支持。你可以使用$Haskell$的内置实现`Array`，但是它使用起来可能会有些问题。严格说，一个`vector`应该有常数时间的索引和更新操作，但是事实上直接的更新操作在一个纯函数式语言里是很难实现的。你可能在看过该系列教程的后面的章节后会对如何实现它有更好的想法。
3. 如果不用`try`组合符的话，你需要将目标从左边开始分解并在接下来调用`parseExpr`解析器自身。最后需要用一个解析器对字符串进行匹配，它要么是空要么是一个点符号加上一个单元素的表达式。这里把这个有趣的练习留给你：把它们的返回值组合成一个要么是`List`要么是`DottedList`的`Either`类型：你也许需要把判断逻辑分解到另外一个辅助函数里。

## 求值：第一部分

### 开始求值

现在，我们仅仅能打印出来我们是否能够将给定的代码片段分辨出来而已。我们现在将向一个能够正常工作的$Scheme$解释器迈出第一步：计算代码片段的值。我们会先从一些简单的例子开始，但是很快你就能够开始进行各种计算了。

让我们从告诉$Haskell$如何将表示各种`LispVal`值的字符串打印出来开始：

```haskell
showVal :: LispVal -> String
showVal (String contents) = "\"" ++ contents ++ "\""
showVal (Atom name) = name
showVal (Number contents) = show contents
showVal (Bool True) = "#t"
showVal (Bool False) = "#f"
```

这是我们第一次真正对模式匹配进行介绍。模式匹配是一种能将代数类型进行解构的方法，依次和基于构造器的子句进行匹配并且把解构得到的部分和变量绑定起来以供之后使用。任何构造器都可以出现在模式中；如果标签和值的标签一致而且所有的子模式都和相应的组件匹配，那么这个模式就匹配了一个值。模式可以任意深的嵌套，而它用一种从里到外、从左到右的顺序匹配。一个函数定义的所有子句按照文本顺序依次尝试，直到一个模式匹配。如果这让你糊涂，你可以参考在我们深入求值器时的一些深嵌套的例子。

目前，你只需要知道每一个上面定义的子句都与一个`LispVal`构造器匹配，而右手边部分会告诉程序对那个构造器中包含的值做什么。

`List`和`DottedList`类似，但是我们需要定义一个辅助函数`unwordsList`来将列表转换成一个字符串：

```haskell
showVal (List contents) = "(" ++ unwordsList contents ++ ")"
showVal (DottedList head tail) = "(" ++ unwordsList head ++ " . " ++ showVal tail ++ ")"
```

`unwordsList`函数与`Prelude`库中的`unwords`函数类似，它把列表中的的单词用空格粘在一起。因为我们要处理的是`LispVal`而不是单词组成的列表，我们需要定义一个函数将`LispVal`转换成为对应的字符串形式然后再对它们使用`unwords`函数：

```haskell
unwordsList :: [LispVal] -> String
unwordsList = unwords . map showVal
```

我们的`unwordsList`定义并没有包含任何的参数。这就是一个$point\text{-}free$编程的例子：完全通过函数组合和局部调用的方式来进行定义，而单独的看待值或者说参数。相反的，这里我们使用了一组内建函数的组合来定义这个函数。首先，我们将`showVal`函数传递给`map`从而通过局部调用的方式创建了一个接受`LispVal`列表然后返回他们的字符串形式的列表的函数。$Haskell$函数是柯里化的：这意味着某个有两个参数的函数，例如`map`，实际上是一个会返回一个只一个参数的函数的函数。因此，如果你只使用一个参数去调用它，你就会得到一个可以传递，结合或是之后在进行调用的单参数函数。在这个例子里，我们将它和`unwords`函数结合：`map showVal`转换一个`LispVal`列表成为它们的字符串形式的列表，然后`unwords`将结果用空白字符结合在一起。

我们在上面使用了`show`函数。这个标准$Haskell$函数让你能够将任意是`Show`实例的类型转换成为一个字符串。我们希望对`LispVal`也能够做同样的事情，因此我们将它定义成`class Show`的一个成员，并将它的`show`方法直接定义成`showVal`：

```haskell
instance Show LispVal where show = showVal
```

完整的类型类的介绍不在这次教程的范围之内；你可以在其他教程或是$Haskell \; 98 \; report$里找到更多的相关信息。

让我们再试试看改变`readExpr`函数让它返回值实际解析值对应的字符串表示形式，而不仅仅是告诉我们解析成功：

```haskell
readExpr input = case parse parseExpr "lisp" input of
    Left err -> "No match: " ++ show err
    Right val -> "Found " ++ show val
```

编译然后运行程序：

```bash
$ ghc -package parsec -o parser listing4.1.hs
$ ./parser "(1 2 2)"
Found (1 2 2)
$ ./parser "'(1 3 (\"this\" \"one\"))"
Found (quote (1 3 ("this" "one")))
```

### 开始求值：初版

现在，让我们开始来编写一个求值器。这个求值器的目的是在于将作为代码的数据类型计算获得对应的表示数据的数据类型，即求出对应代码的结果。而对于$Lisp$来说，代码和数据的数据类型是相同的，因此我们的求值器会返回一个`LispVal`值。而其他有些语言会有更加复杂的代码结构，以及大量的语法形式。

对数字，字符串，布尔值和引用列表则相当简单：只需要返回数据本身就可以了。

```haskell
eval :: LispVal -> LispVal
eval val@(String _) = val
eval val@(Number _) = val
eval val@(Bool _) = val
eval (List [Atom "quote", val]) = val
```

这里我们看到了一种新的模式。`val@(String _)`能够匹配任意的字符串的后将整个`LispVal`值绑定给了`val`变量，而不仅仅是`String`构造器中的值。它是`LispVal`类型而不是字符串类型的。下划线是一个任意变量，它会匹配一个任意的没有与变量绑定的值。它能出现在任何的模式中，但是在和 **@-模式** 一起（你将变量与整个模式绑定）或是当你只对构造器的类型感兴趣的时候它会特别的有用。

在最后一个分支里我们会第一次看到一个嵌套的模式。List构造器中的数据类型是`[LispVal]`，一个`LispVal`的列表。我们会用一个特殊的二元列表`[Atom "quote", val]`去尝试匹配它，这是一个第一个元素是`quote`字符串而第二个元素可以是任意值的列表。匹配之后我们返回列表中的第二个元素。

让我们把`eval`函数集成到我们目前的代码中去。从`readExpr`函数开始，我们将它改回能够返回表达式而不是表达式的字符串表示形式的样子：

```haskell
readExpr :: String -> LispVal
readExpr input = case parse parseExpr "lisp" input of
    Left err -> String $ "No match: " ++ show err
    Right val -> val
```

然后修改我们的主函数，读取一个表达式，计算它，将结果转换成字符串，然后打印出来。既然我们现在知道了`>>=`和函数组合操作符的用法，让我们把整个过程更加简洁的拼接起来：

```haskell
main :: IO ()
main = getArgs >>= print . eval . readExpr . head
```

这里，我们获取`getArgs`操作的结果（一个字符串组成的列表）然后将它传入下面的函数组合中：

1. 取出第一个元素（`head`）
2. 进行解析（`readExpr`）
3. 求值（`eval`）
4. 转换结果成字符串并打印出来。

像之前那样编译并运行程序：

```bash
$ ghc -package parsec -o eval listing4.2.hs
$ ./eval "'atom" 
atom
$ ./eval 2
2
$ ./eval "\"a string\""
"a string"
$ ./eval "(+ 2 2)"
Fail: listing6.hs:83: Non-exhaustive patterns in function eval
```

我们仍然不能够用我们的程序做一些很有用的事情（注意到我们连`(+ 2 2)`都计算不了），但是一个基本的框架已经有了。接下来，让我们通过扩展基本函数的方式来让我们的解释器变得有用一些。

### 添加基本操作

接下来，我们来对我们的解释器进行一些改进从而可以支持基本的计算。虽然它还不是完整的“编程语言”，但也不远了。

我们首先给`eval`函数添加一个模式，从而让它可以处理函数调用。记住函数定义中的所有子句都必须放在一起，它们会依次进行匹配和求值，因此我们把这个表达式放在其他子句的后面：

```haskell
eval (List (Atom func : args)) = apply func $ map eval args
```

这里又是一个嵌套模式，但这次我们尝试使用构造操作符`:`进行匹配而不是像之前那样使用一个列表的形式。事实上在$Haskell$中，列表也仅仅是一个用来表示`cons`函数调用串的语法糖而已：`[1, 2, 3, 4] = 1:(2:(3:(4:[])))`。通过匹配`cons`本身而不是一个字符串列表，我们就像是在“获取列表的剩下的部分”而不是仅仅“获取列表的第二个元素”。例如，如果我们传递`(+ 2 2)`给`eval`函数，`func`变量会与`+`绑定而`args`变量会与`[Number 2, Number 2]`进行绑定。

剩下部分包括了一些我们之前已经熟悉的函数以及最后一个我们还没有定义的函数。由于我们必须递归的对每一个参数进行求值，因此我们对每一个参数调用`eval`函数。这允许我们能够进行`(+2 (- 3 1) (* 5 4))`这样的复合表达式。然后我们再将计算过后的参数传递给先前的函数再进一步进行求值：

```haskell
apply :: String -> [LispVal] -> LispVal
apply func args = maybe (Bool False) ($ args) $ lookup func primitives
```

内置函数`lookup`会在`Pair`列表搜索关键字（`Pair`的第一个元素）。然而，如果列表里没有包含对应的关键字，查找就会出错。因此该函数会返回一个$Haskell$的内建类型`Maybe`的实例从来避免程序异常。我们使用`maybe`函数来分别指定当成功或失败的情况下分别进行什么样的处理。当函数没有找到的情况，我们返回一个`False`值，即是`#f`（之后会添加更健壮的错误检查机制）。而如果找到了，我们就通过函数调用符这样`($ args)`来将它应用到函数的参数。

接下来，我们来定义一些需要支持的基础操作：

```haskell
primitives :: [(String, [LispVal] -> LispVal)]
primitives = [("+", numericBinop (+)),
              ("-", numericBinop (-)),
              ("*", numericBinop (*)),
              ("/", numericBinop div),
              ("mod", numericBinop mod),
              ("quotient", numericBinop quot),
              ("remainder", numericBinop rem)]
```

看下`primitives`函数的类型。事实上它是一个`Pair`类型的列表，恰好是能和`lookup`匹配，但是返回的函数类型都是从`[LispVal]`到`LispVal`的。$Haskell$中，你可以将函数存储到其他的数据结构中，不过所有的函数必须具有同样的类型签名。

同样，我们存储的函数它们本身也只是一个函数的返回结果，例如我们还没有定义的`numericBinop`函数。它读取一个原生$Haskell$函数（大部分情况下应该是操作符）再将它用分解参数列表，应用函数的代码封装起来，最后再将计算的结果通过`Number`构造器进行封装并返回。

```haskell
numericBinop :: (Integer -> Integer -> Integer) -> [LispVal] -> LispVal
numericBinop op params = Number $ foldl1 op $ map unpackNum params

unpackNum :: LispVal -> Integer
unpackNum (Number n) = n
unpackNum (String n) = let parsed = reads n :: [(Integer, String)] in 
                           if null parsed 
                              then 0
                              else fst $ parsed !! 0
unpackNum (List [n]) = unpackNum n
unpackNum _ = 0
```

和$R5RS \; Scheme$中一样，我们不会限制函数的参数只能有两个。我们的数值操作符能在一个任意长度的列表上工作，例如`(+ 2 3 4) = 2+3+4`和`(- 15 5 4 3) = 15-5-3-2`。我们是使用内建函数`foldl1`
来实现这一点的。事实上它即是将列表中每一个连接操作符都替换成了我们提供的二元函数`op`。

与$R5RS \; Scheme$不同，我们的解释器使用了一种弱输入的方式。这意味着如果一个值能够被解释成一个数字（例如字符串`"2"`)，我们就会将它看做一个数字，尽管它也许被标记成一个字符串。我们给`unpackNum`函数添加了一系列子句从使它能够解析各式各样的字符串。如果我们希望分解一个字符串并尝试用$Haskell$的内建函数`reads`去解析它，该函数就会返回一个（分析值，剩余值）对的列表给我们。

而对于列表的情况，我们直接尝试将它和一个单元素列表进行匹配并分解。匹配失败的话则会直接掉入第二个情况。

如果由于某些原因我们无法对数字进行解析，那么我们就暂时直接返回`0`作为结果。我们之后会对它进行修复并让它提示一个错误信息。

像之前那样编译并运行程序。注意到我们并没有做什么特殊的处理就直接能够对嵌套的表达式进行求值了，这是拜我们之前对函数每个参数进行求值所赐：

```bash
$ ghc -package parsec -o eval listing7.hs
$ ./eval "(+ 2 2)"
4
$ ./eval "(+ 2 (-4 1))"
2
$ ./eval "(+ 2 (- 4 1))"
5
$ ./eval "(- (+ 4 6 3) 3 5 2)"
3
```

### 习题

1. 添加对$R5RS$中的类型测试数的原生支持：`symbol?`，`string?`和`number?`等。
2. 修改`unpackNum`函数让它当输入值不是一个数字的时候总是返回`0`，即使它是一个可以被解析成数字的字符串或者列表。
3. 添加对$R5RS$中的`symbol-handling functions`的支持。`symbol`是指我们在之前的`LispVal`类型中被称作`Atom`的东西。

## 错误检查和异常处理

现在我们程序里的很多地方，我们要么是忽略了错误，要么是让它默默返回一个像是`#f`或是`0`这样表示无意义的默认值。一些像$Perl$或者是$PHP$的语言就是用这种方式来处理异常的。然而，这也意味着错误会默默的在整个程序里传递直到最终变成很大的并且让程序员能难定位的问题。我们这里希望一旦有错误发生，它就能立刻被注意到并且让程序停止运行。

首先，我们需要导入`Control.Monad.Error`库来取得$Haskell$的内置错误处理函数：

```haskell
import Control.Monad.Error
```

然后，让我们为错误也定义一个数据类型：

```haskell
data LispError = NumArgs Integer [LispVal]
               | TypeMismatch String LispVal
               | Parser ParseError
               | BadSpecialForm String LispVal
               | NotFunction String String
               | UnboundVar String String
               | Default String
```

这里是到目前为止我们可能会需要的一些构造器，之后我们可能还会想到一些其他的东西然后再添加进去。接下来，我们来定义如何打印`LispError`并且让它成为`Show`的一个实例：

```haskell
showError :: LispError -> String
showError (UnboundVar message varname)  = message ++ ": " ++ varname
showError (BadSpecialForm message form) = message ++ ": " ++ show form
showError (NotFunction message func)    = message ++ ": " ++ show func
showError (NumArgs expected found)      = "Expected " ++ show expected 
                                       ++ " args; found values " ++ unwordsList found
showError (TypeMismatch expected found) = "Invalid type: expected " ++ expected
                                       ++ ", found " ++ show found
showError (Parser parseErr)             = "Parse error at " ++ show parseErr

instance Show LispError where show = showError
```

接下来是时候让我们自己定义的类型成为一个`Error`的实例了。这样子我们才能让它同$GHC$的内置错误处理函数相配合。成为`Error`的一个实例事实上只需要给它提供一个能通过一条的错误消息或者它自身来进行初始化的函数：

```haskell
instance Error LispError where
     noMsg = Default "An error has occurred"
     strMsg = Default
```

接下来我们来定义一个用来表示要么会抛出`LispError`要么会返回值的函数的类型。还记得我们之前是怎么用`Either`类型来表示`parse`中的异常情况的吗？这里也是一样：

```haskell
type ThrowsError = Either LispError
```

类型构造器和函数一样也能够柯里化并被部分的调用。一个完整的类型可能是`Either LispError Integer`或者`Either LispError LispVal`，但是这里我想写成`ThrowsError LispVal`这样子。我们仅仅将`Either`类型部分应用于`LispError`，于是得到了一个能够可以用在任意类型上的构造器`ThrowsError`。

这里`Either`又是一个$Monad$的实例。这个例子中，在`Either`操作中被传递的附加信息是是否在这之间有错误发生。如果`Either`操作中包含的是一个普通值，那绑定操作就会发生，否则就会跳过计算步骤直接传递一个错误。其它语言中的异常就是这样子的，但由于$Haskell$的惰性求值机制，这里不需要一个额外的控制结构。如果绑定时已经能够判断这个值是一个错误，那么这个函数就永远不会被调用。

除了标准的$Monad$函数，`Either`类型还额外提供了另外其他两个函数：

1. `throwError`，传入一个`Error`类型的值然后将它`lift`成`Either`类型的`Left`构造器。
2. `catchError`，同时传入一个`Either`操作和一个将错误转换成另一个`Either`操作的函数。如果传入的`Either`操作是一个错误，就会调用传入的函数，举例来讲就会将你的错误通过`return`转换成一个正常值或者重新抛出另一个错误。

在我们的程序中，我们会能够将所有类型的错误转换成它们对应的字符串表示，然后作为正常值进行返回。让我们来创建这样的一个辅助函数：

```haskell
trapError action = catchError action (return . show)
```

调用`trapError`函数的返回结果是另一个包含合法（`Right`）数据的`Either`操作。我们依然需要将数据从`Either`中抽取出来，这样我们就能讲它传递给其它函数了：

```haskell
extractValue :: ThrowsError a -> a
extractValue (Right val) = val
```

我们这里刻意没有定义`extractValue`函数中传入`Left`值对应的分支，因为这实际上代表一个程序错误。我们只希望在`catchError`之后使用`extractValue`，所以它最好在将不合适的数据注入到其他代码之前就提前挂掉。

现在既然所有的基础架构都齐全了，是时候开始尝试使用我们的处理错误机制了。还记得我们的解析器之前在出错时仅仅会返回一个`"No match"`提示字符串吗？现在我们来让它能够封装并抛出一个原始的`ParseError`：

```haskell
readExpr :: String -> ThrowsError LispVal
readExpr input = case parse parseExpr "lisp" input of
     Left err -> throwError $ Parser err
     Right val -> return val
```

这里我们通过`Parser`构造器将最初的`ParseError`封装成了一个`LispError`类型，然后使用内置的`throwError`函数让它能够作为一个`ThrowsError`类型的$Monad$返回。由于`readExpr`函数现在会返回一个$Monad$值了，我们需要将其他分支也用`return`封装起来。

接下来，我们修改`eval`函数的类型签名让它也根据情况能返回对应$Monad$值，并且添加一个专门用来在遇到识别不了的模式时抛出异常的分支：

```haskell
eval :: LispVal -> ThrowsError LispVal
eval val@(String _) = return val
eval val@(Number _) = return val
eval val@(Bool _) = return val
eval (List [Atom "quote", val]) = return val
eval (List (Atom func : args)) = mapM eval args >>= apply func
eval badForm = throwError $ BadSpecialForm "Unrecognized special form" badForm
```

由于在函数应用分支中我们会递归的调用`eval`函数（现在会返回一个$Monad$值），我们需要进行一点修改。首先我们要把`map`函数修改成`mapM`，后者将一个$Monad$中的函数映射向一个列表并将每个返回值继续作为操作并按顺序进行绑定，最后返回一系列计算结果的列表。而在`Error`这个$Monad$中，这一连串操作都会逐一进行计算，除非其中任意一个失败了，那就会抛出一个异常—成功时你会得到一个`Right [result]`，而失败则是一个`Left error`。接下来，我们用$Monad$的绑定操作符来将结果传入被部分应用的`apply func`，同样当任何操作失败时都返回一个错误。

接下来我们来修改`apply`函数让它也能够在遇到识别不了的模式时抛出错误：

```haskell
apply :: String -> [LispVal] -> ThrowsError LispVal
apply func args = maybe (throwError $ NotFunction "Unrecognized primitive function args" func)
                        ($ args)
                        (lookup func primitives)
```

我们没有给函数调用符`($ args)`添加一个`return`。这是因为我们接下来会改变`primitives`函数，使从`lookup`中返回的函数也会返回一个`ThrowsError`操作：

```haskell
primitives :: [(String, [LispVal] -> ThrowsError LispVal)]
```

同样，显然我们还需要修改`numericBinop`函数，让它在只接受到一个参数的时候抛出错误：

```haskell
numericBinop :: (Integer -> Integer -> Integer) -> [LispVal] -> ThrowsError LispVal
numericBinop op           []  = throwError $ NumArgs 2 []
numericBinop op singleVal@[_] = throwError $ NumArgs 2 singleVal
numericBinop op params        = mapM unpackNum params >>= return . Number . foldl1 op
```

由于需要获取实际传入函数的值用作错误报告，我们这里使用一个 **@-模式** 来捕捉单值传入的情况。我们对一个单元素列表进行匹配，而且我们实际上不关心它到底是什么。我们同样也需要使用`mapM`来按顺序连接`unpackNum`的结果，因为每一次`unpackNum`调用都可能会因`TypeMismatch`而出错：

```haskell
unpackNum :: LispVal -> ThrowsError Integer
unpackNum (Number n) = return n
unpackNum (String n) = let parsed = reads n in 
                           if null parsed 
                             then throwError $ TypeMismatch "number" $ String n
                             else return $ fst $ parsed !! 0
unpackNum (List [n]) = unpackNum n
unpackNum notNum     = throwError $ TypeMismatch "number" notNum
```

最后，我们需要改变主函数来最终使用这整套$Error \; Monad$体系。这貌似有一点复杂，因为现在我们需要同时处理两种$Monad$（`Error`和`IO`）了。事实上，我们需要重新用`do`代码块来组织逻辑，因为要通过$point\text{-}free$风格来处理这种一个$Monad$的结果嵌套在另一个$Monad$中的情况几乎是不可能的：

```haskell
main :: IO ()
main = do
     args <- getArgs
     evaled <- return $ liftM show $ readExpr (args !! 0) >>= eval
     putStrLn $ extractValue $ trapError evaled
```

现在我们的新函数是这样子的：

1. `args`是命令行参数的列表
2. `evaled`以下操作的结果
   1. 获取第一个参数（`args !! 0`）
   2. 解析（`readExpr`）
   3. 传递给`eval`函数（`>>= eval` 绑定符比`$`符号优先级高）
   4. 在Error Monad中调用`show`函数（注意我们整个操作的类型是`IO (Either LispError String)`，因此`evaled`的类型是`Either LispError String`。必须要这样子因为一方面我们的`trapError`函数需要将`Error`类型转化成字符串，而另一方面它也需要和正常情况下的类型匹配）
3. `Caught`则是以下操作的结果
      1. 对`evaled`调用`trapError`函数，将错误转化成对应的字符串形式
      2. 调用`extractValue`函数将`Either LispError String`操作中的值取出来
      3. 通过`putStrLn`函数打印结果。

编译并运行程序，并尝试抛出一系列异常：

```bash
$ ghc -package parsec -o errorcheck [../code/listing5.hs listing5.hs]
$ ./errorcheck "(+ 2 \"two\")"
Invalid type: expected number, found "two"
$ ./errorcheck "(+ 2)"
Expected 2 args; found values 2
$ ./errorcheck "(what? 2)"
Unrecognized primitive function args: "what?"
```

一些读者反应这里和之后的一些例子需要添加`--make`参数才能成功进行编译。实际上这个参数是让$GHC$编译出一个完整的可执行程序，并搜索出所有在导入声明中列出的依赖。上述的命令尽管在我的系统里工作正常，但是如果你失败的话，加上`--make`试试。

## 求值：第二部分

### 更多操作：部分应用

既然现在我们可以来处理类型和参数之类的错误了，我们来重新整理下`primitive`列表并让它能够处理一些计算以外的事情。我们会添加一些布尔操作符，条件语句和一些基本的字符串操作。

从给`primitives`列表添加以下内容开始：

```haskell
("=", numBoolBinop (==)),
("<", numBoolBinop (<)),
(">", numBoolBinop (>)),
("/=", numBoolBinop (/=)),
(">=", numBoolBinop (>=)),
("<=", numBoolBinop (<=)),
("&&", boolBoolBinop (&&)),
("||", boolBoolBinop (||)),
("string=?", strBoolBinop (==)),
("string<?", strBoolBinop (<)),
("string>?", strBoolBinop (>)),
("string<=?", strBoolBinop (<=)),
("string>=?", strBoolBinop (>=)),
```

这里会用到一些我们还没有开始写的辅助函数：`numBoolBinop`，`boolBoolBinop`和`strBoolBinop`。与之前那些读取一些数字参数并返回一个整型的函数不同，这些函数都会读取两个参数并且返回一个布尔值。并且事实上它们仅仅是期望的参数类型不同而已，因此这里我们将逻辑整理成一个通用的`boolBinop`函数并传入一个会对参数进行处理的解包函数：

```haskell
boolBinop :: (LispVal -> ThrowsError a) -> (a -> a -> Bool) -> [LispVal] -> ThrowsError LispVal
boolBinop unpacker op args = if length args /= 2 
                             then throwError $ NumArgs 2 args
                             else do left <- unpacker $ args !! 0
                                      right <- unpacker $ args !! 1
                                      return $ Bool $ left `op` right
```

由于每个参数都有可能会抛出一个类型不匹配的错误，因此我们必须为了`Error Monad`而在一个`do`代码块中将它们依次分解。然后再将操作符运用在两个参数上并且将结果用`Bool`构造器封装起来。任何一个函数都能够通过一对反引号将它变成一个中缀操作符。

同时我们也来看下类型签名。`boolBinop`函数读取两个函数作为它的前两个参数：第一个用来将参数从`LispVal`类型解包成原生的$Haskell$类型，而第二个则是实际进行的操作。通过将部分的行为参数化，代码的重用性变得更好了。

现在来根据不同情况下的解包函数来通过`boolBinop`定义三个函数：

```haskell
numBoolBinop  = boolBinop unpackNum
strBoolBinop  = boolBinop unpackStr
boolBoolBinop = boolBinop unpackBool
```

现在我们还没告诉$Haskell$如何从`LispVal`类型的值中解包出字符串。这其实和unpackNum函数类似，我们只需要对目标值进行模式匹配并且在失败时抛出错误就行了。同样，如果传入的是一个可以被解释成字符串的其他基本类型（数字或者布尔值）我们也会同样默默将它转换成对应的字符串表达形式。

```haskell
unpackStr :: LispVal -> ThrowsError String
unpackStr (String s) = return s
unpackStr (Number s) = return $ show s
unpackStr (Bool s)   = return $ show s
unpackStr notString  = throwError $ TypeMismatch "string" notString
```

使用类似的代码来对布尔值解包：

```haskell
unpackBool :: LispVal -> ThrowsError Bool
unpackBool (Bool b) = return b
unpackBool notBool  = throwError $ TypeMismatch "boolean" notBool
```

在进入下一步之前，先编译并运行几个例子来看看它是否正确：

```bash
$ ghc -package parsec -o simple_parser [../code/listing6.1.hs listing6.1.hs]
$ ./simple_parser "(< 2 3)"
#t
$ ./simple_parser "(> 2 3)"
#f
$ ./simple_parser "(>= 3 3)"
#t
$ ./simple_parser "(string=? \"test\"  \"test\")"
#t
$ ./simple_parser "(string<? \"abc\" \"bba\")"
#t
```

### 条件：模式匹配

现在，我们继续将if语句添加到我们的求值器中。根据Scheme标准，我们这里会认为除了#f以外的其他所有值都是True：

```haskell
eval (List [Atom "if", pred, conseq, alt]) = 
     do result <- eval pred
        case result of
             Bool False -> eval alt
             otherwise  -> eval conseq
```

由于函数定义是会被依次进行计算的，这部分记得需要放在 `eval (List (Atom func : args)) = mapM eval args >>= apply func` 前面不然它会抛出一个Unrecognized primitive function args: "if"错误。

这又是一个嵌套模式匹配的例子。这里，我们要匹配一个四元素的列表。其他第一元素元素必须是Atom类型的if，其他则可能是任意的Scheme类型。我们求出pred的值，如果它是False的，则函数返回alt的值，否则的话，我们计算并返回conseq的值。

编译并运行程序，你就能尝试使用条件分支了：

```bash
$ ghc -package parsec -o simple_parser [../code/listing6.2.hs listing6.2.hs]
$ ./simple_parser "(if (> 2 3) \"no\" \"yes\")"
"yes"
$ ./simple_parser "(if (= 3 3) (+ 2 3 (- 5 1)) \"unequal\")"
9
```

### 列表操作：car、cdr和cons

接下来我们将一些基本的列表操作添加到primitives中。由于我们已经选择了使用Haskell的代数类型而不是Pair类型来表达列表了，因此这里的定义就反而可能比在大部分Lisp里更加复杂一点。通过打印出来得S表达式也许你能够更加容易的理解它们的效果：

. (car '(a b c)) = a
. (car '(a)) = a
. (car '(a b . c)) = a
. (car 'a) = error – not a list
. (car 'a 'b) = error – car only takes one argument

我们可以直接将它们翻译成对应的模式匹配子句，记得(x:xs)会将一个列表分割成第一个元素以及接下来的其他部分：

```haskell
car :: [LispVal] -> ThrowsError LispVal
car [List (x : xs)]         = return x
car [DottedList (x : xs) _] = return x
car [badArg]                = throwError $ TypeMismatch "pair" badArg
car badArgList              = throwError $ NumArgs 1 badArgList
```

cdr函数也是同样：

. (cdr '(a b c)) = (b c)
. (cdr '(a b)) = (b)
. (cdr '(a)) = NIL
. (cdr '(a . b)) = b
. (cdr '(a b . c)) = (b . c)
. (cdr 'a) = error – not a list
. (cdr 'a 'b) = error – too many arguments

我们可以用一个子句来代表前三种情况。我们的解析器将 `'()` 认为是一个空列表 `[]` ，并且当你使用 `(x:xs)` 来对 `[x]` 进行匹配时， `xs` 会绑定到一个空列表 `[]` 。其他的情况我们都用单独的子句来表示：

```haskell
cdr :: [LispVal] -> ThrowsError LispVal
cdr [List (x : xs)]         = return $ List xs
cdr [DottedList [_] x]      = return x
cdr [DottedList (_ : xs) x] = return $ DottedList xs x
cdr [badArg]                = throwError $ TypeMismatch "pair" badArg
cdr badArgList              = throwError $ NumArgs 1 badArgList
```

cons函数会有一点棘手，所以我们还是来一个个看下各种可能发生的情况吧。如果你将任何一个值和空列表（Nil）通过cons结合，那么你就会得到一个单元素的列表，Nil会充当一个终止符：

```haskell
cons :: [LispVal] -> ThrowsError LispVal
cons [x1, List []] = return $ List [x1]
```

如果你将任意值和一个列表通过cons结合，这就像是就那个值插进列表的最前面：

```haskell
cons [x, List xs] = return $ List $ x : xs
```

然后，如果你处理的是一个DottedList，那你需要考虑不正确的尾元素的情况并让它保持还是一个合法的DottedList：

```haskell
cons [x, DottedList xs xlast] = return $ DottedList (x : xs) xlast
```

如果你把两个都不是列表的对象通过cons组合，或者把列表作为第一个参数，那就会得到一个DottedList。这是因为这样通过cons组合的部分不像其他普通列表那样由一个Nil来终结的缘故。

```haskell
cons [x1, x2] = return $ DottedList [x1] x2
```

最后，任意传入大于或小于两个参数的情况都会引起错误：

```haskell
cons badArgList = throwError $ NumArgs 2 badArgList
```

我们的最后一步是实现一个 `eqv?` 函数。Scheme提供了三种不同程度的相等断言： `eq?` ， `eqv?` 以及 `equal?` 。对我们来说， `eq?` 和 `eqv?` 基本上是一样的：如果两个值打印出来的结果是一样的，那它们就相等，虽然貌似这样运行起来也许会比较慢。所以我们这里就为它们两个提供一个实现并且将它注册成 `eq?` 和 `eqv?` 。

```haskell
eqv :: [LispVal] -> ThrowsError LispVal
eqv [(Bool arg1), (Bool arg2)]             = return $ Bool $ arg1 == arg2
eqv [(Number arg1), (Number arg2)]         = return $ Bool $ arg1 == arg2
eqv [(String arg1), (String arg2)]         = return $ Bool $ arg1 == arg2
eqv [(Atom arg1), (Atom arg2)]             = return $ Bool $ arg1 == arg2
eqv [(DottedList xs x), (DottedList ys y)] = eqv [List $ xs ++ [x], List $ ys ++ [y]]
eqv [(List arg1), (List arg2)]             = return $ Bool $ (length arg1 == length arg2) && 
                                                             (all eqvPair $ zip arg1 arg2)
     where eqvPair (x1, x2) = case eqv [x1, x2] of
                                Left err -> False
                                Right (Bool val) -> val
eqv [_, _]                                 = return $ Bool False
eqv badArgList                             = throwError $ NumArgs 2 badArgList
```

除了处理两个List值的部分，其他子句大多都是自解释的。这里，在检查确认了两个列表是相等的长度之后，使用zip函数将列表配对并一一进行对比。eqvPair函数式一个局部定义的例子：它用where关键词来定义，除了它的作用域仅仅是eqv函数的一个子句，其他都和普通的函数一样。这里由于我们已经知道eqv函数只会在传递给它的不是两个参数的时候才会抛出一个错误，因此Left err -> False这行其实是永远也不会被执行的。

### equal?和弱类型：异构列表

之前我们已经介绍过有关弱类型的概念了，因此这里我们尝试创建一个equal?函数，它会忽视类型并仅仅判断两个值是否能被解释成相同的结果。举个栗子，(eqv? 2 "2") = #f，但我们希望能够得到(equal? 2 "2") = #t。基本上，我们需要尝试所有的解包方法，如果它们中的任何一个会让对应的Haskell值相等，那就返回True。

一个显而易见的方法就是把所有解包的函数都放进一个列表里然后通过mapM函数让它们逐个执行。然而很不幸你没法这么干，因为Haskell不允许你将不同类型的值放进同一个列表中。各式各样的解包函数显然会返回不同的类型，因此你没法将它们存在一起。

我们这里需要使用一个GHC的扩展包--Existential Types，来使用异构列表，虽然它仍然需要受到类型类的约束。扩展在Haskell的使用当中是相当常见的：基本上你如果需要写一些靠谱的大型程序都会或多或少用刀，它们也往往能互相兼容（Existential Types在Hugs和GHC里都运行良好并且很有希望被纳入Haskell标准）。注意你需要使用一个特别的编译参数来开启这个功能：-fglasgow-exts。也可以添加-XExistentialQuantification或者是在程序的最开始加上这么一段注解{-# LANGUAGE ExistentialQuantification #-}。（总的来说，编译时的参数位-Xfoo都可以被在源代码中的{-# LANGUAGE foo #-}注解来替代。）

首先我们需要定义一个能够表示LispVal -> something的函数的类型，只要这个something能够支持判等：

```haskell
data Unpacker = forall a. Eq a => AnyUnpacker (LispVal -> ThrowsError a)
```

这里和其他普通的代数数据类型都是类似的，除了这里有一个类型限制。它表示“对于任意是Eq实例的类型，你可以定义一个读取一个将LispVal转换成那个类型并且有可能抛出错误的函数作为参数的Unpacker类型”。我们将这个函数通过AnyUnpacker构造器进行封装，然后我们就可以创建一个Unpacker列表来实现我们之前想要的效果。

在equal?函数的定义之前，我们来首先来看一个读取一个Unpacker类型然后判断两个LispVal值在解包后是否相等的的辅助函数：

```haskell
unpackEquals :: LispVal -> LispVal -> Unpacker -> ThrowsError Bool
unpackEquals arg1 arg2 (AnyUnpacker unpacker) = 
             do unpacked1 <- unpacker arg1
                unpacked2 <- unpacker arg2
                return $ unpacked1 == unpacked2
        `catchError` (const $ return False)
```

在通过模式匹配获取实际的解包函数之后，我们进入了一个ThrowsError Monad的do代码块。这里我们获取两个LispVal值在Haskell中对应的值然后对它们进行比较。如果在解包的过程中发生了任何错误，就也会返回一个False，这里由于catchError函数需要我们传递一个函数用来处理错误值，我们就直接使用const函数就可以了。

最后，我们给出equal?函数的定义。

```haskell
equal :: [LispVal] -> ThrowsError LispVal
equal [arg1, arg2] = do
      primitiveEquals <- liftM or $ mapM (unpackEquals arg1 arg2) 
                         [AnyUnpacker unpackNum, AnyUnpacker unpackStr, AnyUnpacker unpackBool]
      eqvEquals <- eqv [arg1, arg2]
      return $ Bool $ (primitiveEquals || let (Bool x) = eqvEquals in x)
equal badArgList = throwError $ NumArgs 2 badArgList
```

这里第一步操作创建了一个异构列表[unpackNum, unpackStr, unpackBool]，然后将一个被部分应用的(unpackEquals arg1 arg2)映射到它上面。得到一个布尔值列表后，我们使用Prelude中的函数or，如果其中任意一个结果是True则为True。

第二部操作使用eqv?函数对两个参数进行测试。因为我们希望equal?会比eqv?更加宽松的缘故。因此如果eqv?返回True的话，这里也应该直接返回True。这就让我们能够避免处理一些类似于列表或者DottedList的情况了。（事实上这里引入了一个bug；练习2会提到）

最后，将上面的值用or连接起来并且将结果封装在一个Bool构造器里，从而返回一个LispVal。let (Bool x) = eqvEquals in x是一个便捷的从代数类型中分解值得方式：通过模式匹配将eqvEquals中包含的值取出然后返回。这个let表达式的结果即是关键词in之后的部分。

将函数插入到primitives列表中好让它们能够被使用：

```haskell
("car", car),
("cdr", cdr),
("cons", cons),
("eq?", eqv),
("eqv?", eqv),
("equal?", equal)]
```

你需要通过-fglasgow-exts参数来开启GHC扩展功能来进行编译这段代码：

```bash
$ ghc -package parsec -fglasgow-exts -o parser [../code/listing6.4.hs listing6.4.hs]
$ ./parser "(cdr '(a simple test))"
(simple test)
$ ./parser "(car (cdr '(a simple test)))"
simple
$ ./parser "(car '((this is) a test))"
(this is)
$ ./parser "(cons '(this is) 'test)"
((this is) . test)
$ ./parser "(cons '(this is) '())"
((this is))
$ ./parser "(eqv? 1 3)"
#f
$ ./parser "(eqv? 3 3)"
#t
$ ./parser "(eqv? 'atom 'atom)"
#t
```

### 习题

. 改变if函数的定义让它只接受Bool类型的值并在其他情况下抛出异常而不是把所有不是False的值都当做True。
. equal?函数有一个bug由于在列表中的值都是通过eqv?而不是equal?来比较的。例如，(equal? '(1 "2") '(1 2))会得到一个False，而你也许会希望获得True。修改equal?函数让它在对列表进行递归计算的时候也会忽略类型。你可以模仿eqv?函数来显示的定义它也可以将处理list的情况另外创建一个辅助函数来处理，并且将它判等时使用的函数进行参数化。
. 实现cond和case表达式
. 添加剩下的字符串函数。你现在可能还没法实现一个自己的string-set!，这在Haskell里有点难实现，不过在接下来的两章之后你可能就能够实现它了。