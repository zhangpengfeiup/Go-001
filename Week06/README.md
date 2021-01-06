了解了Hystrix 熔断器的原理


![图片](http://tech.ipalfish.com/blog/images/dolphin/circuit_breaker_fsm.png)


滑动窗口内桶的个数=滑动窗口时长/bucket时长

bucket时长 = 滑动窗口时长/桶个数

Hystrix 的滑动窗口计数我目前写不出来，参考网上了别人实现的滑动串口计数，使用了Go的ring (环形队列实现的滑动窗口)

地址：https://studygolang.com/articles/26631

```
package main
import (
    "fmt"
    "net"
    "os"
    "container/ring"
    "sync/atomic"
    "sync"
    "time"
)
var (
    limitCount int = 10
    limitBucket int = 6
    curCount int32 = 0
    head *ring.Ring
)
func main() {
    tcpAddr,err := net.ResolveTCPAddr("tcp4", "0.0.0.0:9090")
    checkError(err)
    listener,err := net.ListenTCP("tcp", tcpAddr)
    checkError(err)
    defer listener.Close()
    
    // 初始化滑动窗口
    head = ring.New(limitBucket)
    for i := 0;i < limitBucket;i++ {
        head.Value = 0
        head = head.Next()
    }
    
    // 启动执行器
    go func() {
        timer := time.NewTicker(time.Second * 1)
        for range timer.C {
            subCount := int32(0 - head.Value.(int))
            newCount := atomic.AddInt32(&curCount,subCount)
            
            arr := [6]int{}
            for i := 0;i < limitBucket;i++ {
                arr[i] = head.Value.(int)
                head = head.Next()
            }
            
            fmt.Println("move subCount,newCount,arr", subCount, newCount,arr)
            head.Value = 0
            head = head.Next()
        }
    }()
    
    
    for {
        conn,err := listener.Accept()
        if err != nil {
            fmt.Println(err)
            continue
        }
        go handle(&conn)
    }
}
func handle(conn *net.Conn) {
    defer (*conn).Close()
    n := atomic.AddInt32(&curCount, 1)
    
    if n > int32(limitCount) {
        atomic.AddInt32(&curCount, -1)
        (*conn).Write([]byte("HTTP/1.1 404 NOT FOUND\r\n\r\nError, too many request, please try again."))
    }else {
        mu := sync.Mutex{}
        mu.Lock()
        pos := head.Prev()
        val := pos.Value.(int)
        val++
        pos.Value = val
        mu.Unlock()
        time.Sleep(1 * time.Second)
        (*conn).Write([]byte("HTTP/1.1 200 OK\r\n\r\nI can change the world!"))
    }
}
func checkError(err error) {
    if err != nil {
        fmt.Fprintf(os.Stderr, "Fatal error:%s", err.Error())
        os.Exit(1)
    }
}
```
