---
title: "Golang缓存读写大文件"
date: 2023-09-27T22:37:49+08:00
draft: false
tags: ["golang","bufio"]
categories: ["笔记"]
author: "dcq"

contentCopyright: '[Creative Commons Attribution-ShareAlike License](https://en.wikipedia.org/wiki/Wikipedia:Text_of_Creative_Commons_Attribution-ShareAlike_3.0_Unported_License)'
---

# Bufio读写大文件

1. 解决大文件读写缓慢的问题，源码：
   
   ```go
   func NewReader(rd io.Reader) *Reader {
       return NewReaderSize(rd, defaultBufSize)
   }
   
   func NewReaderSize(rd io.Reader, size int) *Reader {
       b, ok := rd.(*Reader)
       if ok && len(b.buf) >= size {
           return b
       }
       if size < minReadBufferSize {
           size = minReadBufferSize
       }
       r := new(Reader)
       r.reset(make([]byte, size), rd)
       return r
   }
   ```

2. bufio.NewWriterSize(file, size)
   
   **写入流程**
   
   * 判断buf中可用容量是否可以放下 p
   
   * 如果能放下，直接把p拼接到buf后面，即把内容放到缓冲区；
   
   * 如果缓冲区的可用容量不足以放下，且此时缓冲区是空的，直接把p写入文件即可
   
   * 如果缓冲区的可用容量不足以放下，且此时缓冲区有内容，则用p把缓冲区填满，把缓冲区所有内容写入文件，并清空缓冲区
   
   * 判断p的剩余内容大小能否放到缓冲区，如果能放下（此时和步骤1情况一样）则把内容放到缓冲区
   
   * 如果p的剩余内容依旧大于缓冲区，（注意此时缓冲区是空的，情况和步骤3一样）则把p的剩余内容直接写入文件；
   
   **读取流程**
   
   * 当读取内容小于缓冲区空间时，从缓冲区读取；
   
   * 当缓存区有内容的时，将缓存区内容全部填入p并清空缓存区；
   
   * 当缓存区没有内容的时候且len(p)>len(buf),即要读取的内容比缓存区还要大，直接去文件读取即可；
   
   * 当缓存区没有内容的时候且len(p)<len(buf),即要读取的内容比缓存区小，缓存区从文件读取内容充满缓存区，并将p填满（此时缓存区有剩余内容）；
   
   * 以后再次读取时缓存区有内容，将缓存区内容全部填入p并清空缓存区（此时和情况1一样）；

```go
package main

import (
    "bufio"
    "encoding/binary"
    "fmt"
    "os"
)

type MyStruct struct {
    Field1 uint32
    Field2 uint32
}

func main() {
    // Writing binary data
    file, err := os.Create("example.bin")
    if err != nil {
        panic(err)
    }
    defer file.Close()

    writer := bufio.NewWriter(file)
    data := MyStruct{123, 456}
    err = binary.Write(writer, binary.LittleEndian, data)
    if err != nil {
        panic(err)
    }
    writer.Flush()

   // Reading binary data
    file, err = os.Open("example.bin")
    if err != nil {
        panic(err)
    }
    defer file.Close()

    reader := bufio.NewReader(file)
    var data2 MyStruct
    err = binary.Read(reader, binary.LittleEndian, &data2)
    if err != nil {
        panic(err)
    }
    fmt.Printf("Field1: %d, Field2: %d\n", data2.Field1, data2.Field2)
} writer.Flush()
}
```
