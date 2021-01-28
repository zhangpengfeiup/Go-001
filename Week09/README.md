学习笔记

用 Go 实现一个 tcp server ，用两个 goroutine 读写 conn，两个 goroutine 通过 chan 可以传递 message，能够正确退出
1. 什么是 goroutine
A goroutine is a lightweight thread of execution
2. 什么是 chan
Channels are the pipes that connect concurrent goroutines.You can send values into channels from one goroutine and receive those values into another goroutine.
3. 什么是 tcp server
提供tcp服务连接服务的服务器
自己不会写，所以就参考了github 名为 ：cty898 同学的作业，也是自己抄的作业，诶。。。

```
package main
import (
   "bufio"
   "fmt"
   "log"
   "net"
   "strconv"
   "sync"
   "time"
)
// 1. 用Go实现一个 tcp server,用两个 goroutine 读写conn,两个 goroutine 通过 chan 可以传递message,能够正确退出
// 生成用户ID
var (
  globalID int
  idLocker sync.Mutex
)
func GenUserId() int {
   idLocker.Lock()
   defer idLocker.Unlock()
   globalID++
   return globalID
}
type User struct {
   ID int
   Addr string
   EnterAt time.Time
   MessageChannel chan string
}
func sendMessage(conn net.Conn,ch <-chan string) {
    for msg := range ch {
    fmt.Fprintln(conn,msg)
    }
}
func handleConn(conn net.Conn) {
   defer conn.Close()
   user := &User{
    ID: GenUserId(),
    Addr: conn.RemoteAddr().String(),
        EnterAt: time.Now(),
        MessageChannel: make(chan string, 8),
   }
   // 启动写 conn的协程
   go sendMessage(conn, user.MessageChannel)
   input := bufio.NewScanner(conn)
   for input.Scan() {
    fmt.Println(strconv.Itoa(user.ID) + ":" + input.Text())
    // 通过chan可以传递 message,客户端发来什么消息就回什么消息
    user.MessageChannel <- input.Text()
   }
   if err := input.Err(); err != nil {
    log.Println("读取错误:", err)
    }
}
func main() {
    listener,err := net.Listen("tcp", ":88")
    if err != nil {
    panic(err)
    }
    for {
    conn,err := listener.Accept()
    if err != nil {
        log.Println(err)
        continue
    }
    // 启动读conn的协程
    go handleConn(conn)
    }
}
```
