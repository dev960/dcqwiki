---
title: "合宙Esp32+Micropython温湿度传感器"
date: 2023-09-10T01:37:56+08:00
lastmod: 2023-09-10T01:37:56+08:00
draft: false
tags: ["Esp32", "笔记", "MicroPython","湿度","心率"]
categories: ["硬件"]
author: "dcq"

contentCopyright: '<a rel="license noopener" href="https://en.wikipedia.org/wiki/Wikipedia:Text_of_Creative_Commons_Attribution-ShareAlike_3.0_Unported_License" target="_blank">Creative Commons Attribution-ShareAlike License</a>'

---

# 价格：9.9包邮

![](https://gitee.com/spring3th/pubpic/raw/master/img/esp32c3.png)

## 温湿度传感器

💡 Tips：GND：接地，VCC:3.3v, DAT:pin4。

- 支持温度，湿度。
- 支持定时刷新温湿度和时间。

## OLED显示屏

💡 Tips：GND：接地， VCC:5v, SCL: pin5, SDA: pin4

- 支持中英文。
- 支持定时刷新数据。

## 超声波HCSR04传感器

💡 Tips：GND:接地， VCC:5v, TRI: pin12, CHO: pin13。

- 支持最大400mm

## MAX10302心率血氧传感器

💡 Tips：GND:接地， VCC:5v, SCL:pin9, SDA:pin8

- 支持血氧，心率，温度

![](https://gitee.com/spring3th/pubpic/raw/master/img/心率.jpg)

```python
# -*- coding: utf-8 -*-
from machine import Pin,Timer,RTC
from ssd1306_lib import SSD1306
import gc
import math
import font
import dht11
import utime
#import mqtt
import utils
import time
from HCSR04 import HCSR04
from mqtt import mqttClient
from wifi_manager import WifiManager

hc = HCSR04(trigger_pin=12, echo_pin=13,echo_timeout_us=10000)

print("distance:",hc.distance_cm())


rtc = RTC()

wm = WifiManager("TP-LINK_8132","123456789")
wm.connect()

utils.sync_ntp()
print(rtc.datetime())
print("同步后本地时间：%s" %str(time.localtime()))

#mqtt.on_mqtt_connect()




count=0
def time_interrupt(timer):             #定时器中断服务函数
    global count
    count +=1
    gc.collect()
    temp = d.temperature()                #获取温度
    humi = d.humidity()                   #获取湿度
    #print("temp: "+str(temp) + "\thumidity: " + str(humi))
    #mqtt.on_publish("esp32/temp","temp: "+str(temp) + "\thumidity: " + str(humi))
    display.draw_chinese('温度',1,2)        #显示中文
    display.draw_chinese('湿度',1,4)

    display.draw_string(':'+str(temp),6,2)  #显示温度
    display.draw_string(':'+str(humi),6,4)  #显示湿度
    #display.draw_text(40,56,'Loop:'+str(count),)
    #display.draw_text(40, 56, "{:02}:{:02}:{:02}".format(time.localtime()[3], time.localtime()[4], time.localtime()[5]))
    display.draw_text(0,56, "{:02}-{:02}-{:02}({:1}){:02}:{:02}:{:02}".format(time.localtime()[0], time.localtime()[1], time.localtime()[2], time.localtime()[6]+1, time.localtime()[3], time.localtime()[4], time.localtime()[5]))
    display.display()

def rtc_time(timer):
    display.draw_text(0,56, "{:02}-{:02}-{:02}({:1}){:02}:{:02}:{:02}".format(time.localtime()[0], time.localtime()[1], time.localtime()[2], time.localtime()[6]+1, time.localtime()[3], time.localtime()[4], time.localtime()[5]))
    display.display()



if __name__ == '__main__':
    d = dht11.DHT11(Pin(4))        #dht11引脚
    d.measure()
    tim0 = Timer(0)                 #创建定时器
    #开启定时器中断，每隔1秒触发一次
    tim0.init(period=1000, mode=Timer.PERIODIC, callback=time_interrupt)

    display = SSD1306()             #OLED
    display.init_display()          #初始化
    display.draw_text(16,8,'ESP32 Micropython',)
    display.display()
    count=0
```

```python
from machine import Pin,I2C
import font

# Constants
DISPLAYOFF          = 0xAE
SETCONTRAST         = 0x81
DISPLAYALLON_RESUME = 0xA4
DISPLAYALLON        = 0xA5
NORMALDISPLAY       = 0xA6
INVERTDISPLAY       = 0xA7
DISPLAYON           = 0xAF
SETDISPLAYOFFSET    = 0xD3
SETCOMPINS          = 0xDA
SETVCOMDETECT       = 0xDB
SETDISPLAYCLOCKDIV  = 0xD5
SETPRECHARGE        = 0xD9
SETMULTIPLEX        = 0xA8
SETLOWCOLUMN        = 0x00
SETHIGHCOLUMN       = 0x10
SETSTARTLINE        = 0x40
MEMORYMODE          = 0x20
COLUMNADDR          = 0x21
PAGEADDR            = 0x22
COMSCANINC          = 0xC0
COMSCANDEC          = 0xC8
SEGREMAP            = 0xA0
CHARGEPUMP          = 0x8D
EXTERNALVCC         = 0x10
SWITCHCAPVCC        = 0x20
SETPAGEADDR         = 0xB0
SETCOLADDR_LOW      = 0x00
SETCOLADDR_HIGH     = 0x10
ACTIVATE_SCROLL                      = 0x2F
DEACTIVATE_SCROLL                    = 0x2E
SET_VERTICAL_SCROLL_AREA             = 0xA3
RIGHT_HORIZONTAL_SCROLL              = 0x26
LEFT_HORIZONTAL_SCROLL               = 0x27
VERTICAL_AND_RIGHT_HORIZONTAL_SCROLL = 0x29
VERTICAL_AND_LEFT_HORIZONTAL_SCROLL  = 0x2A

# I2C devices are accessed through a Device ID. This is a 7-bit
# value but is sometimes expressed left-shifted by 1 as an 8-bit value.
# A pin on SSD1306 allows it to respond to ID 0x3C or 0x3D. The board
# I bought from ebay used a 0-ohm resistor to select between "0x78"
# (0x3c << 1) or "0x7a" (0x3d << 1). The default was set to "0x78"
DEVID = 0x3c

# I2C communication here is either <DEVID> <CTL_CMD> <command byte>
# or <DEVID> <CTL_DAT> <display buffer bytes> <> <> <> <>...
# These two values encode the Co (Continuation) bit as b7 and the
# D/C# (Data/Command Selection) bit as b6.
CTL_CMD = 0x80
CTL_DAT = 0x40

class SSD1306(object):

    def __init__(self, i2c_devid=DEVID):
        self.external_vcc = False
        self.height       = 64
        self.pages        = int(self.height / 8)
        self.columns      = 128

        self.buffer = bytearray(self.pages * self.columns)
        self.i2c = I2C(scl=Pin(5), sda=Pin(4), freq=100000)
        self.devid = i2c_devid
        self.offset = 1
        # I2C command buffer
        self.cbuffer = bytearray(2)
        self.cbuffer[0] = CTL_CMD

    def clear(self):
        self.buffer = bytearray(self.offset + self.pages * self.columns)
        if self.offset == 1:
            self.buffer[0] = CTL_DAT

    def write_command(self, command_byte):
        self.cbuffer[1] = command_byte
        self.i2c.writeto(self.devid, self.cbuffer)

    def invert_display(self, invert):
        self.write_command(INVERTDISPLAY if invert else NORMALDISPLAY)

    def display(self):
        self.write_command(COLUMNADDR)
        self.write_command(0)
        self.write_command(self.columns - 1)
        self.write_command(PAGEADDR)
        self.write_command(0)
        self.write_command(self.pages - 1)
        self.i2c.writeto(self.devid, self.buffer)


    def set_pixel(self, x, y, state):
        index = x + (int(y / 8) * self.columns)
        if state:
            self.buffer[self.offset + index] |= (1 << (y & 7))

        else:
            self.buffer[self.offset + index] &= ~(1 << (y & 7))

            def draw_chinese(self,ch_str,x_axis,y_axis):
            offset_=0
            y_axis=y_axis*8#中文高度一行占8个
            x_axis=127-(x_axis*16)#中文宽度占16个
            for k in ch_str:
            code = 0x00#将中文转成16进制编码
            data_code = k.encode("utf-8")
            code |= data_code[0]<<16
            code |= data_code[1]<<8
            code |= data_code[2]
            byte_data=font.byte2[code]
            for y in range(0,16):
            a_=bin(byte_data[y]).replace('0b','')
            while len(a_)<8:
            a_='0'+a_

            b_=bin(byte_data[y+16]).replace('0b','')
            while len(b_)<8:
            b_='0'+b_
            for x in range(0,8):
            self.set_pixel(x_axis-x-offset_,y+y_axis,int(a_[x]))#文字的上半部分
            self.set_pixel(x_axis-x-8-offset_,y+y_axis,int(b_[x]))#文字的下半部分
            offset_+=16

            def draw_string(self,ch_str,x_axis,y_axis):
            offset_=0
            y_axis=y_axis*8#字符高度一行占8个
            x_axis=127-(x_axis*8)#字符宽度占8个
            for k in ch_str:
            code = 0x00
            code = ord(k)-0x20  
            byte_data=font.byte3[code]  #字符查表
            for y in range(0,16):
            a_=bin(byte_data[y]).replace('0b','')
            while len(a_)<8:
            a_='0'+a_

            for x in range(0,8):
            self.set_pixel(x_axis-x-offset_,y+y_axis,int(a_[x]))
            offset_+=8

            def init_display(self):
            chargepump = 0x10 if self.external_vcc else 0x14
            precharge  = 0x22 if self.external_vcc else 0xf1
            multiplex  = 0x1f if self.height == 32 else 0x3f
            compins    = 0x02 if self.height == 32 else 0x12
            contrast   = 0xff # 0x8f if self.height == 32 else (0x9f if self.external_vcc else 0x9f)
            data = [DISPLAYOFF,
            SETDISPLAYCLOCKDIV, 0x80,
            SETMULTIPLEX, multiplex,
            SETDISPLAYOFFSET, 0x00,
            SETSTARTLINE | 0x00,
            CHARGEPUMP, chargepump,
            MEMORYMODE, 0x00,
            SEGREMAP | 0x10,
            COMSCANDEC,
            SETCOMPINS, compins,
            SETCONTRAST, contrast,
            SETPRECHARGE, precharge,
            SETVCOMDETECT, 0x40,
            DISPLAYALLON_RESUME,
            NORMALDISPLAY,
            DISPLAYON]
            for item in data:
            self.write_command(item)
            self.clear()
            self.display()

            def poweroff(self):
            self.write_command(DISPLAYOFF)

            def contrast(self, contrast):
            self.write_command(SETCONTRAST)
            self.write_command(contrast)

            def draw_text(self, x, y, string, size=1, space=1):
            def pixel_x(char_number, char_column, point_row):
            char_offset = x + char_number * size * font.cols + space * char_number
            pixel_offset = char_offset + char_column * size + point_row
            return self.columns - pixel_offset

            def pixel_y(char_row, point_column):
            char_offset = y + char_row * size
            return char_offset + point_column

            def pixel_mask(char, char_column, char_row):
            char_index_offset = ord(char) * font.cols
            return font.bytes[char_index_offset + char_column] >> char_row & 0x1

            pixels = (
            (pixel_x(char_number, char_column, point_row),
            pixel_y(char_row, point_column),
            pixel_mask(char, char_column, char_row))
            for char_number, char in enumerate(string)
            for char_column in range(font.cols)
            for char_row in range(font.rows)
            for point_column in range(size)
            for point_row in range(1, size + 1))
            for pixel in pixels:
            self.set_pixel(*pixel)
```

![](https://gitee.com/spring3th/pubpic/raw/master/img/温湿度.jpg)
