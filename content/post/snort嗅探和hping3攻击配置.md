---
title: "snort嗅探和hping3攻击配置"
date: 2023-09-14T22:37:56+08:00
lastmod: 2023-09-14T22:37:56+08:00
draft: false
tags: ["snort", "hping3", "嗅探"]
categories: ["笔记"]
author: "dcq"

contentCopyright: '<a rel="license noopener" href="https://en.wikipedia.org/wiki/Wikipedia:Text_of_Creative_Commons_Attribution-ShareAlike_3.0_Unported_License" target="_blank">Creative Commons Attribution-ShareAlike License</a>'

---

```bash
   $sudo apt install snort
   $sudo vim /etc/snort/ruls/local.rule
   $sudo sudo snort -c /etc/snort/snort.conf -i enp1s0 -A unsock -l /tmp/
   -c 指定配置文件
   -i 指定网卡
   -A 指定报警类型
   -l 指定socket unix路径，/tmp/snort_adlert
```

1. snort.conf-> local.rule
   
   ```sh
      alert icmp any any -> $HOME_NET any (msg:"ICMP flood"; sid:1000001; rev:1; classtype:icmp-event; detection_filter:track by_dst, count 10, seconds 3;)
   
       alert tcp any any -> $HOME_NET any (flags: S; msg:"DoS Attack    Type : SYNflood"; flow:stateless; sid:3; detection_filter:track by_dst, count 10, seconds 3;)
   
       alert udp $EXTERNAL_NET any -> $HOME_NET any (msg:"DDOS shaft agecondent to handler, Type: Udp flood"; content:"alive"; reference:arachnids,256; classtype:attempted-dos; sid:240; rev:2;)
   ```

2. hping3
   
   ```sh
   #sys flood
   sudo hping3 -S --flood -V -p 3306 192.168.171.129
   sudo hping3 -S --flood -c 1 -p 3306 192.168.171.129
   #icmp
   sudo hping3 -1 --flood  192.168.171.129
   #udp
   sudo hping3 -2 --flood  192.168.171.129
   ```

3. socket接口
   
   ```python
   import os
   import sys
   import time
   import socket
   import logging
   import array
   import requests
   import json
   import random
   ```

from ryu.lib import alert
from ryu.lib.packet import packet
from ryu.lib.packet import ethernet
from ryu.lib.packet import ipv4
from ryu.lib.packet import icmp

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# 注意和snort指定的unixsocket通信的路径是一样的

SOCKFILE = "/tmp/snort_alert"
BUFSIZE = 65863

# 控制器IP

CONTROLLER_IP = '127.0.0.1'

# 控制器RESTful API访问的端口以及URL

CONTROLLER_PORT="8080"
CONTROLLER_URL="http://127.0.0.1:8080/admin/sys/snort/v1/api"

# 控制器socket通信的端口

CONTROLLER_PORT1 = 51234
user_agents = ['Mozilla/5.0 (Windows NT 6.1; rv:2.0.1) Gecko/20100101 Firefox/4.0.1',
                   'Mozilla/5.0 (Windows; U; Windows NT 6.1; en-us) AppleWebKit/534.50 (KHTML, like Gecko) Version/5.1 Safari/534.50',
                   'Opera/9.80 (Windows NT 6.1; U; en) Presto/2.8.131 Version/11.11']
headers = {'User-Agent': random.choice(user_agents), 'Content-type': 'application/json'}

class SnortListener():

    def __init__(self):
        self.unsock = None
        self.nwsock = None
    def packet_print(self, pkt,alert):
        s = ''
        pkt = packet.Packet(array.array('B', pkt))
        eth = pkt.get_protocol(ethernet.ethernet)
        _ipv4 = pkt.get_protocol(ipv4.ipv4)
        _icmp = pkt.get_protocol(icmp.icmp)
        _tcp = pkt.get_protocol(ipv4.tcp.tcp)
        _udp = pkt.get_protocol(ipv4.udp.udp)
        if _tcp:
            s = "%r" % _tcp
        if _udp:
            s += "%r" % _udp
        if _icmp:
            s += "%r" % _icmp
        if _ipv4:
            s += "%r" % _ipv4
        if eth:
            s += "%r" % eth
        s={'alert': alert, 'src-ip': _ipv4.src, 'dst-ip': _ipv4.dst, 'src-mac': eth.src, 'dst-mac': eth.dst}
        return s
    
    def start_send(self):
        '''Open a client on Network Socket'''
        self.nwsock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        try:
            self.nwsock.connect((CONTROLLER_IP, CONTROLLER_PORT1))
        except Exception as e:
            print(CONTROLLER_PORT1)
            logger.info("Network socket connection error: %s" % e)
            sys.exit(1)
    
    def start_recv(self):
        if os.path.exists(SOCKFILE):
            os.unlink(SOCKFILE)
    
        self.unsock = socket.socket(socket.AF_UNIX, socket.SOCK_DGRAM)
        self.unsock.bind(SOCKFILE)
        logger.info("Unix Domain Socket listening...")
        self.recv_loop()
    
    def recv_loop(self):
    
        #self.start_send()#establish network socket
        while True:
            data = self.unsock.recv(BUFSIZE)
            time.sleep(0.5)
            if data:
                logger.debug("Send {0} bytes of data.".format
                             (sys.getsizeof(data)))
                self.tcp_send(data)
            else:
                pass
    
    def send_json(self,data):
        s = json.dumps(data)
        print(s)
        #r = requests.post(CONTROLLER_URL, headers=headers, data=s)
        #print(r.status_code)
    
    def tcp_send(self, data):
        data2 = data[:BUFSIZE]
        msg = alert.AlertPkt.parser(data2)
        s1 = '%s' % b''.join(msg.alertmsg)
        s2 = self.packet_print(msg.pkt, s1)
        print("json:", json.dumps(s2))
        self.send_json(s2)# 用RESTful API发送警告
    #self.nwsock.sendall(json.dumps(s2)+'\n') # 用socket发送警告
        logger.info("Send the alert messages to floodlight.")

if __name__ == '__main__':
    server = SnortListener()
    server.start_recv()