#### Interface
*	An interface is a set of abstract methods, and can be implemented by non-interface types.

```
package main

import "fmt"

type Human struct {
	name  string
	age   int
	phone string
}

type Student struct {
	Human
	school string
	loan   float32
}

type Employee struct {
	Human
	company string
	money   float32
}

func (h Human) SayHi() {
	fmt.Printf("Hi, I am %s you can call me on %s\n", h.name, h.phone)
}

func (h Human) Sing(lyrics string) {
	fmt.Println("La la la la...", lyrics)
}

func (e Employee) SayHi() {
	fmt.Printf("Hi, I am %s, I work at %s. Call me on %s\n", e.name,
		e.company, e.phone) //Yes you can split into 2 lines here.
}

// Interface Men implemented by Human, Student and Employee
type Men interface {
	SayHi()
	Sing(lyrics string)
}

func main() {
	mike := Student{Human{"Mike", 25, "222-222-XXX"}, "MIT", 0.00}
	paul := Student{Human{"Paul", 26, "111-222-XXX"}, "Harvard", 100}
	sam := Employee{Human{"Sam", 36, "444-222-XXX"}, "Golang Inc.", 1000}
	tom := Employee{Human{"Sam", 36, "444-222-XXX"}, "Things Ltd.", 5000}

	// define interface i
	var i Men

	//i can store Student
	i = mike
	fmt.Println("This is Mike, a Student:")
	i.SayHi()
	i.Sing("November rain")

	//i can store Employee
	i = tom
	fmt.Println("This is Tom, an Employee:")
	i.SayHi()
	i.Sing("Born to be wild")

	// slice of Men
	fmt.Println("Let's use a slice of Men and see what happens")
	x := make([]Men, 3)
	// these three elements are different types but they all implemented interface Men
	x[0], x[1], x[2] = paul, sam, mike

	for _, value := range x {
		value.SayHi()
	}
}
```

##### Empty Interface
*	An empty interface is an interface that doesn't contain any methods, so all types implement an empty interface. This fact is very useful when we want to store all types at some point

*	If a function uses an empty interface as its argument type, it can accept any type; if a function uses empty interface as its return value type, it can return any type.

```
// define a as empty interface
var a interface{}
var i int = 5
s := "Hello world"
// a can store value of any type
a = i
a = s

func anyOne(any interface{}) (v interface{}) {}

```

##### Method arguments of an interface

```
package main

import (
	"fmt"
	"strconv"
)

// Element 是任意类型
// List 可存任意类型数据
type Element interface{}
type List []Element

type Person struct {
	name string
	age  int
}

// 实现Stringer接口方法String()
func (p Person) String() string {
	return "(name: " + p.name + " - age: 	" + strconv.Itoa(p.age) + " years)"
}

func main() {
	list := make(List, 3)
	list[0] = 1       // an int
	list[1] = "Hello" // a string
	list[2] = Person{"Dennis", 70}

	for index, element := range list {
		// switch value := element.(type) 改写
		if value, ok := element.(int); ok {
			fmt.Printf("list[%d] is an int and its value is %d\n", index, value)
		} else if value, ok := element.(string); ok {
			fmt.Printf("list[%d] is a string and its value is %s\n", index, value)
		} else if value, ok := element.(Person); ok {
			fmt.Printf("list[%d] is a Person and its value is %s\n", index, value)
		} else {
			fmt.Printf("list[%d] is of a different type\n", index)
		}
	}
}
```

##### Embedded interfaces
```
// heap.go
type Interface interface {
	sort.Interface      // embedded sort.Interface
	Push(x interface{}) //a Push method to push elements into the heap
	Pop() interface{}   //a Pop method that pops elements from the heap
}

// sort.go
type Interface interface {
	// Len is the number of elements in the collection.
	Len() int
	// Less returns whether the element with index i should sort
	// before the element with index j.
	Less(i, j int) bool
	// Swap swaps the elements with indexes i and j.
	Swap(i, j int)
}

// io.ReadWriter
type ReadWriter interface {
	Reader
	Writer
}
```

##### Reflect
```
// get meta-data in type i, and use t to get all elements
t := reflect.TypeOf(i)
// get actual value in type i, and use v to change its value
v := reflect.ValueOf(i)
```

*	Convert the reflected types to get the values that we need.

```
var x float64 = 3.4
v := reflect.ValueOf(x)
fmt.Println("type:", v.Type())
fmt.Println("kind is float64:", v.Kind() == reflect.Float64)
fmt.Println("value:", v.Float())
```

*	 Change the values from reflect types.

```
var x float64 = 3.4
p := reflect.ValueOf(&x)
v := p.Elem()
v.SetFloat(7.1)
```


#### References
[Go Interface](https://studygolang.com/articles/7865)