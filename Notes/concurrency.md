#### Concurrency
1.	Parallelism is about doing a lot of things at once.
2.	Concurrency is about managing a lot of things at once.
3.	Parallelism can only be achieved when multiple pieces of code are executing simultaneously against different physical processors.

```
// wg is used to wait for the program to finish.
var wg sync.WaitGroup

// When the value of a WaitGroup is greater than zero, the Wait method will block.
wg.Add(2)

go func(){
	// Schedule the call to Done to tell main we are done.
	defer wg.Done()
}
```

```
import "runtime"
// Allocate a logical processor for every available core.
runtime.GOMAXPROCS(runtime.NumCPU())
// let the CPU execute other goroutines
runtime.Gosched() 
```

```
package main

import (
    "fmt"
    "runtime"
)

func say(s string) {
    for i := 0; i < 5; i++ {
        runtime.Gosched()
        fmt.Println(s)
    }
}

func main() {
    go say("world") // create a new goroutine
    say("hello")    // current goroutine
}
```



##### Goroutines and Channels
1.	Goroutines are like threads, but use far less memory and require less code to use.
2.	Channels are data structures that let you send typed messages between goroutines with synchronization built in. 

```
func log(msg string){
	//... some logging code here
}

// Elsewhere in our code after we've discovered an error.
go log("something dire happened")
```

##### Race conditions

```
go build -race // Build the code using the race detector flag
./example // Run the code
```

多加了一个`-race`标志，这样生成的可执行程序就自带了检测资源竞争的功能.

##### Locking shared resources

Atomic functions provide low-level locking mechanisms for synchronizing access to
integers and pointers. 

```
import (
	"fmt"
	"runtime"
	"sync"
	"sync/atomic"
)

var (
	counter int64
	wg sync.WaitGroup
)

func main() {
	wg.Add(2)
	go incCounter(1)
	go incCounter(2)
	wg.Wait()
	fmt.Println("Final Counter:", counter)
}

func incCounter(id int) {
	defer wg.Done()
	for count := 0; count < 2; count++ {
		atomic.AddInt64(&counter, 1)
		runtime.Gosched()
	}	
}
```

```
atomic.LoadInt64(&aInt)
atomic.StoreInt64(&aInt, 1)
```

##### Mutexes
A mutex is used to create a critical section around code that ensures only one goroutine at a time can execute that code section.

```
package main
import (
	"fmt"
	"runtime"
	"sync"
	//"sync/atomic"
)

var (
	counter int64
	wg sync.WaitGroup
	mutex sync.Mutex
)

func main() {
	wg.Add(2)
	go incCounter(1)
	go incCounter(2)
	wg.Wait()
	fmt.Println("Final Counter:", counter)
}

func incCounter(id int) {
	defer wg.Done()
	for count := 0; count < 2; count++ {
		mutex.Lock()
		{
			value := counter
			runtime.Gosched()
			value++
			counter = value
		}
		mutex.Unlock()
	}
}
```

##### Channels
*	You also have `channels` that synchronize goroutines as they send and receive the resources they need to share between each other.
*	When declaring a channel, the type of data that will be shared needs to be specified.
*	Values and pointers of built-in, named, struct, and reference types can be shared through a channel.
*	Creating a channel in Go requires the use of the built-in function `make`.
*	Sending a value or pointer into a channel requires the use of the `<-` operator.
*	For another goroutine to receive that string from the channel, we use the same `<-` operator

```
ci := make(chan int)
cs := make(chan string)
cf := make(chan interface{})

// Unbuffered channel of integers.
unbuffered := make(chan int)

// Buffered channel of strings.
buffered := make(chan string, 10)
// Send a string through the channel.
buffered <- "Gopher"
// Receive a string from the channel.
value := <- buffered
// 关闭channel
close(buffered)
```

###### Unbuffered channels
*	An unbuffered channel is a channel with no capacity to hold any value before it’s
received. These types of channels require both a sending and receiving goroutine to
be ready at the same instant before any send or receive operation can complete. 

*	发送方和接收方同步,属于同步通道,否则会阻塞

