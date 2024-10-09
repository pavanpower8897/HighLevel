
```
type ConnectionPool struct {
}

type Connection interface {
	GetConnectionWithTImeout()
}

type RedisConnectionPool struct {
	AvailableConnections []int
	Mu                   *sync.Mutex
	Timeout              time.Duration
	ReleaseChan          chan int
}

func (rc *RedisConnectionPool) GetConnectionWithTImeout(i int) (int, error) {
	var availableConn int
	for {
		timeout := time.After(rc.Timeout)
		rc.Mu.Lock()
		var connFound bool
		if len(rc.AvailableConnections) > 0 {
			availableConn = rc.AvailableConnections[0]
			rc.AvailableConnections = rc.AvailableConnections[1:]
			connFound = true
		}
		rc.Mu.Unlock()

		if connFound {
			return availableConn, nil
		}

		for {
			var released bool
			select {
			case <-rc.ReleaseChan:
				released = true
			case <-timeout:
				return availableConn, errors.New("timedout")
			}

			if released {
				break
			}
		}
	}
}

func (rc *RedisConnectionPool) ReleaseConnection(i int, conn int) {
	rc.Mu.Lock()
	fmt.Printf("go routin %d released conn: %d \n", i, conn)
	rc.AvailableConnections = append(rc.AvailableConnections, conn)
	rc.ReleaseChan <- 1
	rc.Mu.Unlock()
}

func acquireRedisConnections() int {
	rc := RedisConnectionPool{
		AvailableConnections: []int{1, 2, 3, 4, 5},
		Mu:                   new(sync.Mutex),
		Timeout:              5 * time.Second,
		ReleaseChan:          make(chan int, 10),
	}

	var wg sync.WaitGroup
	for i := 0; i < 10; i++ {
		wg.Add(1)
		go func() {
			defer func() {
				wg.Done()
			}()

			conn, err := rc.GetConnectionWithTImeout(i)
			if err != nil {
				fmt.Printf("go routin %d connection timedout found \n", i)
				return
			}

			fmt.Printf("go routin %d with conn: %d \n", i, conn)
			time.Sleep(1 * time.Second)
			fmt.Printf("go routin time lapsed %d with conn: %d \n", i, conn)
			rc.ReleaseConnection(i, conn)
		}()
	}

	wg.Wait()
	close(rc.ReleaseChan)
	fmt.Println("------done------")
	return 0
}

func main() {
	x := acquireRedisConnections()
	fmt.Println(x)
	return
```


Above one is practised one, Below is efficient one shared by gpt

```
package main

import (
	"fmt"
	"time"
)

type RedisConnectionPool struct {
	connections chan int
	timeout     time.Duration
}

func NewRedisConnectionPool(size int, timeout time.Duration) *RedisConnectionPool {
	pool := &RedisConnectionPool{
		connections: make(chan int, size), // Buffered channel
		timeout:     timeout,
	}
	// Initialize connections
	for i := 1; i <= size; i++ {
		pool.connections <- i
	}
	return pool
}

// GetConnectionWithTimeout tries to acquire a connection with a timeout
func (rc *RedisConnectionPool) GetConnectionWithTimeout(i int) (int, error) {
	select {
	case conn := <-rc.connections:
		return conn, nil // Acquired connection
	case <-time.After(rc.timeout):
		return 0, fmt.Errorf("goroutine %d: timed out", i) // Timeout exceeded
	}
}

func main() {
	rc := NewRedisConnectionPool(2, 2*time.Second) // Pool with 2 connections

	for i := 0; i < 5; i++ {
		go func(i int) {
			conn, err := rc.GetConnectionWithTimeout(i)
			if err != nil {
				fmt.Println(err)
				return
			}
			fmt.Printf("Goroutine %d acquired connection %d\n", i, conn)
			time.Sleep(1 * time.Second) // Simulate work
			rc.connections <- conn // Release the connection
			fmt.Printf("Goroutine %d released connection %d\n", i, conn)
		}(i)
	}

	time.Sleep(5 * time.Second) // Wait for all goroutines to finish
}

```
