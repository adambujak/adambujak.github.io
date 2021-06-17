# Introduction

There's a bit of a learning curve to move from developing high level software to developing embedded code on something like an Arduino or a BeagleBone. I found the learning curve of moving from Arduino to a more professional microcontroller (the nRF52832 in my case) to be much more troublesome. I hope to make this a bit easier for you.

I'm assuming you're familiar with basic C, and the very basics of Arduino (GPIO, ADC, etc.)

# Getting Started

Once you know how to program your microcontroller, it all makes sense (duh,) but when I was getting started with my nRF52DK I was so confused by all the things that were available to download. There's an SDK, the "Soft Devices", the nRF Programming Tools, the JLink Software, SEGGER Embedded Studio, the ARM GNU Compiler... needless to say, it's overwhelming when you're new to this kind of stuff.

I'm going to break some of these things individually since they are generally applicable to most microcontrollers (MCUs.)

## SDK

You have likely heard of SDKs (Software Development Kits) before; they are essentially a collection of pieces of software that are made to make your development work quicker and easier. If you want to develop Android apps, you download the Android SDK, if you want to develop embedded software you download the SDK for the microcontroller you're using (provided it exists.) In an embedded environment the SDK usually includes example projects, a HAL (Hardware Abstraction Layer - see below), tools, etc - basically everything you need to get started to building your software.

### HAL - Hardware Abstraction Layer
The HAL is one of the most useful components in an SDK. You don't have to use it if you don't want to, but you'd be crazy not to. To understand what it is consider the following Arduino code:

```
#define LED_PIN 13
void setup() {
  pinMode(LED_PIN, OUTPUT);
}

void loop() {
  digitalWrite(LED_PIN, HIGH);
  delay(1000);
  digitalWrite(LED_PIN, LOW);
  delay(1000);
}
```

This code toggles the voltage level on pin 13 every second. Now, how does that actually happen?

The Arduino UNO R3 uses an ATmega328P as the MCU, and its datasheet can be found [here](https://ww1.microchip.com/downloads/en/DeviceDoc/ATmega48A-PA-88A-PA-168A-PA-328-P-DS-DS40002061B.pdf).

If we look at section 14 of the datasheet we can understand how I/Os work on this chip. I recommend reading this yourself, but the important takeaway is we can configure the MCU's pins by writing the registers in the I/O hardware block.

We set the pin direction (in or out) with the DDRx register, by setting the corresponding pin's bit (1 for output, 0 for input,) and we set the pin's output value by setting the pins bit in the PORTx register, where x represents the letter of the port.

For instance for pin PB5 we'd be using the registers DDRB and PORTB.

That's a bit hard to understand if you've never seen anything like that before, so keep reading for a practical example.

Pin D13 on the Arduino UNO R3 is actually pin PB5 on the ATmega328P, so I'll continue to use that in the following examples.

These are the registers we are working with:

![Registers](assets/images/atmega_gpio_registers.png)

So in order to toggle the pin we need to set it as an output by setting bit 5 of DDRB to 1 this can be done by the following C code: `DDRB = 1 << 5;`. The `<<` is called a bitshift operator, in case you're confused by that. Keep in mind that this will write the value of 1 << 5 to the register, so if you already have some other pin in port B configured as an output, this will reconfigure it to an input. To avoid this you can bitwise OR the register with your new value: `DDRB |= (1 << 5);` .

Then to set the pin high we need to set bit 5 of the PORTB register to 1. Again this can be done by `PORTB = (1 << 5);`.


So the equivalent C code of the Arduino code above will look like this:

```
#include "ATmega328P.h" // include the register information so we have DDRB and PORTB
#include "delay.h"      // include a delay function

int main()
{
	DDRB = 1 << 5;  // Configure pin PB5 as output

	while(1) {
		PORTB = (1 << 5);   // Set pin PB5 high
		delay(1000);
		PORTB = (0 << 5);   // Set pin PB5 low
		delay(1000);
	}
}
```

There are three main things that are simplified in this code:
  1) We include the ATmega328P.h file. In practice you will always include something like this as this is what will define the registers on your MCU.
  2) We inclue delay.h. This is just to make this code simpler, usually there will be a delay function defined for you somewhere in the SDK.
  3) We ignored any chip setup that usually would happen such as setting up any external oscillators, enabling clocks, etc. that is out of the scope of this example.

There is still some mystery around point 1) above. Where do DDRB and PORTB come from? Well they are defined for us in the above example, but we could define them ourselves by casting a pointer to an address.

From the datasheet we can see that DDRB is at address 0x24 and PORTB is at address 0x25.

So we can write some simple code above our main function to define these registers:

```
#include <stdint.h>
#define DDRB    (*((volatile uint8_t*) 0x24))
#define PORTB   (*((volatile uint8_t*) 0x25))
```

[This](https://blog.feabhas.com/2019/01/peripheral-register-access-using-c-structs-part-1/) is a good article for more details about this.
