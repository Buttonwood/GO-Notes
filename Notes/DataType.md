#### 1.	String
*	Go uses the UTF-8 character set.

```
var str1 string = "ahfahfaf"
var str2 string = `hello
	world`
```

*	String objects do not now allow value change. 

```
var s string = "hello"
s[0] = 'c' // Error
```

*	For changing a string, a new string has to be created using the old string.

```
s := "hello"
c := []byte(s) //convert string to []byte type
c[0] = 'c'
s2 := string(c)  // convert back to string type
fmt.Printf("%s\n", s2)

s := "hello"
// returns substring from index 1 till the end.
s = "c" + s[1:] 
fmt.Printf("%s\n", s)
```

*	`+` operator cane be used to concatenate two strings.

```
s := "hello,"
m := " world"
a := s + m
fmt.Printf("%s\n", a)
```

#### `iota` enumerate
```
const(
	x = iota  // x == 0
	y = iota  // y == 1
	z  		  // z == 2
}

// set v = 0.
const v = iota 

const ( 
	// e=0,f=0,g=0 values of iota are same in one line.
	e, f, g = iota, iota, iota 
)
```

#### 2.	Array
*	Since length is a part of the array type, `[3]int` and `[4]int` are different types. We cannot change the length of arrays.

*	Arrays are passed as copy rather than references when passed to a function as arguments.

```
var arr [n]type

var arr [10]int
arr[0] = 42      // array is 0-based
arr[1] = 13      // assign value to element

a := [3]int{1, 2, 3} 
b := [10]int{1, 2, 3} 
c := [...]int{4, 5, 6}

doubleArray := [2][4]int{[4]int{1, 2, 3, 4}, [4]int{5, 6, 7, 8}}
easyArray := [2][4]int{{1, 2, 3, 4}, {5, 6, 7, 8}}
```

#### 3.	Slice(dynamic array)
*	We use `[…]` to let Go calculate `array` length but use `[]` to define `slice` only. 

*	Slices do not have length, slices point to arrays which have lengths.

*	`slice` is a reference type, so any changes will affect other variables pointing to the same slice or array.

```
var fslice []int
slice := []byte {'a', 'b', 'c', 'd'}

var ar = [10]byte {'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j'}
var a, b []byte
a = ar[2:5]
b = ar[3:5]

// ar[:n] equals to ar[0:n]
// ar[n:] equals to ar[n:len(ar)]
// ar[:] to slice whole array

len()
cap()
append()
copy()
```

#### 4.	Maps
*	`map` is disorderly.
*	`map` doesn't have a fixed length.
*	It's a reference type just like `slice`.
*	`len()` and `cap()`

```
map[keyType]valueType

// 1.声明式
var numbers map[string] int
// 2.定义式
numbers := make(map[string]int)
// assign value by key
numbers["one"] = 1

// 3.赋值式
rating := map[string]float32 {"C":5, "Go":4.5, "Python":4.5, "C++":2 }
if csharpRating, ok := rating["C#"]; ok {
	
} else {
	
}
// delete element with key "c"
delete(rating, "C")
```

#### 5.	`make(T)` and `new(T)`
*	`make` does memory allocation for built-in models, such as `map`, `slice`, and `channel`, while `new` is for types' memory allocation.
*	`new(T)` allocates zero-value to type T's memory, returns its memory address, which is the value of type `*T`. By Go's definition, it returns a pointer which points to type T's zero-value.