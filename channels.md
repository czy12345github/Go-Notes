## Unbuffered Channels (eg. `ch := make(chan int)`)
A send operation on an unbuffered blocks the sending goroutine until another goroutine executes a corresponding receive on the same channel,
at which point the value is transmitted and both goroutines may continue. Conversely, if the receive operation was attempted first, the
receiving goroutine is blocked until another goroutine performs a send on the same channel.

可用于同步 goroutines:
```go
func main() {
	done := make(chan struct{})
	go func() {
		time.Sleep(1 * time.Second)
		log.Print("goroutine1 ends")
		done <- struct{}{} // signal goroutine2
	}()
	go func() {
		<-done // wait for goroutine1 to finish
		log.Print("goroutine2 ends")
	}()

	time.Sleep(2 * time.Second)
}
```

## Pipelines
Channels can be used to connect goroutines together so that the output of one is the input to another. This is called a *pipeline*.

示例：
```go
func main() {
	naturals := make(chan int)
	squares := make(chan int)

	go func() {
		for x := 0; x < 100; x++ {
			naturals <- x
		}
		close(naturals)
	}()

	go func() {
		for x := range naturals {
			squares <- x * x
		}
		close(squares)
	}()

	for x := range squares {
		fmt.Println(x)
	}
}
```

## Unidirectional Channel Types
channel作为函数参数时，其类型应指定为*receive-only*或*send-only*以避免误用。

示例：
```go
func main() {
	naturals := make(chan int)
	squares := make(chan int)

	go counter(naturals)
	go squarer(squares, naturals)
	printer(squares)
}

func counter(out chan<- int) {
	for x := 0; x < 100; x++ {
		out <- x
	}
	close(out)
}

func squarer(out chan<- int, in <-chan int) {
	for x := range in {
		out <- x * x
	}
	close(out)
}

func printer(in <-chan int) {
	for x := range in {
		fmt.Println(x)
	}
}
```

## Buffered Channels (eg. `ch := make(chan int, 3)`)
If the channel is full, the send operation blocks until space is made available by another goroutine's receive. Conversely, if the
channel is empty, a receive operation blocks until a value is sent by another goroutine.

When applied to a channel, the built-in *cap* function returns the channel's buffer capacity, and the built-in *len* function returns
the number of elements currently buffered.
