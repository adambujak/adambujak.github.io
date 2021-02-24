# Introduction 

There's a bit of a learning curve to move from developing high level software to developing embedded code on something like an Arduino or a BeagleBone. I found the learning curve of moving from Arduino to a more professional microcontroller (the nRF52832 in my case) to be much more troublesome. I hope to make this a bit easier for you. 

I'm assuming you're familiar with basic C, and the very basics of Arduino (GPIO, ADC, etc.)

# Getting Started

Once you know how to program your microcontroller, it all makes sense (duh,) but when I was getting started with my nRF52DK I was so confused by all the things that were available to download. There's an SDK, the "Soft Devices", the nRF Programming Tools, the JLink Software, SEGGER Embedded Studio, the ARM GNU Compiler... needless to say, it's overwhelming when you're new to this kind of stuff. 

I'm going to break down each of these things individually since the concepts are generally applicable to most microcontroller families. 

## SDK 

You have likely heard of SDKs (Software Development Kits) before; they are essentially a collection of pieces of software that are made to make your development work quicker and easier. If you want to develop Android apps, you download the Android SDK, if you want to develop embedded software you download the SDK for the microcontroller you're using (provided it exists.) In an embedded environment the SDK usually includes example projects, a HAL (Hardware Abstraction Layer - see below), tools, etc - basically everything you need to get started to building your software. 

### HAL - Hardware Abstraction Layer
The HAL is one of the most useful components in an SDK. You don't have to use it if you don't want to, but it makes life much easier. To understand what it is consider the following Arduino code:

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
