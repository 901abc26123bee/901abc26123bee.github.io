---
title: Debouncing mechanism
date: 2023-12-25 12:38:26
categories: Debounce
tags:
- Golang
- Debounce
---

Debouncing mechanism can be useful in scenarios where you want to handle events that may be triggered frequently but should only be processed once within a certain time window.
- Button Clicks
- Event Handling
- Throttling

## Debouncing to ensure that only the final input registers or prevails

```sh

--|--|--|--|--|--|--|--|-------------------|--|--|--|--|
                       trigger                         trigger
```

[reference code](https://github.com/bep/debounce/blob/master/debounce.go)
This code snippet provides a simpler and more lightweight debouncing mechanism. The debouncer function takes a callback function and ensures that it is called only after a specified time duration has passed since the last invocation.

``` go
const debounceDelay = 1 * time.Second
const eventTriggerInterval = 500 * time.Millisecond

func main() {
	// Create a debouncer with a 500ms delay
	debounce := New(debounceDelay)

	// Create a channel to receive events
	eventChannel := make(chan struct{})

	// Goroutine to listen to the channel and debounce events
	go func() {
		for {
			// Wait for an event to be received on the channel
			<-eventChannel

			// Trigger the debouncer with the function you want to execute
			debounce(func() {
				// Your code to be executed after the debounce delay
				println("Event debounced")
			})
		}
	}()

	// Simulate events by sending to the channel
	for i := 0; i < 10; i++ {
		fmt.Printf("event %d at time %v\n", i, time.Now())
		// Simulate an event by sending to the channel
		eventChannel <- struct{}{}

		// Sleep for a short duration to simulate time passing between events
		time.Sleep(eventTriggerInterval)
	}

	// Wait for some time to allow debouncer to process the last event
	time.Sleep(5 * time.Second)
}


// The debounced function can be invoked with different functions, if needed,
// the last one will win.
func New(after time.Duration) func(f func()) {
	d := &debouncer{after: after}

	return func(f func()) {
		d.add(f)
	}
}

type debouncer struct {
	mu    sync.Mutex
	after time.Duration
	timer *time.Timer
}

func (d *debouncer) add(f func()) {
	d.mu.Lock()
	defer d.mu.Unlock()

	if d.timer != nil {
		d.timer.Stop()
	}

  // Create a new timer and start it
	d.timer = time.AfterFunc(d.after, f)
}

```


## Debouncing for fine-grained control over frequent events, achieving precise timing control

This code provides a more comprehensive implementation to handle frequent events with precise timing control. The triggers are collected in a channel, and the debouncing logic ensures that only the last trigger within a specified interval is processed.

```sh
-----|-----|-----|-----|-----|-----|-----|-----|-----=-----------|----- ----- -----=-----
              trigger           trigger           trigger                      trigger
```

```go
package main

import (
	"fmt"
	"runtime"
	"sync"
	"sync/atomic"
	"time"
)

const debounceInterval time.Duration = 3 * time.Second

func main() {
	// Set the time interval for the ticker
	interval := 1 * time.Second

	// Set the total duration to listen to the ticker
	totalDuration := 10 * time.Second

	// Create a ticker that ticks at the specified interval
	ticker := time.NewTicker(interval)
	defer ticker.Stop()

	// Create a channel to signal the end of the program after the specified duration
	done := time.After(totalDuration)

	manager := &ResourceManager{
		TriggerTimes: make(chan time.Time, 100),
		Done:         make(chan struct{}, 1),
	}

	go func() {

		manager.DebounceTrigger()

	}()

	// Start a loop to listen to ticker events
	for {
		select {
		case <-ticker.C:
			// Do something on each tick
			fmt.Println("Tick")
			manager.TriggerTimes <- time.Now()
		case <-done:
			// Break out of the loop after the specified duration
			fmt.Println("Time's up!")
			manager.Done <- struct{}{}
			numGoroutines := runtime.NumGoroutine()
			fmt.Println(numGoroutines)
			return
		}

	}

}

type ResourceManager struct {
	mutex sync.Mutex
	// TriggerTimes define a channel that collects the receiving time of each message from the callback channel.
	TriggerTimes chan time.Time
	Done         chan struct{}
}


func (m *ResourceManager) DebounceTrigger() {
	expectedTime := time.Now()
	ignoredAtomic := atomic.Bool{}

	for t := range m.TriggerTimes {
		if t.After(expectedTime) {
			expectedTime = time.Now().Add(debounceInterval)
			fmt.Println("update expectedTime")
			// insert a new event to m.TriggerTimes after debounceInterval.
			go func() {
				// since AfterFunc is blocking, need to run in another thread.
				time.AfterFunc(debounceInterval, func() {
					// load the ignored boolean and reset it.
					if ignored := ignoredAtomic.Swap(false); ignored {
						fmt.Printf("do process with time %v \n", t)
					}
				})
			}()

			fmt.Printf("do process with time %v \n", t)
		} else {
			ignoredAtomic.Store(true)
		}
	}
}

```





