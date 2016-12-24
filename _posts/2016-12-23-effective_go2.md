---
layout: post
title: Effective Go笔记2(个人的一点理解和思考)
category: golang
---

1. [数据](#v1/data)  
2. [初始化](#v1/init)  
3. [方法](#v1/method)  
<!--description-->

<a name="v1/data"></a>  

### 1. 数据  

##### 1. new分配内存  
- new(T)语句会新分配一块保存类型T数据的内存，并对内存空间进行清零，然后返回内存空间的地址(即*T)  
- 因为对申请的内存会清零，所以很多数据结构的声明定义正好可以利用这点。比如`sync.Mutex`  
- 简单变量声明也会对内存空间进行清零操作。下面的变量定义不需要进一步的设置都可以使用。  

```golang
type SyncedBuffer struct {
    lock     sync.Mutex
    buffer  bytes.Buffer
}

p := new(SyncedBuffer)    // type *SyncedBuffer
var v SyncedBuffer           // type SyncedBuffer
```

##### 2. make分配内存
- make仅用于`slices`, `maps`, `channels`类型数据的创建。而new不会用于他们的创建。原因说明如下:  

```
用slice具体说明，slice的底层结构中包括3个元素(指向保存数据的底层数组的指针，切片的长度，切点的容量)，代码表示如下:
type slice struct {
	p *[SIZE]T
    len int
    cap int
}

var p *[]int = new([]int)          // new仅仅把slice的各个成员内存空间清零，并不会构建底层的数组。所以 *p = nil
var v []int = make([]int, 100)  // make将会构建底层用于保存数据的数组[100]int

正因为这3类数据都需要有底层数据结构需要创建，所以不能用new来创建，而必须采用make
```

- make直接返回这3类数据，而不是这3类数据的指针。因为这3类数据直接用变量操作就很清晰，没有必要再加上指针符`*`   

##### 3. 数组  
- 数组在go编码中很少使用，除非像矩阵相关代码中。  主要还是用切片，关键是太方便了。  
- go和c语言的数组区别:  
  - go的数组变量是值，不是指针。当数组变量之间赋值时会copy所有元素.  
  - 当用数组当函数参数时，参数表示的是数组的一个备份，而不是指针。这样函数内部对数组的修改外部都不可见  
  - 数组的大小也是数组类型的一部分。也就是说[10]int和[20]int的类型是不同的。之间不能直接赋值，数据拷贝时需要用copy函数  

##### 4. 切片  
- 切片作为函数参数时，虽然也是传值。但是拷贝的切片变量也会指向相同的底层数组，所以函数内部对切片的修改函数外部是可见的  
- 当看到切片变量时，其实他已经包括切片的长度信息。Read函数的定义中只传入切片(告知把切片填满为止)，而不需要另外传入长度信息.  

```
func (f *File) Read(buf []byte) (n int, err error)
```

- 当只需要对切边的部分元素进行操作时，可以直接指定范围。如下: 只读32Bytes  

```
n, err := f.Read(buf[0:32])
```

- 二维切片定义中，可以每个切片的长度都可以单独定义。比如说保存文件内容时，每行长度可能不一样。二维切片的定义和初始化如下:  

```
// Allocate the top-level slice.
picture := make([][]uint8, YSize) // One row per unit of y.
// Loop over the rows, allocating the slice for each row.
for i := range picture {
	picture[i] = make([]uint8, XSize)
}
```

##### 5. Maps  
- map[key]value的key可以任何具备相等比较的数据类型。比如intergers, floating point, structs和arrays。但是切片不行。因为其不能进行相等比较。  
- map的取值操作如下:  

```
func offset(tz string) int {
    if seconds, ok := timeZone[tz]; ok {
        return seconds
    }
    log.Println("unknown time zone:", tz)
    return 0
}
```

- 当需要删除map中的项时，采用内置函数delete.如果需要删除ma全体，需要结合for~range进行循环删除。  

##### 6. 打印  
- 格式化字符串(比如%d)不支持符号或者大小，而是由需要打印的变量来决定这些属性。  

```
var x uint64 = 1<<64 - 1
fmt.Printf("%d %x; %d %x\n", x, x, int64(x), int64(x))

output:

18446744073709551615 ffffffffffffffff; -1 -1
```

- 格式化字符串%T会输出一个变量的数据类型  
- 对于自定义数据类型，一般会定义String()函数来自定义他的打印内容格式。但是在定义String()函数时，要小心调入无限循环的陷阱。  

```
type MyString string

func (m MyString) String() string {
    return fmt.Sprintf("MyString=%s", m) // Error: will recur forever.因为Sprintf()中格式化字符串%s将需要调用变量m的String()函数，也就是继续调用自己。
}

简单解决办法如下:

type MyString string
func (m MyString) String() string {
    return fmt.Sprintf("MyString=%s", string(m)) // OK: note conversion.
}
```

##### 7. 切片Append  
- append内置函数定义如下，虽然函数参数是值传递，函数里面对变量slice的修改，如果不新创建切片(容量够大)的情况下，外部的原始切片变量可以感知到追加的数据。但是如果slice容量不够，append函数内部将创建新的切片，这样追加的数据外面的切片变量就感知不到了。所以一定要用append函数的返回值覆盖原始的切片变量。  

```
func append(slice []T, elements ...T) []T
```

- 也就是说append的用法一般是下面这样的，其他使用时除非使用者知道自己在干什么。  

```
s = append(s, elements...)   // use return value to set slice s
```

<a name="v1/init"></a>  

### 2. 初始化  

##### 1. 常量  
- 常量在编译阶段就创建完成，所以如果是表达式的话就必须是编译器可以识别的常量表达式。比如:`1<<3`就是常量表达式，而` math.Sin(math.Pi/4)`就不是，虽然结果也是常量，但是math.Sin()函数调用需要在运行时才会执行，编译阶段是不能计算到常量结果的。


##### 2. 变量  
- 变量可以像常量一样被初始化，但是也可以是普通的表达式(在运行时执行就可以)  

```
var (
    home   = os.Getenv("HOME")
    user   = os.Getenv("USER")
    gopath = os.Getenv("GOPATH")
)
```

##### 3. init函数  
- init函数是在包中所有变量初始化完成后执行的。同时变量初始化是在所有导入包完成初始化后执行的。  


<a name="v1/method"></a>  

### 3. 方法  
- 可以给所有自定义类型定义方法(但是指针和接口类型除外)  
- 方法的接受者是指针还是值: 值的方法可以被指针和值两方调用，而指针的方法只能被指针调用。原因如下:  
  - *指针方法在方法体内可以修改方法接受者，如果用值来调用的话，方法体内接收到的是值的一个拷贝，所以任何值拷贝的修改在方法外部都无法感知。*   
  - *用值调用指针的方法有一个例外: 如果值是可寻址的，go语言会在值调用指针的方法时，自动在值前加一个取地址操作符。`b.Write`值的指针方法调用时，编译器会为我们重写成`(&b).Write`*  