```
package main

import (
    "fmt"
)

func main() {
    ch := make(chan int)

    go func(){
        var s int = 0
        for i := 0; i<10; i++ {
            s += i
        }
        ch <- s
     }()
    //0xc8200180c0
	fmt.Println(ch)
	// 45
    fmt.Println(<-ch)
}
```

*	管道:一个通道的输出，当成下一个通道的输入

```
package main

import (
    "fmt"
)

func main() {
    ch1 := make(chan int)
    ch2 := make(chan int)

    go func(){
        ch1 <- 100
    }()

    go func(){
        v := <- ch1
        ch2 <- v
    }()
    //fmt.Println(<-ch1)
    fmt.Println(<-ch2)
}

```

*	等待

```
package main
import (
	"fmt"
	"math/rand"
	"sync"
	"time"
)

var wg sync.WaitGroup

func init(){
	rand.Seed(time.Now().UnixNano())
}

func main() {
	// Create an unbuffered channel
	cnt := make(chan int)
	wg.Add(2)
	go player("Nadal", cnt)
	go player("Djokovic", cnt)
	// Start the set.
	cnt <- 1
	// Wait for the game to finish.
	wg.Wait()
}

func palyer(name string, cnt chan int){
	defer wg.Done()
	for {
		ball, ok := <- cnt
		if !ok {
			fmt.Printf("Player %s Won\n", name)
			return
		}
		n := rand.Intn(100)
		if n%13 == 0 {
			fmt.Printf("Player %s Missed\n", name)
			// Close the channel to signal we lost.
			close(cnt)
			return
		}
		// Display and then increment the hit count by one.
		fmt.Printf("Player %s Hit %d\n", name, ball)
		ball++
		// Hit the ball back to the opposing player.
		cnt <- ball
	}
}
```

###### Buffered channels
*	A buffered channel is a channel with capacity to hold one or more values before they’re received. These types of channels don’t force goroutines to be ready at the same instant to perform sends and receives. 
*	A receive will block only if there’s no value in the channel to receive. 
*	A send will block only if there’s no available buffer to place the value being sent. 
*	Unbuffered channels provide a guarantee between an exchange of data. Buffered channels do not.
*	`cap(ch)` and `len(ch)`
*	We can use `range` to operate on buffer channels as in `slice` and `map`.(注意关闭channel,否则一直阻塞)

```
package main
import (
	"fmt"
	"math/rand"
	"sync"
	"time"
)

const (
	// Number of goroutines to use.
	nGoroutines = 4
	// Amount of work to process.
	taskLoad = 10
)

// wg is used to wait for the program to finish.
var wg sync.WaitGroup

func init(){
	rand.Seed(time.Now().UnixNano())
}

func main() {
	// Create a buffered channel to manage the task load.
	tasks := make(chan string, taskLoad)

	// Launch goroutines to handle the work.
	wg.Add(numberGoroutines)
	for gr := 1; gr <= numberGoroutines; gr++ {
		go worker(tasks, gr)
	}

	// Add a bunch of work to get done.
	for post := 1; post <= taskLoad; post++ {
		tasks <- fmt.Sprintf("Task : %d", post)
	}

	// Close the channel so the goroutines will quit
	// when all the work is done.
	close(tasks)

	// Wait for all the work to get done.
	wg.Wait()
}

func worker(tasks chan string, worker int) {
	defer wg.Done()
	for {
		// Wait for work to be assigned.
		task, ok := <- tasks
		if !ok {
			// This means the channel is empty and closed.
			fmt.Printf("Worker: %d : Shutting Down\n", worker)
			return
		}
		
		// Display we are starting the work.
		fmt.Printf("Worker: %d : Started %s\n", worker, task)

		// Randomly wait to simulate work time.
		sleep := rand.Int63n(100)
		time.Sleep(time.Duration(sleep) * time.Millisecond)

		// Display we finished the work.
		fmt.Printf("Worker: %d : Completed %s\n", worker, task)
	}
}
```

