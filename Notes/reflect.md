#### reflect
Go也实现运行时反射，这为我们提供一种可以在运行时操作任意类型对象的能力。比如我们可以查看一个接口变量的具体类型，看看一个结构体有多少字段，如何修改某个字段的值等等。

##### `TypeOf()` and `ValueOf`
在Go的反射定义中，任何接口都会由两部分组成的，一个是接口的具体类型，一个是具体类型对应的值。
比如 `var i int = 3` ，因为`interface{}`可以表示任何类型，所以变量`i`可以转为`interface{}`，所以可以把变量`i`当成一个接口，那么这个变量在Go反射中的表示就是`<Value,Type>`，其中`Value`为变量的值`3`,`Type`变量的为类型`int`。

```
package main

import (
    "fmt"
    "reflect"
)

type User struct {
    Name string
    Age int
}

func main() {
    u := User{"张珊",18}
    t := reflect.TypeOf(u)
    // main.User
    fmt.Println(t)
    fmt.Printf("%T\n",u)

    v := reflect.ValueOf(u)
    // {张珊 18}
    fmt.Println(v)
    fmt.Printf("%v\n",u)

    // {张珊 18}
    u1 := v.Interface().(User)
    fmt.Println(u1)

    // main.User
    t1 := v.Type()
    fmt.Println(t1)
}
```

##### `reflect.Value`转原始类型
*	我们可以通过`reflect.ValueOf`函数把任意类型的对象转为一个`reflect.Value`，那我们如果我们想逆向转过回来呢，其实也是可以的，`reflect.Value`为我们提供了`Inteface`方法来帮我们做这个事情。
*	我们也从一个`reflect.Value`获取对应的`reflect.Type`

##### 获取底层类型
对象`u`的实际类型是`User`，但是对应的底层类型是`struct`这个结构体类型.

```
fmt.Println(v.Kind())
```

##### 遍历字段和方法
*	通过反射，我们可以获取一个结构体类型的字段,也可以获取一个类型的导出方法，这样我们就可以在运行时了解一个类型的结构.
*	`NumField`方法获取结构体有多少个字段，然后通过`Field`方法传递索引的方式，循环获取每一个字段
*	`NumMethod`方法获取结构体有多少个方法，然后通过`Method`方法传递索引的方式，循环获取每一个方法

```
    for i:=0; i<t.NumField(); i++ {
        fmt.Println(t.Field(i).Name)
    }

    for i:=0; i<t.NumMethod(); i++  {
        fmt.Println(t.Method(i).Name)
    }
```

##### 修改字段的值
假如我们想在运行中动态的修改某个字段的值有什么办法呢？一种就是我们常规的有提供的方法或者导出的字段可以供我们修改(`setter`,`getter`方法等)，还有一种是使用反射。

```
package main

import (
    "fmt"
    "reflect"
)

func main() {
    x := 2
    fmt.Println(x)
    v := reflect.ValueOf(&x)
    v.Elem().SetInt(100)
    fmt.Println(x)
}
```

*	`reflect.ValueOf`函数返回的是一份值的拷贝，所以这里我们是传入要修改变量的地址
*	调用`Elem`方法找到这个指针指向的值
*	调用`SetInt`方法修改值
*	`CanSet`方法可以帮助我们判断是否可以修改该对象

##### 动态调用方法
*	结构体的方法我们不光可以正常的调用，还可以使用反射进行调用。
*	要想反射调用，我们先要获取到需要调用的方法，然后进行传参调用

```
package main

import (
    "fmt"
    "reflect"
)

type User struct {
    Name string
    Age int
}

func (u User) Print(prefix string){
    fmt.Printf("%s:Name is %s, Age is %d", prefix, u.Name, u.Age)
}

func main() {
    u := User{"张珊",18}
    v := reflect.ValueOf(u)

    mPrint := v.MethodByName("Print")
    args := []reflect.Value{reflect.ValueOf("前缀")}
    fmt.Println(mPrint.Call(args))
}
```

*	`MethodByName`方法可以让我们根据一个方法名获取一个方法对象，然后我们构建好该方法需要的参数，最后调用`Call`就达到了动态调用方法的目的。
*	获取到的方法我们可以使用`IsValid`来判断是否可用（存在）。