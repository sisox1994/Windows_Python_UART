```python
# Windows 10 64-bit
# Python 3.9.7 
# HW: USB UART Dongle CP2102

import serial
import threading

import tkinter as tk
from tkinter import filedialog


# ==================== UART Parameter Config Region =================
uart = serial.Serial()
uart.port = "COM18"
uart.baudrate = 115200
uart.bytesize = serial.EIGHTBITS  # number of bits per bytes
uart.parity = serial.PARITY_NONE  # set parity check
uart.stopbits = serial.STOPBITS_ONE  # number of stop bits
uart.timeout = 0.5  # non-block read 0.5s
uart.writeTimeout = 0.5  # timeout for write 0.5s
uart.xonxoff = False  # disable software flow control
uart.rtscts = False  # disable hardware (RTS/CTS) flow control
uart.dsrdtr = False  # disable hardware (DSR/DTR) flow control


#========== Global variable Define ============
uart_rx_buffer = b''


def clr_uart_rx_buffer():
    global uart_rx_buffer
    uart_rx_buffer = b''


def Master():    
    print("<Master ID>:", threading.get_ident())   

    # -------------- UI code here --------------


# ================ UART RX ===============================
def UartRxBack():  

    global uart_rx_buffer     
    print("<UartRxBack ID>:", threading.get_ident())

    while 1:
        
        try:
            read_byte = uart.read(1)
        except:
            read_byte = b''
        
        if(read_byte != b'\n'):
            uart_rx_buffer += read_byte
        elif ((read_byte == b'\n')):
            # 偵測到 '\n' 換行符號 用utf8解析，並print出來
            string_buffer = str( uart_rx_buffer , encoding = "utf-8" )
            print(string_buffer)

            # 取出字串裡面的數值spilt出來，並放進 number_array 陣列
            number_array=[]
            for temp in string_buffer.split(":"):
                for temp2 in temp.split(","):
                    if(temp2.isdigit()):
                        number_array.append(int(temp2))

            # 將spilt後的數值隔空格印出來
            number_array_string = ''
            for number_value in  number_array:
                number_array_string += str(number_value) + " "
            print(number_array_string)

            # 資料包解析完成 清空 uart_rx_buffer 才能接收下一包資料
            clr_uart_rx_buffer()

# ================ UART RX ====================================

if __name__ == "__main__":
    
    print("<Main ID>:", threading.get_ident())

    try:
        uart.open()
        print("open serial port OK ")

        # ===== UART TX ==========
        w_buffer = b'Hello'
        uart.write(w_buffer)
        # ===== UART TX ==========

    except Exception as ex:
        print("open serial port error " + str(ex))
        exit()

   

    task_1 = threading.Thread(target=Master)
    task_2 = threading.Thread(target=UartRxBack)

    # 如果有寫UI才需要把 task_2.setDaemon 設 True (功能:視窗關閉時，同時結束task_2)
    #task_2.setDaemon(True)

    task_1.start()
    task_2.start()
```

