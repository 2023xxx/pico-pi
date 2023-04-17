# pico-pi
questions
from machine import Pin , UART ,Timer
import time
import utime
import uos



gsm=UART(0,baudrate=115200,parity=None,tx=Pin(12),rx=Pin(13))
led = Pin(25, Pin.OUT)
my_pin=Pin(16,Pin.OUT)
#int_pin = Pin(4, Pin.IN,Pin.PULL_DOWN)
in_buf=[]
uart_recive_buf=[]
gl_str_input=''
gsm_button=Pin(0,Pin.OUT)
input_pin=Pin(4,machine.Pin.IN)
admin_num=9154811626




def gsm_interupt(x):
    print("we should check gsm module")

#INITIALIZING A PIN FOR GSM SMS & CALL RECIEVED  INTERUPT    
gsm_interrupt_pin=machine.Pin(0,machine.Pin.IN,machine.Pin.PULL_UP)
#setup the interrupt request handeling for gsm_interrupt_pin change of state
gsm_interrupt_pin.irq(trigger=machine.Pin.IRQ_FALLING,handler=gsm_interupt)



def my_timer_handler(x,uart=gsm):
    global uart_recive_buf
    led.toggle()
    AT("AT")

            
    

my_timer=Timer(mode=Timer.PERIODIC,period=15000,callback=my_timer_handler)

def AT(cmd , uart=gsm,timeout=100):
    in_buf.clear()
    print("CMD: "+ cmd)
    uart.write(cmd.encode()+"\n\r")
    waitResp(uart,timeout)

def AT_m(cmd , uart=gsm,timeout=1000):
    in_buf.clear()
    print("CMD: "+ cmd)
    uart.write(cmd)
    waitResp(uart,timeout)
    

def waitResp(uart=gsm,timeout=100):
    global gl_str_input
    prvMills=utime.ticks_ms()
    resp=b""
    
    while(utime.ticks_ms()-prvMills)<timeout:
        if uart.any():
            resp=b"".join([resp,uart.read()])            
            if resp !='':
                #in_buf.append(resp)
                gl_str_input=resp
                print("resp >>" ,resp)
                #print("in buff >>" ,in_buf)

def gsm_turn_ON_or_OFF():
    print("gsm is turning ON")
    gsm_button.high()
    utime.sleep(3)
    gsm_button.low()
    
                

def send_sms(massage):
    mobile_num=+9151111111
    print("sending >>>> SMS")
    AT("AT+CMGF=1")   #changing to text type isteas of PDU
    AT("AT+CMGS=\""+str(mobile_num)+"\"")
    AT_m(str(massage))
    AT_m(chr(26))    # ctrlz
    
    
    
def check_AT_Ok():
    global gl_str_input
    #in_buf.clear()
    AT("AT")
    if  'OK'  in gl_str_input:
        print("GSM module is >>>> working\r\n")
        return True
    else:
        return False


    

def gsm_init():
     print((machine.freq()/1000000),"MHZ") 
     if  not check_AT_Ok():
         
         gsm_turn_ON_or_OFF()
     AT("AT+CNMI=2,2,0,1,0") #sms reciving configuration

     AT("AT+CMGF=1")

        
gsm_init()
in_buf=[]
led.high()

cnt=0
while True :
        cnt+=1
        if(cnt%100000==0):
            print(cnt/100000)

        while gsm.any():
            print(gsm.read())
  
    

