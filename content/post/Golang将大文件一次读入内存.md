---

title: "Golang大文件发送提速"
date: 2023-09-10T01:37:56+20:00
lastmod: 2023-09-10T01:37:56+21:00
draft: false
tags: ["Golang", "大文件处理"]
categories: ["笔记"]
author: "dcq"

contentCopyright: '<a rel="license noopener" href="https://en.wikipedia.org/wiki/Wikipedia:Text_of_Creative_Commons_Attribution-ShareAlike_3.0_Unported_License" target="_blank">Creative Commons Attribution-ShareAlike License</a>'

---

# Golang将大文件一次读入内存

> 利用Slice切片，比每次读取一小块导致频繁Gc

```go
package main

import (
    "bufio"
    "container/list"
    "fmt"
    "io"
    "os"
    "sync"
    "time"
)

func main(){
    start := time.Now()
    content, err := ioutil.ReadFile("d:\\software\\cn_windows_10_consumer_editions_version_21h1_x64_dvd_1baf479d.iso")
    if err != nil {
        fmt.Println("file read fail")
    }
    fmt.Printf("read file len :%d, time cost :%v slices: %d\n", len(content), time.Since(start), len(content)/(8964*20))
    step := 8964 * 20
    fmt.Println(len(content)%step, step)

    for i := 0; i < len(content); i += step {
        if i+step > len(content) {
            slice := make([]byte, len(content)%step)
            copy(slice, content[i:i+len(slice)])
            fmt.Printf("slice addr :%p slice len: %d\n", slice, len(slice))
        } else {
            slice := make([]byte, step)
            copy(slice, content[i:i+step])
            fmt.Printf("slice addr :%p slice len: %d\n", slice, len(slice))
        }
   }
}
```

💡 Tips: 一次将文件读入内存，用slice来遍历文件，slice本身只是引用，不会复制byte array，所以很快，而且不占用内存，减少IO,GC，提速到9000mbps