```
func mirroredQuery() string {
    responses := make(chan string, 3)
    go func() { responses <- request("asia.gopl.io") }()
    go func() { responses <- request("europe.gopl.io") }()
    go func() { responses <- request("americas.gopl.io") }()
    return <- responses // return the quickest response
}
func request(hostname string) (response string) { /* ... */ }
```

```
package main

import (
    "fmt"
)

func fibonacci(n int, c chan int) {
    x, y := 1, 1
    for i := 0; i < n; i++ {
        c <- x
        x, y = y, x+y
    }
    close(c)
}

func main() {
    c := make(chan int, 10)
    go fibonacci(cap(c), c)
    for i := range c {
        fmt.Println(i)
    }
}
```


##### 单向通道
*	限制一个通道只可以接收，但是不能发送；或限制一个通道只能发送，但是不能接收。
*	定义单向通道也很简单，只需要在定义的时候，带上`<-`即可。

```
var send chan <- int    // only send
var receive <- chan int // only receive
```

*	单向通道应用于函数或者方法的参数比较多

```
func counter(cnt chan <- int){}
```


#### Concurrency Patterns
##### Runner
*	channels can be used to monitor the amount of time a program is running and terminate the program if it runs too long. 
*	This pattern is useful when developing a program that will be scheduled to run as a background task process. 

```
package runner
import (
	"errors"
	"os"
	"os/signal"
	"time"
)

type Runner struct {
	interrupt chan os.Signal
	complete chan error
	timeout <- chan time.Time
	tasks []func(int)
}

var ErrTimeout = errors.New("received timeout")
var ErrInterrupt = errors.New("received interrupt")

func New(d time.Duration) *Runner {
	return &Runner{
		interrupt: make(chan os.Signal, 1),
		complete: make(chan error),
		timeout: time.After(d),
}

func (r *Runner) Add(tasks ...func(int)) {
	r.tasks = append(r.tasks, tasks...)
}

func (r *Runner) Start() error {
	signal.Notify(r.interrupt, os.Interrupt)
	go func() {
		r.complete <- r.run()
	}()
	select {
		case err := <- r.complete: return err
		case <- r.timeout: return ErrTimeout
	}
}

func (r *Runner) run() error {
	for id, task := range r.tasks {
		if r.gotInterrupt() {
			return ErrInterrupt
		}
		task(id)
	}
	return nil
}

func (r *Runner) gotInterrupt() bool {
	select {
		case <- r.interrupt:
			signal.Stop(r.interrupt)
			return true
		default:
			return false
	}
}
```
##### Pooling
*	use a buffered channel to pool a set of resources that can be shared and individually used by any number of goroutines.
*	This pattern is useful when you have a static set of resources to share, such as database connections or memory buffers.

```
package pool

import (
	"errors"
	"log"
	"io"
	"sync"
)

type Pool struct {
	// 互斥锁
	m sync.Mutex
	// 资源
	resources chan io.Closer
	// 创建新资源
	factory func() (io.Closer, error)
	// 是否关闭
	closed bool
}

var ErrPoolClosed = errors.New("Pool has been closed.")

func New(fn func() (io.Closer, error), size uint) (*Pool, error) {
	if size <= 0 {
		return nil, errors.New("Size value too small.")
	}
	return &Pool{
		factory: fn,
		resources: make(chan io.Closer, size),
	}, nil
}

// 可以获取就获取一个,不能就创建一个
func (p *Pool) Acquire() (io.Closer, error) {
	select {
		case r, ok := <- p.resources:
			log.Println("Acquire:", "Shared Resource")
			if !ok {
				return nil, ErrPoolClosed
			}
			return r, nil
		default:
			log.Println("Acquire:", "New Resource")
			return p.factory()
	}
}

// 释放资源池
func (p *Pool) Release(r io.Closer) {
	p.m.Lock()
	defer p.m.Unlock()
	if p.closed {
		r.Close()
		return
	}
	select {
		case p.resources <- r:
			log.Println("Release:", "In Queue")
		default:
			log.Println("Release:", "Closing")
			r.Close()
	}
}

// 关闭资源池
func (p *Pool) Close() {
	p.m.Lock()
	defer p.m.Unlock()
	if p.closed {
		return
	}
	p.closed = true
	close(p.resources)
	for r := range p.resources {
		r.Close()
	}
}
```

