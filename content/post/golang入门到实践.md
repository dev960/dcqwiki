---
title: "Golang入门到实践"
date: 2023-09-16T21:50:50+08:00
draft: false
tags: ["golang", "编程"]
categories: ["笔记"]
author: "dcq"

contentCopyright: '[Creative Commons Attribution-ShareAlike License](https://en.wikipedia.org/wiki/Wikipedia:Text_of_Creative_Commons_Attribution-ShareAlike_3.0_Unported_License)'
---

### golang入门

**基本语法**

1. 过一遍《go语言圣经》

2. 撸一遍Go By Example

**B站视频**

1. 刘丹冰大佬的视频[8小时转职Golang工程师(如果你想低成本学习Go语言)](https://www.bilibili.com/video/BV1gf4y1r79E)

2. 项目实战，跟大佬一步一步构建zinx网络框架

### 项目实战

1. 主要实现TCP/UDP通信；

2. 将UDP改造成可靠的数据传输通道；

3. 利用RS编解码对字节流进行冗余编码；

4. 整合MYSQL+Redis；

5. 实现大文件零拷贝、可靠传输；

### 对比java

1. 内存占用减少，java打包后80M，golang可执行文件大概7M;

2. golang协程并发比java线程轻量；

3. channel实现线程之间并发的协作；

4. slice不会new底层数组，只是在原数组上计算偏移，对比java new byte []明显降低了内存损耗；

5. io.copy零拷贝，速度更快；

6. 代码量更少；

### zinx协议TLV格式编解码

```go
package znet

import (
	"bytes"
	"encoding/binary"
	"errors"
	"zinx/utils"
	"zinx/ziface"
)

type DataPack struct{}

func NewDataPack() *DataPack {
	return &DataPack{}
}

func (dp *DataPack) GetDataLen() uint32 {
	//DataLen uint32:4bit, ID uint32 4bit
	return 8
}

func (dp *DataPack) Pack(msg ziface.IMessage) ([]byte, error) {
	//创建一个缓存
	dataBuf := bytes.NewBuffer([]byte{})
	//将dataLen 写入databuf
	if err := binary.Write(dataBuf, binary.LittleEndian, msg.GetMsgLen()); err != nil {
		return nil, err
	}
	//将MsgId 写入databuf
	if err := binary.Write(dataBuf, binary.LittleEndian, msg.GetMsgId()); err != nil {
		return nil, err
	}
	//将data 写入databuf
	if err := binary.Write(dataBuf, binary.LittleEndian, msg.GetData()); err != nil {
		return nil, err
	}
	return dataBuf.Bytes(), nil
}

func (dp *DataPack) Unpack(data []byte) (ziface.IMessage, error) {
	//创建一个缓存
	dataBuf := bytes.NewBuffer(data)

	//只解压head信息，得到dataLen和msgId
	msg := &Message{}

	//读dataLen
	if err := binary.Read(dataBuf, binary.LittleEndian, &msg.DataLen); err != nil {
		return nil, err
	}

	//读msgId
	if err := binary.Read(dataBuf, binary.LittleEndian, &msg.Id); err != nil {
		return nil, err
	}
	//判断datalen是否已经超出我们允许的最大包长度
	if utils.GlobalObject.MaxPackageSize > 0 && msg.DataLen > utils.GlobalObject.MaxPackageSize {
		return nil, errors.New("Too large msg data recv")
	}

	return msg, nil

}

```
