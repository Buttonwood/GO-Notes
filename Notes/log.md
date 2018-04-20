##### log
```
package main
import "log"

func init() {
	log.SetPrefix("TRACE: ")
	log.SetFlags(log.Ldate | log.Lmicroseconds | log.Llongfile)
}

func main() {
	log.Println("message")
	log.Fatalln("fatal message")
	log.Panicln("panic message")
}
```

```
const (
	Ldate = 1 << iota 	// 1 << 0 = 000000001 = 1	//日期示例： 2009/01/23
	Ltime 				// 1 << 1 = 000000010 = 2	//时间示例: 01:23:23
	Lmicroseconds 		// 1 << 2 = 000000100 = 4	//毫秒示例: 01:23:23.123123.
	Llongfile 			// 1 << 3 = 000001000 = 8	//绝对路径和行号: /a/b/c/d.go:23
	Lshortfile 			// 1 << 4 = 000010000 = 16	//文件和行号: d.go:23.
	LstdFlags = Ldate(1) | Ltime(2) // 00000011 = 3	//Go提供的标准抬头信息
)
```

```
package main
import (
	"io"
	"io/ioutil"
	"log"
	"os"
)

var (
	// Logger type pointer variables
	Trace 	*log.Logger
	Info 	*log.Logger
	Warning *log.Logger
	Error 	*log.Logger
)

func init() {
	file, err := os.OpenFile("errors.txt", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0666)
	if err != nil {
		log.Fatalln("Failed to open error log file:", err)
	}
	Trace = log.New(ioutil.Discard, "TRACE: ", log.Ldate|log.Ltime|log.Lshortfile)
	Info  = log.New(os.Stdout,      "INFO: ",  log.Ldate|log.Ltime|log.Lshortfile)
	Warning = log.New(os.Stdout, "WARNING: ", log.Ldate|log.Ltime|log.Lshortfile)
	Error = log.New(io.MultiWriter(file, os.Stderr), "ERROR: ", log.Ldate|log.Ltime|log.Lshortfile)
}

func main() {
	Trace.Println("I have something standard to say")
	Info.Println("Special Information")
	Warning.Println("There is something you need to know about")
	Error.Println("Something has failed")
}
```