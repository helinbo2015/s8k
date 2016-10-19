#### 错误处理代码怎么写  

刚开始写go语言代码的时候，经常有一半都是下面这样的错误处理重复代码。  
```golang
if err != nil {
	return err
}
```
后来接触到k8s时，发现它里面的错误处理代码并不多。它对重复的错误代码采用了比较优雅的处理方式。下面是k8s里面的一段代码(REST GET请求)  
```golang
func Get(req  *client.Request) (runtime.Object, error) {
	return request.Namespace(namespace).
			Resource(resource).
			VersionedParams(&options, api.ParameterCodec).
			FieldsSelectorParam(fieldSelector).
			Do().
            Get()
}
```
看完上面的代码，可能会有下面两个疑问。  
1. 首先`Get()`方法需要返回`error`，但是`return`语句中的一系列方法调用后，都没有使用错误处理代码。  
2. 当`return`语句中的某个方法(比如: `Resource()`)执行产生`error`时，是怎么继续执行后续的处理的。  

其实设计比较简单，首先`client.Request`结构中定义了一个error.  
```golang
type Request struct {
	...
	// output
	err  error
    ...
}
```

然后我们看各个方法中对`err`变量的处理(用`Namespace()`说明)  
```golang
func (r *Request) Namespace(namespace string) *Request {
	if r.err != nil {
		return r
	}
	if r.namespaceSet {
		r.err = fmt.Errorf("namespace already set to %q, cannot change to %q", r.namespace, namespace)
		return r
	}
	... 
	return r
}
```
1. 首先方法入口进行错误判定处理，当`error`已经产生的情况下直接退出。  
2. 然后当方法中有错误产生的时候，设定`r.err`然后退出。这样后续的方法可以检验到已经发生的`error`  

最后总结一下:  
- go语言中`error`是一个值，所以可以各种比较、判断、赋值等。而不同于其他语言中的`error`处理(发生错误时需要用try-catch来捕获)  
- 正因为`error`是一个值，就可以很自由的跟`struct`，`func`结合，发挥程序员的想象，写出更优雅的错误代码。  

参考链接:  
1. [k8s源码](https://github.com/kubernetes/kubernetes)
2. [Errors are values](https://blog.golang.org/errors-are-values)
3. [Golang Error Handling lesson by Rob Pike](http://jxck.hatenablog.com/entry/golang-error-handling-lesson-by-rob-pike)