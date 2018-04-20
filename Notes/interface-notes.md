#### interface
接口是一组方法，但同时也是一种类型

##### 作为一组方法
```
type Animial interface {
	Speak() string
}

type Dog struct { // .. }
func (d Dog) Speak() string { // }

type Cat struct { // .. }
func (c Cat) Speak() string { // }

func main() {
	animals := []Animal{Cat{}, Dog{}}
	for _, animal := range animals {
		fmt.Println(animal.Speak())
	}
}
```

##### 作为一种类型
*	空的`interface{}`没有任何的函数，所有的类型都可以认为是实现了`interface{}`。
*	这意味着如果你定义了一个函数，函数参数是空的`Interface`的话，那么函数可以接受任意类型的参数。

```
// 类似于模板参数
// 需要显示转化
func Job(v []interface{}){ 
	for _, data := range v {
		fmt.Println(data)
	}
}

data := []int{1,2,3,4}
value := make([]interface{}, len(data))
for i :=0; i < len(data); i++ {
	value[i] = data[i]
}
Job(value)
```


