
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
