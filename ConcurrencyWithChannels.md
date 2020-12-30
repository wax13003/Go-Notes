## Channels
Channels are a typed conduit through which you can send and receive values.
```
ch <- // data is going into the channel
<- ch // data is coming out of the channel
```

### Buffered Channels
By default channels are unbuffered -- sends on a channel will block until there is something ready to receive from it.

```go
// Adding a second argument to the make function creates a buffered channel
    messages := make(chan string, 2)
```
Use buffered channels cautiously. They do not guarantee delivery of the message.

### Channels Are Not Message Queues  
Only one goroutine will receive a message sent down a channel. If multiple goroutines are listening to a channel, only one will receive the message.  
  
It is also possible, and likely, that one goroutine will receive more messages than another goroutine.
```go
ch := make(chan int)
for i := 0; i < N; i++ {
	go func(i int) {
		for m := range ch {
			fmt.Printf("routine %d received %d\n", i, m)
		}
	}(i)
}

for i := 0; i < N; i++ {
	ch <- i
}
time.Sleep(time.Second)
```
### Closing Channels
You can signal that there are no more values in a channel by closing it. Receivers can test to see if a channel is closed during a read.
```close(c)```  

**Detecting Closed Channels On Read**  
This can be done, as with type detection and maps, by using the optional, boolean, second argument that comes with reading from a channel.
```go
package main

import "fmt"

func main() {
	c := make(chan int)
	go count(c)
	for {
		i, ok := <-c
		if !ok {
			fmt.Println("closed channel")
			return
		}
		fmt.Printf("read %d from channel\n", i)
	}
}

func count(c chan int) {
	for i := 0; i < 10; i++ {
		c <- i
	}
	close(c)
}
```

**Zero Value On Closed Read**
It is important to ignore the value returned on a closed read from a channel, as the runtime will explicitly set your values back to the zero values for the type defined in the channel:
```go
    var ok bool
    
	// Proper way to read
	if u1, ok = <-c; ok {
		fmt.Printf("read successful: %+v\n", u1)
	} else {
		fmt.Println("attempted read of closed channel, ignore value returned")
    }
```

### Ranging Through Channels
You can use the **range** keyword to read channels. When using **range**, it is a shortcut for using a **for** loop with on conditionals.

with **range**:
```go
func rangeLoop(messages chan string, quit chan struct{}) {
	// loop until the channel is closed
	for m := range messages {
		fmt.Println(m)
	}

	close(quit)
}
```
**for** loop:
```go
func forLoop(messages chan string, quit chan struct{}) {
	// loop until the channel is closed
	for {
		m, ok := <-messages
		if !ok {
			// channel was closed
			break
		}
		fmt.Println(m)
	}
	close(quit)
}
```
### Reading From Closed Buffered Channels
Even if a buffered channel is closed, you can still read all the data written to it.
```go
func main() {
	c := make(chan int, 10)

	// Write values into a channel
	for i := 0; i < cap(c); i++ {
		c <- i
	}

	// Close the channel
	close(c)

	// You can still read all the data in the channel
	for i := range c {
		fmt.Println(i)
	}

	// but you can no longer write to the channel as it's closed
	c <- 11
}
```
### Channels As Signals
You can also use a channel as a **signal** only used to block a process.
```go
func main() {
	quit := make(chan struct{})
	go monitor()
	<-quit
}

func monitor() {
	for {
		log.Println("monitoring...")
		time.Sleep(time.Second)
	}
}
```
This program will block until you terminate the program.

### Signal Vs. WaitGroup
sync.WaitGroup
```go
func main() {
	wg := &sync.WaitGroup{}
	wg.Add(1)
	go count(5, wg)
	wg.Wait()
}

func count(i int, wg *sync.WaitGroup) {
	defer wg.Done()
	for j := 0; j < i; j++ {
		fmt.Println("processing...", j)
	}
}
```
re-written with a channel:
```go
func main() {
	quit := make(chan struct{})
	go count(5, quit)
	<-quit
}

func count(i int, quit chan struct{}) {
	defer close(quit)
	for j := 0; j < i; j++ {
		fmt.Println("processing...", j)
	}
}
```
### Using Select
You can use a **select** statement to allow a goroutine to operate on multiple communication operations

Select With Default
If no operations inside a **select** statement can process, the **select** will block. To work around this problem, you can add a **default** statement to your select:
```go
package main

import "time"

func main() {
	tick := time.Tick(100 * time.Millisecond)
	done := time.After(500 * time.Millisecond)
	for {
		select {
		case <-tick:
			print("tick")
		case <-done:
			print(".done")
			return
		default:
			print(".")
			time.Sleep(10 * time.Millisecond)
		}
	}
}
```

### Uni-Directional Channels
By default channels are bi-directional, meaning you can both send and receive data from the channel.  

A common use for uni-directional channels is when you are passing a channel as an argument or receiving a channel as return value. This allows for control of the channel for the function/method and prevents outside callers from polluting the channel.

**Receive Only Channels**
```go
package main

import "fmt"

func main() {
	// make a full directional channel
	c := make(chan string)
	quit := make(chan struct{})

	go receive(c, quit)

	// Write into the channel
	for _, s := range []string{"one", "two", "three"} {
		c <- s
	}
	close(c)
	// wait for quit channel to close
	<-quit
}

// receive takes the first argument as a read only channel
func receive(c <-chan string, quit chan struct{}) {
	// read from the channel
	for s := range c {
		fmt.Println(s)
	}
	// You can't write to this channel:
	//c <- "foo"

	// You also can't close a read only channel.  This is to protect the sender from you also polluting this channel:
	//close(c)

	close(quit)
}
```

*Receive Only Channel Limitations：*
- You can't write to a receive only channel, that will result in a compile time error
- You can't close a receive only channel, that will also result in a compile time error

**Send Only Channels**
```go
package main

import "fmt"

func main() {
	// make a full directional channel
	c := make(chan string)

	go send(c)

	// read from the channel
	for s := range c {
		fmt.Println(s)
	}

}

// send takes the first argument as a send only channel
func send(c chan<- string) {
	// Write into the channel
	for _, s := range []string{"one", "two", "three"} {
		c <- s
	}
	// You can't read from this channel:
	//s := <-c

	// it is ok to close this channel
	close(c)
}
```
*Send Only Channel Limitations*
- You can't read from a send only channel, that will result in a compile time error

### Panics
Here are the four ways you can encounter a **panic** with channels:
```go
c1 := make(chan int)
close(c1)
c1 <- 1 // panic: send on a closed channel

c2 := make(chan int)
close(c2)
close(c2) // panic: closeing a channel that is already closed

var c3 chan int
c3 <- 1 // panic: send on a nil channel

var c4 chan bool
<-c4 // panic: read from a nil channel: all goroutines are asleep
```
### Work Pools And Rate Limiting
- [Worker Pools](https://gobyexample.com/worker-pools)
- [Rate Limiting](https://gobyexample.com/rate-limiting)