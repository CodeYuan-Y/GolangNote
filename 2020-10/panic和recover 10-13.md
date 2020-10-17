#### 异常处理

`panic` 抛出异常 `recover`获取异常

Go中可以抛出一个`panic`的异常，然后在`defer`中通过`recover`捕获这个异常，然后正常处理。

**panic**
   1、假如函数F中书写了`panic`语句，会终止其后要执行的代码，在`panic`所在函数F内如果存在要执行的`defer`函数列表，按照defer的逆序执行
   2、返回函数F的调用者G，在G中，调用函数F语句之后的代码不会执行，假如函数G中存在要执行的`defer`函数列表，按照`defer`的逆序执行
   3、直到`goroutine`整个退出，并报告错误
**recover**
1、用来控制一个`goroutine`的`panicking`行为，捕获`panic`，从而影响应用的行为

2、一般的调用建议

​		a). 在`defer`函数中，通过`recover`来终止一个`goroutine`的`panicking`过程，从而恢复正常代码的执行

​		b). 可以获取通过`panic`传递的`error`

上面通俗的解释:

`panic`英语原意:恐慌 `recover`英语愿意:救

当你遇到`panic`(恐慌)你会崩溃掉 - 啥也干不了

当你被救治(`recover`) 以后 - 你又能工作了

就是程序只会执行`panic`之前和`recover`之后的程序,程序"恐慌中"是什么也干不了的

```go
func f(){
   fmt.Println("a")
   defer println("b")
   panic("这里抛出一个错误,程序在此终结")
   defer fmt.Println("c")
   fmt.Println("d")
}
```

```go
defer func(){ // 必须要先声明defer，否则不能捕获到panic异常
   fmt.Println("e")

   if err:=recover();err!=nil{
      fmt.Println(err) // 这里的err其实就是panic传入的内容，55
   }
   fmt.Println("f")
}()
f()
```

```
以上打印结果
a
e
这里抛出一个错误,程序在此终结
f
b
```

**几个注意点**

1.`recover`只有在`defer`调用的函数中才有效

2.`recover`所在的`defer`必须放在`panic`之前定义(放在`panic`后面，当`panic`中断程序那不就不会再执行下面的`defer`了吗)

3.`recover` 处理异常后，逻辑并不会恢复到 panic 那个点去，函数跑到 defer 之后的那个点

4.多个` defer` 会形成 `defer `栈，后定义的 `defer `语句会被最先调用

```go
func except(a int,b int)  {
	fmt.Println("hello")
	fmt.Println(a/b)
}

defer func() {
	fmt.Println("a")
    if err := recover();err != nil {
    	fmt.Println(err)
    }
    fmt.Println("b")
}()

except(2,0)
/*
打印结果
hello
a
runtime error: integer divide by zero
b
*/
```