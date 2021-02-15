# Go

**Google Go Style Guide**

[golang/go](https://github.com/golang/go/wiki/CodeReviewComments)

**Uber Go Style Guide**

[uber-go/guide](https://github.com/uber-go/guide/blob/master/style.md)

# The Zen of Go

**Each package fulfils a single purpose**

A well designed Go package provides a single idea, a set of related behaviours. A good Go package starts by choosing a good name. Think of your package’s name as an elevator pitch to describe what it provides, using just one word.

**Handle errors explicitly**

Robust programs are composed from pieces that handle the failure cases before they pat themselves on the back. The verbosity of `if err != nil { return err }` is outweighed by the value of deliberately handling each failure condition at the point at which they occur. Panic and recover are not exceptions, they aren’t intended to be used that way.

**Return early rather than nesting deeply**

Every time you indent you add another precondition to the programmer’s stack consuming one of the 7 ±2 slots in their short term memory. Avoid control flow that requires deep indentation. Rather than nesting deeply, keep the success path to the left using guard clauses.

**Leave concurrency to the caller**

Let the caller choose if they want to run your library or function asynchronously, don’t force it on them. If your library uses concurrency it should do so transparently.

**Before you launch a goroutine, know when it will stop**

Goroutines own resources; locks, variables, memory, etc. The sure fire way to free those resources is to stop the owning goroutine.

**Avoid package level state**

Seek to be explicit, reduce coupling, and spooky action at a distance by providing the dependencies a type needs as fields on that type rather than using package variables.

**Simplicity matters**

Simplicity is not a synonym for unsophisticated. Simple doesn’t mean crude, it means *readable* and *maintainable*. When it is possible to choose, defer to the simpler solution.

**Write tests to lock in the behaviour of your package’s API**

Test first or test later, if you shoot for 100% test coverage or are happy with less, regardless your package’s API is your contract with its users. Tests are the guarantees that those contracts are written in. Make sure you test for the behaviour that users can observe and rely on.

**If you think it’s slow, first prove it with a benchmark**

So many crimes against maintainability are committed in the name of performance. Optimisation tears down abstractions, exposes internals, and couples tightly. If you’re choosing to shoulder that cost, ensure it is done for good reason.

**Moderation is a virtue**

Use goroutines, channels, locks, interfaces, embedding, in moderation.

**Maintainability counts**

Clarity, readability, simplicity, are all aspects of maintainability. Can the thing you worked hard to build be maintained after you’re gone? What can you do today to make it easier for those that come after you?

# Go学习涉及的知识点

[Go 学习涉及的知识点](Go%206b78115568114ab4beb4f9ed731cbcc2/Go%20%E5%AD%A6%E4%B9%A0%E6%B6%89%E5%8F%8A%E7%9A%84%E7%9F%A5%E8%AF%86%E7%82%B9%203ea3d05c1fa5477ea97e6665b1710a88.md)

# Go并发编程

![Go%206b78115568114ab4beb4f9ed731cbcc2/Untitled.png](Go%206b78115568114ab4beb4f9ed731cbcc2/Untitled.png)

## 基本并发原语

- Mutex、RWMutex、Waitgroup、Cond、Pool、Context等标准库中的并发原语

### Mutex

**使用场景**

- 计数器
- 同时更新用户的账户信息
- 秒杀系统
- 往同一个buffer中并发写入数据

**互斥锁实现机制**

概念临界区: 在并发编程中，如果程序中的一部分会被并发访问或修改，那么，为了避免并发访问导致的异常结果，这部分程序需要被保护起来

**互斥锁限定临界区只能同时由一个线程持有**

![Go%206b78115568114ab4beb4f9ed731cbcc2/Untitled%201.png](Go%206b78115568114ab4beb4f9ed731cbcc2/Untitled%201.png)

**Mutex的基本使用方法**

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	var mu sync.Mutex
	var count = 0
	var wg sync.WaitGroup
	wg.Add(10)
	for i := 0; i < 10; i++ {
		go func() {
			defer wg.Done()
			for j := 0; j < 100000; j++ {
				mu.Lock()
				count++
				mu.Unlock()
			}
		}()
	}
	wg.Wait()
	fmt.Println(count)
}
```

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
  var mu sync.Mutex
	var count = 0
	var wg sync.WaitGroup
	wg.Add(10)
	for i := 0; i < 10; i++ {
		go func() {
			defer wg.Done()
			for j := 0; j < 100000; j++ {
				mu.Lock()
				count++
				mu.Unlock()
			}
		}()
	}
	wg.Wait()
	fmt.Println(count)
}
```

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
    var counter Counter
	  var wg sync.WaitGroup
	  wg.Add(10)
	  for i := 0; i < 10; i++ {
        go func() {
            defer wg.Done()
            for j := 0; j < 100000; j++ {
				         counter.Lock()
				         counter.Count++
 				         counter.Unlock()
			       }
        }()
    }
    wg.Wait()
    fmt.Println(count)
}

type Counter struct {
    sync.Mutex
    Count uint64
}
```

```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    var counter Counter

    var wg sync.WaitGroup
    wg.Add(10)

    for i := 0; i < 10; i++ {
        go func() {
            defer wg.Done()
            for j := 0; j < 100000; j++ {
                counter.Incr()
            }
        }()
    }
    wg.Wait()
    fmt.Println(counter.Count())
}

