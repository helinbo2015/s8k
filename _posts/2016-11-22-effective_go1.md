---
layout: post
title: Effective Go笔记(个人的一点理解和思考)
category: golang
---

1. [格式化](#v1/formatting)  
2. [包名](#v1/package)  
3. [Getters](#v1/getter)  
4. [接口名](#v1/interface)  
5. [多单词变量写法](#v1/multiWords)  
6. [控制结构](#v1/control)  
7. [函数](#v1/function)
<!--description-->

<a name="v1/formatting"></a>  

### 1. 格式化  
各种编程语言的代码格式一直都是程序员争网上争论的焦点，比如说代码缩进是用`tab`还是`space`，再比如`{`是换行写好还是跟`if`，`for`等语句后好一点。  
go语言中没有给大家留下争执的空间，而是通过工具`go fmt`来统一代码格式。所以不管你代码中怎么缩进，最后执行完`go fmt`之后就会统一成标准格式。主要有一下几点:  

- 采用`tab`来缩进，只在需要的地方使用`space`。  
- 代码行长度无限制。如果觉得代码行太长，请自行换行并用`tab`和上一行缩进。  
- `if`，`for`，`switch`等判定语句都不带括号`()`,当然你写了，`go fmt`会帮你去掉。  

<a name="v1/package"></a>  

### 2. 包名  
当`import`一个包后，将通过`包名`来访问包中的内容(比如函数，变量等)。所以包名取名建议: `短小(short)`，`精简(concise)`，`见词知义(evocative)`。  

- 惯例1: 包名小写(lower case)，单词汇(single-word)  
- 惯例2: 代码所在目录名和包名一致。比如用`encoding/base64`导入目录`src/encoding/base64`下的包，包名应该为`base64`，而不应该为`encoding_base64`或者`encodingBase64`  

<a name="v1/getter"></a>  

### 3. Getters  
go语言中没有对`getters`和`setters`提供自动支持，并且不建议在变量名前加`Get`的方式来定义`getters。`这一点主要还是为了写出更简洁优雅的代码，比如说有一个owner(小写，不可导出)的变量，`getter`函数建议取名为`Owner`，而非`GetOwner`。而`setter`函数，取名类似为`SetOwner`，在实际使用中如下:  

```Go
owner := obj.Owner()
if owner != user {
    obj.SetOwner(user)
}
```

<a name="v1/interface"></a>  

### 4. 接口名(`Interface Names`)  
- 惯例1: 只有一个方法的接口取名: 方法名+er，比如: `Reader`，`Writer`，`Formatter`等  

<a name="v1/multiWords"></a>  

### 5. 多单词变量写法  
- 惯例1: 多单词的变量名写法: `MixedCaps`或者`mixedCaps`，而不采用下划线(mixed_caps)  

<a name="v1/control"></a>  

### 6. 控制结构  
和C语言相比，有如下区别:  

* `go`语言中没有do~while循环，但是for, switch更灵活。(可以用for代替do~while)  
* `if`和`switch`判断语句中可以像`for`语句一样写初始化声明。  
* `break`和`continue`可以用加`label`的方式来指示出口。  
* 引入多个不同判定条件的控制结构`select`  

##### 1. If语句  
- 带初始化声明的`if`写法  

```Go
if err := file.Chmod(0664); err != nil {
    log.Print(err)
    return err
}
```

- 重复声明和重复赋值(go的实用主义例子),使用`:=`的变量简短声明例子:  

```Go
f, err := os.Open(name)
if err != nil {
    return err
}
d, err := f.Stat()
if err != nil {
    f.Close()
    return err
}
```

上面代码中的`err`出现在两次声明中，使用`:=`的重复声明要求满足一下条件(否则编译报错)  

  - 重复声明的变量应该和已声明变量在同一可用范围内。  
  - 重复声明的赋值应该保持数据类型不发生变化。  
  - 重复声明的语句中必须至少有一个新声明的变量。  

##### 2. For语句  
- 针对数组，切片，字符串，map，读取管道的循环，结合`for + range`语句来实现  

```Go
for key, value := range oldMap {
    newMap[key] = value
}
```

- 只需要取range结果中第一个(比如key或者index)时, 写法如下:  

```Go
for key := range m {
    if key.expired() {
        delete(m, key)
    }
}
```

- 如果需要取range结果中第二个数据时, 用下划线`_`丢弃第一个数据。写法如下:  

```Go
sum := 0
for _, value := range array {
    sum += value
}
```

##### 3. switch语句  
- switch判断语句可以不为常量或者整数，可以是各类普通代码片段  
- switch语句为空时，将直接进行各个case的判断。类似于if-else-if-else的处理了。  

```Go
func unhex(c byte) byte {
    switch {
    case '0' <= c && c <= '9':
        return c - '0'
    case 'a' <= c && c <= 'f':
        return c - 'a' + 10
    case 'A' <= c && c <= 'F':
        return c - 'A' + 10
    }
    return 0
}
```

<a name="v1/function"></a>  

### 7. 函数  
- 可以定义多个返回值  
  * 带来的方便是不言而喻的，在C语言中想返回多个值，只能返回结构体指针(返回数据全部塞到结构体中)或者通过传递指针型的参数。这样C代码的可读性和复杂性都比go要差不少。  

- 带名称的返回值可以和函数参数一样在函数体中直接使用。  
  * 给返回值命名，主要是代码更简洁，同时生成文档时可读性更好。不过也有缺点是函数内部变量定义时要防止重名。  

- 延迟执行(Defer)  
  * defer语句会预设一个延迟(deferred)函数调用，当函数执行return前，将以LIFO的顺序执行延迟(deferred)函数。  
  * 当函数从不同分支退出都需要执行的处理，比如说关闭文件，释放锁等，就可以利用Defer.  
  * 延迟(deferred)函数的参数是在defer语句执行的时候被评估，而不是在延迟函数执行时被评估。这样就避免了defer执行时传入的变量在延迟函数执行被改变  

```
func trace(s string) string {
    fmt.Println("entering:", s)
    return s
}

func un(s string) {
    fmt.Println("leaving:", s)
}

func a() {
    defer un(trace("a"))
    fmt.Println("in a")
}

func b() {
    defer un(trace("b"))
    fmt.Println("in b")
    a()
}

func main() {
    b()
}
```

输出:  

```
entering: b
in b
entering: a
in a
leaving: a
leaving: b
```

