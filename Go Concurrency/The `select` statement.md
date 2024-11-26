The `select` statement is the control structure for concurrency in Go. `select` allows a goroutine to **wait on multiple** communication operations. 

You can have non-blocking operations on channels, provided that you have implemented your `select` blocks appropriately.

A `select` statement is not evaluated sequentially, as all of its channels are examined simultaneously:
- If none of the channels in a select statement are ready, the `select` statement blocks (_waits_) until one of the channels is ready.
- **If multiple channels of a** `select` **statement are ready, then the Go runtime makes a random selection from the set of these ready channels**.

> [!tip]
> a select statement should always contain a case(usually a default case) for an exit strategy so that the goroutine is not blocked forever if all of the case statements are blocked.

- a `select` without any cases (`select{}`) waits forever.

One advantage of `select` choosing at random is that it prevents one of the most common causes of **deadlocks**: **acquiring locks** in an **inconsistent order**.

> [!warning] If you have two goroutines that both access the same two channels, they must be accessed in the same order in both goroutines, or they will _deadlock_.

```go
func main() {
    ch1 := make(chan int)
    ch2 := make(chan int)
    go func() {
        inGoroutine := 1
        ch1 <- inGoroutine
        fromMain := <-ch2
        fmt.Println("goroutine:", inGoroutine, fromMain)
    }()
    inMain := 2
    ch2 <- inMain
    fromGoroutine := <-ch1
    fmt.Println("main:", inMain, fromGoroutine)
}

// Output: fatal error: all goroutines are asleep - deadlock!
// The goroutine that is explicitly launched cannot proceed until `ch1` is read, and the main goroutine cannot proceed until `ch2` is read.
```

If the channel read and the channel write in the main goroutine are wrapped in a `select`, deadlock is avoided:
```go
func main() {
    ch1 := make(chan int)
    ch2 := make(chan int)
    go func() {
        inGoroutine := 1
        ch1 <- inGoroutine
        fromMain := <-ch2
        fmt.Println("goroutine:", inGoroutine, fromMain)
    }()
    inMain := 2
    var fromGoroutine int
    select {
    case ch2 <- inMain:
    case fromGoroutine = <-ch1:
    }
    fmt.Println("main:", inMain, fromGoroutine)
}
```

Because a `select` checks whether any of its cases can proceed, the deadlock is avoided. The goroutine that is launched explicitly wrote the value 1 into `ch1`, so the read from `ch1` into `fromGoroutine` in the main goroutine is able to succeed.
# for-select
Since `select` is responsible for communicating over a number of channels, it is often embedded within a `for` loop. This is so common that the combination is often referred to as a `for-select` loop.
```go
for {
    select {
    case <-done:
        return
    case v := <-ch:
        fmt.Println(v)
    }
}
```

> [!warning]
> When using a `for-select` loop, you must include a way to exit the loop. Although, in this case, we should not use `default` case to encase the escape logic.
> 
> Having a `default` case inside a `for-select` loop is almost always the wrong thing to do. It will be triggered every time through the loop when there’s nothing to read or write for any of the cases. This makes your `for` loop run constantly, which uses a great deal of CPU.

#### Common Patterns:
###### Ending a goroutine:

```go
func gen(min, max int, createNumber chan int, end chan bool) {
    time.Sleep(time.Second)
    for {
        select {
        case createNumber <- rand.Intn(max-min) + min:
	        return
        case <-end:
            fmt.Println("Ended!")
            return
        case <-time.After(4 * time.Second):
            fmt.Println("time.After()!")
            return
        }
    }
}

```

###### Sending iteration variables out on a channel:
Oftentimes you’ll want to convert something that can be iterated over into values on a channel. This is nothing fancy, and usually looks something like this:
```go
for _, s := range []string{"a", "b", "c"} {
	select {
	case <-done:
		return
	case stringStream <- s:
	}
}
```