##### Work
*	Use an unbuffered channel to create a pool of goroutines that will perform and control the amount of work that gets done concurrently.
*	Unbuffered channels provide a guarantee that data has been exchanged between two goroutines.

```
package work
import "sync"

// Worker must be implemented by types that want to use the work pool.
type Worker interface {
	Task()
}

// Pool provides a pool of goroutines that can execute any Worker tasks that are submitted.
type Pool struct {
	work chan Worker
	wg sync.WaitGroup
}

// New creates a new work pool.
func New(maxGoroutines int) *Pool {
	p := Pool{ work: make(chan Worker),}
	p.wg.Add(maxGoroutines)
	for i := 0; i < maxGoroutines; i++ {
		go func() {
			for w := range p.work {
				w.Task()
			}
			p.wg.Done()
		}()
	}
	return &p
}

// Run submits work to the pool.
func (p *Pool) Run(w Worker) {
	p.work <- w
}

// Shutdown waits for all the goroutines to shutdown.
func (p *Pool) Shutdown() {
	close(p.work)
	p.wg.Wait()
}
```


##### 读写锁
```
package common

//安全的Map
type SynchronizedMap struct {
	rw *sync.RWMutex
	data map[interface{}]interface{}
}

//存储操作
func (sm *SynchronizedMap) Put(k, v interface{}){
	sm.rw.Lock()
	defer sm.rw.Unlock()
	sm.data[k]=v
}

//获取操作
func (sm *SynchronizedMap) Get(k interface{}) interface{}{
	sm.rw.RLock()
	defer sm.rw.RUnlock()
	return sm.data[k]
}

//删除操作
func (sm *SynchronizedMap) Delete(k interface{}) {
	sm.rw.Lock()
	defer sm.rw.Unlock()
	delete(sm.data,k)
}

//遍历Map，并且把遍历的值给回调函数，可以让调用者控制做任何事情
func (sm *SynchronizedMap) Each(cb func (interface{},interface{})){
	sm.rw.RLock()
	defer sm.rw.RUnlock()
	for k, v := range sm.data {
		cb(k,v)
	}
}

//生成初始化一个SynchronizedMap
func NewSynchronizedMap() *SynchronizedMap{
	return &SynchronizedMap{
		rw:new(sync.RWMutex),
		data:make(map[interface{}]interface{}),
	}
}
```

##### `select` to listen to many channels
*	`select` is blocking by default and it continues to execute only when one of channels has data to send or receive.
*	If several channels are ready to use at the same time, `select` chooses which to execute randomly.

```
package main

import "fmt"

func fibonacci(c, quit chan int) {
    x, y := 1, 1
    for {
        select {
        case c <- x:
            x, y = y, x+y
        case <-quit:
            fmt.Println("quit")
            return
        }
    }
}

func main() {
    c := make(chan int)
    quit := make(chan int)
    go func() {
        for i := 0; i < 10; i++ {
            fmt.Println(<-c)
        }
        quit <- 0
    }()
    fibonacci(c, quit)
}
```

###### Timeout
```
    c := make(chan int)
    o := make(chan bool)
    go func() {
        for {
            select {
            case v := <-c:
                println(v)
            case <- time.After(5 * time.Second):
                println("timeout")
                o <- true
                break
            }
        }
    }()
```


#####  `runtime`
```
runtime.Goexit()
runtime.Gosched()
runtime.NumCPU() int
runtime.NumGoroutine() int
runtime.GOMAXPROCS(n int) int
```


###### Examples
```
package main

import (
	"fmt"
	"time"
)

func printCount(c chan int){
	num := 0
	for num >= 0 {
		num <- c
		fmt.Print(num, ";")
	}
}

func main() {
	c := make(chan int)
	a := []int{8,7,6,5,4,3,2,1}

	go printCount(c)

	for _, v := range a {
		c <- v
	}	
}
```