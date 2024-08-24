---
title: 'Golang CSP: Channel And Gorutine'
date: 2023-12-20 23:56:24
categories: Golang
tags:
- Golang
- CSP
- Channel
- Gorutine
---

In Go (Golang), channels and goroutines are key components of the language's concurrency model, which is inspired by Communicating Sequential Processes (CSP). CSP is a formal language for describing patterns of interaction in concurrent systems. The main ideas of CSP include processes communicating by sending and receiving messages over channels, and synchronization through communication rather than through shared memory.
Channels in Go are a communication mechanism designed to allow one goroutine to safely send data to another goroutine in concurrent or multi-goroutine scenarios.
<!--more-->
> Do not communicate by sharing memory; instead, share memory by communicating.

## Channels
[chan.go](https://github.com/golang/go/blob/master/src/runtime/chan.go)
```go
type hchan struct {
	qcount   uint           // total data in the queue
	dataqsiz uint           // size of the circular queue
	buf      unsafe.Pointer // points to an array of dataqsiz elements
	elemsize uint16
	closed   uint32
	elemtype *_type // element type
	sendx    uint   // send index
	recvx    uint   // receive index
	recvq    waitq  // list of recv waiters
	sendq    waitq  // list of send waiters
	lock mutex
}
```

<img src="./channel_structure.jpg" width="100%" height="100%" title="image" alt="channel_structure"/>


### UnBuffered Channels
By default channels are unbuffered, meaning that they will only accept sends (chan <-) if there is a corresponding receive (<- chan) ready to receive the sent value. It enforces a synchronous communication between the two goroutines involved in the communication

1. The producers and consumers of a channel appear in pairs, but both within the same goroutine.
    ```go
    func main() {
      c := make(chan string)
      c <- "hello" // block for ever
      msg := <-c
      fmt.Println(msg)
    }
    /*
    fatal error: all goroutines are asleep - deadlock!
    goroutine 1 [chan send]:
    main.main()
      /tmp/sandbox3222265902/prog.go:9 +0x36
    */
    ```


2. The producers and consumers of a channel operate in pairs but within different goroutines.
    ```go
    func main() {
      ms := make(chan string)
      go func(c chan string) {
        c <- "hello"
      }(ms)
      msg := <-ms
      fmt.Println(msg)  // hello
    }
    ```

3. Unbuffered channels are useful in scenarios where synchronization between two goroutines since it ensure that the sender and receiver are ready to communicate at the same time.
    ```go
    var (
      dog  = make(chan struct{})
      cat  = make(chan struct{})
      fish = make(chan struct{})
      wg   sync.WaitGroup
    )

    func Dog() {
      <-dog
      fmt.Println("dog")
      cat <- struct{}{}
    }

    func Cat() {
      <-cat
      fmt.Println("cat")
      fish <- struct{}{}
    }

    func Fish() {
      <-fish
      fmt.Println("fish")
      dog <- struct{}{}
    }

    func main() {
      for i := 0; i < 10; i++ {
        wg.Add(3)
        go func() {
          defer wg.Done()
          Dog()
        }()
        go func() {
          defer wg.Done()
          Cat()
        }()
        go func() {
          defer wg.Done()
          Fish()
        }()
      }

      // Start the cycle
      dog <- struct{}{}

      wg.Wait()
    }
    // A fatal error occurred at the end of the process due to the absence of any receiver for the last message sent on the unbuffered channel 'fish'.
    /*
    dog
    cat
    fish
    ...
    dog
    cat
    fish
    fatal error: all goroutines are asleep - deadlock!
    */
    ```


4. In some scenarios like timeout control, may lead to memory leak.
    - If there is no timeout, `doLongTask` signals the unbuffered channel `done`, and the `select` statement receives the signal. Consequently, `doLongTask` and the child coroutine can exit gracefully.
    - When a timeout occurs, the `select` statement receives the timeout signal from `time.After` and returns. Since unbuffered channel `done` has no receiver at this point, and `doLongTask` sends a signal to channel `done` after 1 second of execution, the sender will be blocked indefinitely due to the lack of a receiver and no buffer, preventing the coroutine from exiting.
    ```go
    func doLongTask(done chan bool) {
      time.Sleep(time.Second) // long task
      done <- true
    }

    func timeout(f func(chan bool)) error {
      done := make(chan bool) // unbuffered channel
      go f(done)
      // If there is no default statement in the select statement, it will block and wait for any of the case statements.
      select {
      case <-done:
        fmt.Println("done")
        return nil
      case <-time.After(time.Millisecond):
        return fmt.Errorf("timeout")
      }
    }

    func TestDoLongTask(t *testing.T) {
      for i := 0; i < 1000; i++ {
        timeout(doLongTask)
      }
      time.Sleep(time.Second * 2)
      t.Log(runtime.NumGoroutine())
    }


    /*
    === RUN   TestDoLongTask
        prog_test.go:38: 1002
    --- PASS: TestDoLongTask (3.00s)
    PASS
    */
    ```


### Buffered Channels
Golang provides buffered channels, which allow you to specify a fixed length of buffer capacity so one can send that number of data values at once. These channels are only blocked when the buffer is full.

1. To prevent memory leaks, utilize buffered channels to avoid the sender from being blocked, even in the absence of a receiver.
    ```go
    func doLongTask(done chan bool) {
      time.Sleep(time.Second) // long task
      done <- true
    }

    func timeout(f func(chan bool)) error {
      done := make(chan bool, 1) // buffered channel
      go f(done)
      // If there is no default statement in the select statement, it will block and wait for any of the case statements.
      select {
      case <-done:
        fmt.Println("done")
        return nil
      case <-time.After(time.Millisecond):
        return fmt.Errorf("timeout")
      }
    }

    func TestDoLongTask(t *testing.T) {
      for i := 0; i < 1000; i++ {
        timeout(doLongTask)
      }
      time.Sleep(time.Second * 2)
      t.Log(runtime.NumGoroutine())
    }


    /*
    === RUN   TestBufferTimeout
        prog_test.go:34: 2
    --- PASS: TestBufferTimeout (3.00s)
    PASS
    */
    ```


### Select Statement
`select` statement is used to deal with multiple communication operations. It allows a goroutine to wait on multiple communication operations and proceed as soon as one of them can proceed. This is particularly useful when working with channels to handle concurrent communication and synchronization.
  ```go
  func main() {
    taskCh := make(chan struct{})

    go func() {
      fmt.Println("First stage started...")
      time.Sleep(2 * time.Second)
      fmt.Println("First stage ending...")
      close(taskCh)
    }()

    // Wait for the first segment of a task to complete or timeout using the select statement
    select {
    case <-taskCh:
      fmt.Println("First stage completed. Continuing with the next stages...")

    case <-time.After(1 * time.Second):
      fmt.Println("Timeout occurred. Aborting further stages.")
    }

    fmt.Println("Second stage started...")
    time.Sleep(2 * time.Second)
    fmt.Println("Second stage ending...")
  }
  ```

[selectgo](https://github.com/golang/go/blob/master/src/runtime/select.go#L121)
1. The select statement in Go is a specialized statement used exclusively for sending and receiving messages on channels. This statement is blocking during execution. When there is no case statement in the select, it will block the current goroutine.
2. select is Go's language-level mechanism for I/O multiplexing. It is specifically designed to detect whether multiple channels are ready: either for reading or writing.
3. In the select statement, except for default, each case operates on a single channel, either for reading or writing.
4. In the select statement, except for default, the execution order of each case is random.
5. If there is no default statement in the select statement, it will block and wait for any of the case statements.
6. In the select statement, when performing a read operation, it is necessary to check whether the read was successful. Closed channels can also be read in the select statement.


### Channel Operations

#### 1. Sending Data (chan <-)
- If the receiving queue (recvq) is not empty, it signifies an absence of data in the buffer or the absence of a buffer. In such a scenario, retrieve a Goroutine (G) from recvq, write the data, awaken G, and conclude the sending process.
- If there is space available in the buffer, write the data into the buffer, concluding the sending process.
- If the buffer has no available space, write the data to be sent into G, add the current G to the sending queue (sendq), enter a sleep state, and await awakening by a reading

```go
func chansend(c *hchan, ep unsafe.Pointer, block bool, callerpc uintptr) bool {
  // ...
  lock(&c.lock)

  // ** Send to a closed channel cause panic **
  if c.closed != 0 {
    unlock(&c.lock)
    panic(plainError("send on closed channel"))
  }

  // ** Dequeuing from Receiver Queue **
  if sg := c.recvq.dequeue(); sg != nil {
    // Found a waiting receiver. We pass the value we want to send
    // directly to the receiver, bypassing the channel buffer (if any).
    send(c, sg, ep, func() { unlock(&c.lock) }, 3)
    return true
  }

  // ** Enqueuing to Channel Buffer **
  if c.qcount < c.dataqsiz {
    // Space is available in the channel buffer. Enqueue the element to send.
    qp := chanbuf(c, c.sendx)
    if raceenabled {
      racenotify(c, c.sendx, nil)
    }
    typedmemmove(c.elemtype, qp, ep)
    c.sendx++
    if c.sendx == c.dataqsiz {
      c.sendx = 0
    }
    c.qcount++
    unlock(&c.lock)
    return true
  }

  // ...

  // ** Block on the channel if is allowed, it then enqueues the current goroutine(mysg) to the send queue (c.sendq) and parks the goroutine (gopark). This means the goroutine will be put to sleep until a receiver completes the operation. **
  gp := getg()
  mysg := acquireSudog()
  mysg.releasetime = 0
  if t0 != 0 {
    mysg.releasetime = -1
  }
  // ...
  gp.waiting = mysg
  gp.param = nil
  c.sendq.enqueue(mysg)
  // ...
  // ** Ensure the value being sent is kept alive until the receiver copies it out. **
  KeepAlive(ep)

  // someone woke us up.
  // ...
}
```

#### 2. Receiving Data (<- chan)
- If the waiting sending queue (sendq) is not empty and there is no buffer, directly retrieve a Goroutine (G) from sendq,  wake up G, end the reading process.
- If the waiting sending queue (sendq) is not empty, indicating that the buffer is full, read data from the head of the buffer, write the data from G to the end of the buffer, wake up G, and  end the reading process.
- If there is data in the buffer, retrieve the data from the buffer and end the reading process
- If the waiting sending queue (sendq) and buffer are empty, add the current Goroutine to recvq, enter a sleep state, and await awakening by a writing Goroutine.

```go
func chanrecv(c *hchan, ep unsafe.Pointer, block bool) (selected, received bool) {
  // ...

  lock(&c.lock)

  // ** Read from a closed channel is allowed **
  if c.closed != 0 {
    if c.qcount == 0 {
      if raceenabled {
        raceacquire(c.raceaddr())
      }
      unlock(&c.lock)
      if ep != nil {
        typedmemclr(c.elemtype, ep)
      }
      return true, false
    }
    // The channel has been closed, but the channel's buffer have data.
  } else {
    // Just found waiting sender with not closed.
    if sg := c.sendq.dequeue(); sg != nil {
      // Found a waiting sender. If buffer is size 0, receive value
      // directly from sender. Otherwise, receive from head of queue
      // and add sender's value to the tail of the queue (both map to
      // the same buffer slot because the queue is full).
      recv(c, sg, ep, func() { unlock(&c.lock) }, 3)
      return true, true
    }
  }

  // ** Dequeuing from Channel Buffer **
  if c.qcount > 0 {
    // Receive directly from queue
    qp := chanbuf(c, c.recvx)
    if raceenabled {
      racenotify(c, c.recvx, nil)
    }
    if ep != nil {
      typedmemmove(c.elemtype, ep, qp)
    }
    typedmemclr(c.elemtype, qp)
    c.recvx++
    if c.recvx == c.dataqsiz {
      c.recvx = 0
    }
    c.qcount--
    unlock(&c.lock)
    return true, true
  }

  // ...
  // no sender available: block on this channel.
  gp := getg()
  mysg := acquireSudog()
  mysg.releasetime = 0
  if t0 != 0 {
    mysg.releasetime = -1
  }
  // ...
  gp.waiting = mysg
  gp.param = nil
  c.recvq.enqueue(mysg)

  // someone woke us up
  // ...
  return true, success
}

```

#### 3. Close channel


```sh

operation         | nil channel | closed channel     |
------------------------------------------------------
close(ch)         | panic       | panic              |
------------------------------------------------------
ch <-             | blocking    | panic              |
------------------------------------------------------
v, ok := <- ch    | blocking    | ok always be false |

```

[How to Gracefully Close Channels](https://go101.org/article/channel-closing.html)

1. M receivers, one sender, the sender says "no more sends" by closing the data channel
2. One receiver, N senders, the only receiver says "please stop sending more" by closing an additional signal channel
3. M receivers, N senders, any one of them says "let's end the game" by notifying a moderator to close an additional signal channel

Solutions Which Close Channels Politely: Use sync.Once or a mutex (sync.Mutex) to ensure that the channel is closed only once

  ```go
  type MyChannel struct {
    C    chan T
    once sync.Once
  }

  func NewMyChannel() *MyChannel {
    return &MyChannel{C: make(chan T)}
  }

  func (mc *MyChannel) SafeClose() {
    mc.once.Do(func() {
      close(mc.C)
    })
  }
  ```