### Synchronization Between Goroutines
Since goroutines run in the same address space, access to shared memory must be synchronized.

### Concurrency & Goroutines
Goroutines execute concurrently. In this case, main() and sayHello() were running at the same time. main() finished and sayHello() never got a chance to run!
```go
func main() {
	go sayHello()
}

func sayHello() {
	fmt.Println("hello")
}
```
### WaitGroup
A **WaitGroup** is defined in the **sync** package in the standard library. They're a simple, and powerful, way to ensure that all of the goroutines you launch have completed before you move on to do other things.

 A WaitGroup is, essentially, an atomic counter, that can be safely accessed concurrently. A WaitGroup had three methods:
- Add(i int)
- Done()
- Wait()  
When a new **WaitGroup** is created its internal counter is set to **0**.

**WaitGroup.Add**
wg.Add(1)  
The number you add will almost always be the total number of goroutines you intend to launch.

- It is important to remember that while you can call **Add** as many times as needed, once **Wait** is called, **Add** should no longer be called. Doing so may create a race condition in your code and will result in potentially non-deterministic code.

**WaitGroup.Done**
To remove an item from the group, we can call the **Done()** method. This method will decrement the counter by 1.

**WaitGroup.Wait**
The **Wait()** function on a **WaitGroup** will block execution until the internal counter is 0. This is always done after all **Add**'s have been called.

The common flow of using a WaitGroup will look like this:
1. Declare your WaitGroup
2. Call Add as many times as necessary and launch corresponding goroutines
3. Call Wait from your current routine
4. Call Done from inside each of the launched goroutines from your second step

Using Closures With A WaitGroup
The [golang.org/x/sync/errgroup#WithContext](https://godoc.org/golang.org/x/sync/errgroup#WithContext) function will create a new errgroup.Group as well as return a context.Context that others can listen to for cancellation should the errgroup.Group return an error.

ErrorGroup Vs. WaitGroup  
An **ErrorGroup** takes away a lot of the granular control that you with a **WaitGroup**, but they give you a simpler API and easier error handling.  

A **WaitGroup** provides total control, but you have to plumb in your error handling.

### Protecting Data With A Mutex
A **Mutex** allows you to synchronize goroutines for safe access to the same shared memory. Go provides two **Mutex** implementations, a **full lock**, and a **read/write** lock.
```go
var m sync.Mutex // Make a new mutex

m.Lock() // lock the mutex

// Code in here that is protected by this mutex is safe to
// read or write for all goroutines observing this mutex.
m.Unlock() // unlock the mutex
```
- [https://golang.org/pkg/sync/#Mutex](https://golang.org/pkg/sync/#Mutex)
- [https://golang.org/pkg/sync/#RWMutex](https://golang.org/pkg/sync/#RWMutex)

### Running Without A Mutex
When trying to access a map concurrently, without a mutex, the app will panic and crash.

Here are two functions that access the same map:
```go
var data = map[int]int{}

func get(i int) int {
	return data[i]
}

func set(i int) {
	data[i] = i
}
```
Call them concurrently without a mutex to synchronize access:
```go
func main() {
	var wg sync.WaitGroup

	for i := 0; i < 100; i++ {
		wg.Add(2)
		go func(i int) {
			defer wg.Done()
			set(i)
		}(i)
		go func(i int) {
			defer wg.Done()
			fmt.Println(get(i))
		}(i)
	}
	wg.Wait()
}
```
If Go detects access to the same memory in different goroutines (specifically in a map), it will panic

**Synchronizing With A Mutex**  
By using a **Mutex** in the **get** and **set** functions that wrap the **map**, we are able to synchronize access to the map and prevent a **panic** in the application.
```go
var data = map[int]int{}
var mu sync.Mutex

func get(i int) int {
	mu.Lock()
	defer mu.Unlock()
	return data[i]
}

func set(i int) {
	mu.Lock()
	defer mu.Unlock()
	data[i] = i
}
```
**Synchronizing With A RWMutex**  
We can make use of a **RWMutex** as well so that we can have multiple read locks open allowing for less lock contention in a read heavy application:
```go
var data = map[int]int{}
var mu sync.RWMutex

func get(i int) int {
	mu.RLock()
	defer mu.RUnlock()
	return data[i]
}

func set(i int) {
	mu.Lock()
	defer mu.Unlock()
	data[i] = i
}
```
**Embedding A Mutex**
embed a mutex in your own struct to synchronize access to internal data
```go
type Hits struct {
	mu sync.Mutex // methods are no longer promoted and only accessible within this package now.
	n  int
}

func (h *Hits) Inc() {
	h.mu.Lock()
	defer h.mu.Unlock()
	h.n++
}
```
**Mutex Lock Order**  
Mutexes can run in two modes: **normal** and **starvation**.

- In normal mode, waiters are queued in **FIFO** order.
- In starvation mode, ownership of the mutex is directly handed off from the unlocking goroutine to the waiter at the front of the queue.

### Use Mutexes To Build A WaitGroup
```go
type WaitGroup struct {
	quit    chan (struct{})
	counter int
}

func (wg *WaitGroup) Done() {
	wg.counter += -1
	if wg.counter <= 0 {
		if wg.quit != nil {
			close(wg.quit)
		}
	}
}

func (wg *WaitGroup) Add(n int) {
	wg.counter += n
}

func (wg *WaitGroup) Wait() {
	if wg.quit == nil {
		wg.quit = make(chan struct{})
	}
	<-wg.quit
}
```
Enabling Race Detection  
To turn on the race detector, pass in the **-race** flag:  
```go run -race ./main.go```

Adding a mutex to the WaitGroup will allow us to synchronize access to memory.
```go
type WaitGroup struct {
	mu      sync.Mutex
	quit    chan (struct{})
	counter int
}

func (wg *WaitGroup) Done() {
	wg.mu.Lock()
	defer wg.mu.Unlock()
	wg.counter += -1
	if wg.counter <= 0 {
		if wg.quit != nil {
			close(wg.quit)
		}
	}
}

func (wg *WaitGroup) Add(n int) {
	wg.mu.Lock()
	defer wg.mu.Unlock()
	wg.counter += n
}

func (wg *WaitGroup) Wait() {
	wg.mu.Lock()
	if wg.quit == nil {
		wg.quit = make(chan struct{})
	}
	wg.mu.Unlock()
	<-wg.quit
}
```
Using A Read/write Mutex  
Using **sync.RWMutex** we can lock access to data based on how that data is being used.

An **sync.RWMutex** is a reader/writer mutual exclusion lock. The lock can be held by an arbitrary number of readers or a single writer.
```go
type WaitGroup struct {
	mu      sync.RWMutex
	quit    chan (struct{})
	counter int
}

func (wg *WaitGroup) Remaining() int {
	wg.mu.RLock()
	defer wg.mu.RUnlock()
	return wg.counter
}
```

