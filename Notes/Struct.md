#### Struct
##### Struct Tag
*	可以通过反射获取struct的tag. Struct Tag可以提供字符串到Struct的映射能力，以便我们作转换，除此之外，还可以作为字段的元数据的配置

```
package main

import (
    "reflect"
    "fmt"
)

type User struct {
    Name string `json:"name" bson:"b_name"`
    Age int `json:"age" bson:"b_age"`
    //Name string `name`
    //Age int `age`
}

func main () {
    var u User

    //t := reflect.TypeOf(u)
    t := reflect.TypeOf(u)

    for i := 0; i < t.NumField(); i++ {
        sf := t.Field(i)
        //fmt.Println(sf.Tag)
        fmt.Println(sf.Tag.Get("json"),",",sf.Tag.Get("bson"))
    }
}
```


*	JSON字符串对象转换

```
package main

import (
        "encoding/json"
        "fmt"
)

type User struct {
        Name string `name`
        Age int `age`
}

func main () {
        var u User
        h := `{"name":"aaaa", "age":15}`

        err := json.Unmarshal([]byte(h), &u)
        if err != nil {
                fmt.Println(err)
        }else{
                fmt.Println(u)
        }

        newjson, err := json.Marshal(&u)
        if err != nil {
                fmt.Println(err)
        }else{
                fmt.Println(string(newjson))
        }
}
```

```
go get -u github.com/mailru/easyjson/
go install  github.com/mailru/easyjson/easyjson
```


#### STL
```
sqlx/Gorm // ORM

```