type Counter struct {
    CounterType int
    Name        string

    mu    sync.Mutex
    count uint64
}

func (c *Counter) Incr() {
    c.mu.Lock()
    c.count++
    c.mu.Unlock()
}

func (c *Counter) Count() uint64 {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.count
}
```

```python

```

Mutex**实现原理**

### WaitGroup

- 主要用来做任务编排的一个并发原语；要解决的就是并发 - 等待的时间问题。

**基本语法**

```go
func (wg *WaitGroup) Add(delta int)
func (wg *WaitGroup) Done()
func (wg *WaitGroup) Wait()
```

- Add: 用来设置WaitGroup的计数值
- Done: 用来将WaitGroup的计数值减1，调用了Add(-1)
- Wait: 调用这个方法的goroutine会一直阻塞，知道WaitGroup的计数值为0

```go
package main
import (
    "fmt"
    "sync"
)

func main() {
    var counter Counter

    var wg sync.WaitGroup
    wg.Add(10)

    for i := 0; i < 10; i++ {
        go worker(&counter, &wg)
    }
    wg.Wait()
    fmt.Println(counter.Count())
}

type Counter struct {
    mu    sync.Mutex
    count uint64
}

func (c *Counter) Incr() {
    c.mu.Lock()
    c.count++
    c.mu.Unlock()
}

func (c *Counter) Count() uint64 {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.count
}

func worker(c *Counter, wg *sync.WaitGroup) {
    defer wg.Done()
    time.Sleep(1 * time.Second)
    c.Incr()
}
```

**实现**

数据结构

- noCopy的辅助字段: 辅助vet工具检查是否通过copy赋值这个WaitGroup实例
- state1: 一个具有符合意义的字段，包含WaitGroup的计数、阻塞在检查点的waiter数和信号量

**使用时常见错误**

1. 计数器设置为负值: 直接panic "sync: negative WaitGroup counter"
    - 调用Add时传递了一个负值
    - 调用Done方法的次数超过了WaitGroup的计数值

    **注意: 预先确定好WaitGroup的计数值，然后调用相同次数的Done**

    ```go
    package main

    import (
        "fmt"
        "sync"
    )

    // ex1
    func main() {
        var wg sync.WaitGroup
        wg.Add(10)
        wg.Add(-10)
        wg.Add(-1) // panic

        for i := 0; i < 10; i++ {
            go func() {
                fmt.Println(i)
            }()
            wg.Done()
        }
        wg.Wait()
        fmt.Println("done")
    }

    //ex2
    func main() {
        var wg sync.WaitGroup
        wg.Add(1)

        wg.Done()

        wg.Done() // panic
    }
        

    ```

2. Add的时机不对

    **注意: 等所有的Add方法调用之后再调用Wait**

    ```go
    package main

    import (
        "fmt"
        "sync"
        "time"
    )

    func main() {
        var wg sync.WaitGroup
        // wg.Add(4) 行A
        go do(100, &wg)
        go do(110, &wg)
        go do(120, &wg)
        go do(130, &wg)

        wg.Wait()
        fmt.Println("Done")
    }

    func do(millisecs time.Duration, wg *sync.WaitGroup) {
        duration := millisecs * time.Millisecond
        time.Sleep(duration)

        wg.Add(1) // 行B
        fmt.Println("exec, duration", duration)
        wg.Done()

        // change method 1: 注释行B 添加行A

        // change method 2
        /* 启动子goroutine之前调用Add
        wg.Add(1)
        
        go func() {
            duration := millisecs * time.Millisecond
            time.Sleep(duration)
            fmt.Println("exec, duration", duration)
            wg.Done()
        */
    }
    ```

3. 前一个Wait还没有结束就重用WaitGroup

    **注意: WaitGroup是可以重用的，前提为必须等上一轮的Wait完成之后才能重用WaitGroup执行下一轮的Add/Wait**

    ```go
    func main() {
        var wg sync.WaitGroup
        wg.Add(1)
        go func() {
            time.Sleep(time.Second)
            wg.Done()
            wg.Add(1) // 重用
        }()
        wg.Wait() // 主goroutine等待，有可能和上面的并发执行 出现panic
    }
    ```

**noCopy: 辅助vet检查**

noCopy是一个通用的计数技术，其他并发原语中也会用到

```go
type noCopy struct{}

// Lock is a no-op used by -copylocks checker from `go vet`.
func (*noCopy) Lock()   {}
func (*noCopy) Unlock() {}
```

**避免错误使用WaitGroup的注意事项**

1. 不重用 WaitGroup
2. 保证所有的 Add 方法调用都在 Wait 之前
3. 不传递负数给 Add 方法，只通过 Done 来减值
4. 不做多余的 Done 方法调用，保证 Add == Done
5. 不遗漏 Done 方法的调用，否则导致 Wait hang 住无法返回

## 原子操作

## Channel

## 扩展并发原语

- 信号量、SingleFlight、循环栅栏、ErrGroup等

## 分布式并发原语

- ETCD为例: Leader选举、分布式互斥锁、分布式读写锁、分布式队列等

- 学习主线

# Goroutine

# Channel

# 面试

[面试整理](Go%206b78115568114ab4beb4f9ed731cbcc2/%E9%9D%A2%E8%AF%95%E6%95%B4%E7%90%86%207b841739a19445ea9d5dea5f12befc54.md)