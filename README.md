# micropython-aiobutton
[![PyPI version](https://badge.fury.io/py/micropython-aiobutton.svg)](https://badge.fury.io/py/micropython-aiobutton) [![Downloads](https://pepy.tech/badge/micropython-aiobutton)](https://pepy.tech/project/micropython-aiobutton)

A MicroPython module for asyncio button.

This module only works under MicroPython and it is tested with MicroPython V1.13.

This module depends on uasyncio which is a standard package since MicroPython V1.13. It supports all kinds of buttons including buttons sharing a single ADC.

Please consider [![Paypal Donate](https://github.com/jacklinquan/images/blob/master/paypal_donate_button_200x80.png)](https://www.paypal.me/jacklinquan) to support me.

## Installation
``` Python
>>> import upip
>>> upip.install('micropython-aiobutton')
```
Alternatively just copy aiobutton.py to the MicroPython device.

## Usage
``` Python
from machine import Pin, ADC
import uasyncio as aio
from aiobutton import AIOButton

# Button UP and DOWN share a single ADC on pin 32.
adc_up_down = ADC(Pin(32))
adc_up_down.atten(ADC.ATTN_11DB)
adc_up_down.width(ADC.WIDTH_10BIT)
MAX_ADC = 1023

# Button ENTER is on pin 33 with pull-up enabled.
pin_enter = Pin(33, Pin.IN, Pin.PULL_UP)

# Initialise the button with a handler that takes one argument (the button itself).
# This handler should return True when the button is pressed, or False otherwise.
# Default button check time is 10ms, debounce time 50ms and hold time 1000ms.
btn_up = AIOButton(
    lambda btn: (MAX_ADC // 3) < adc_up_down.read() <= (MAX_ADC // 3 * 2)
)
# Press handler is triggered when the button is pressed down.
btn_up.set_press_handler(
    lambda btn: print("UP is pressed! State:{}".format(btn.get_debounced()))
)

btn_down = AIOButton(lambda btn: (MAX_ADC // 3 * 2) < adc_up_down.read())
# Release handler is triggered when the button is released.
btn_down.set_release_handler(
    lambda btn: print("DOWN is released! State:{}".format(btn.get_debounced()))
)

btn_enter = AIOButton(lambda btn: not pin_enter.value())
# Hold handler is triggered when the button is held for certain time (default 1000ms).
btn_enter.set_hold_handler(
    lambda btn: print("ENTER is held! State:{}".format(btn.get_debounced()))
)


async def main():
    task_up = aio.create_task(btn_up.coro_check())
    task_down = aio.create_task(btn_down.coro_check())
    task_enter = aio.create_task(btn_enter.coro_check())
    while True:
        await aio.sleep_ms(1)


aio.run(main())
```