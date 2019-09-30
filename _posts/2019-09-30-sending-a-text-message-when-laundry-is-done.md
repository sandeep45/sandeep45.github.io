---
title: Sending a text message when laundary is done
status: True
layout: post
categories: [nodejs, javascript, raspberrypi]
tags: [nodejs, javascript, raspberrypi, pigpio, twilio]
published: True
---

## Why
Wife insists that I must wash my gym clothes before going to bed, as they stink up the place. But if i just wash them and forget to put them in the dryer, they stink up my clothes, just a different kind of stink.

Unfortunately my washing machine doesnt have the ability to message and remind me to move my washed clothes to the dryer. Neither do i have a fancy All-in-1 washer and dryer machine. 

So I retrofitted my age old washing machine with the ability to text when it goes off.   

## What
I need to detect when the machine has stopped and then notify myself via a text message. So Lets break this down:

- Detect when machine is off
- Somehow know that machine was on before and has just gone off
- Send an alert to whoever cares to receive it   
 
At high level this can be done by using a:

- Current Transformer(CT)
- Raspberry Pi
- Some basic electronic parts like resitor, wire, PCB etc. 
- [Twilio](www.twilio.com/referral/rEcrln)

## How

When the machine is ON, current goes through its wire. A current transformer is clamped on this wire and it senses the current going through it. It produces a current on its output/secondary circuit proportional to the current going through the wire in its primary circuit. The current in the secondary circuit can be used to determine if the machine is ON or OFF.

Once I have a signal, I am able to have it go through a resistor, and then feed that to an Analog to Digital Convertor(ADC) and then connect that to a raspberry pi.

When its all done the circuit looks like:

<img src='/assets/laundry/pic1.jpg' class="center" />

and the CT looks like :

<img src='/assets/laundry/pic3.jpg' class="center" />

Within my raspberry pi, I maintain state of the machine and on state change from 0 to 1, a text message can be sent easily via twilio.

## Tricky Parts

- The Current Transformer(CT) needs to go over only 1 wire. if it goes over both of them, then it will produce no result as current flow will cancel it self. Take a look at this picture and you will see that I have seperated one wire and transformer is only over that wire.

<img src='/assets/laundry/pic7.jpg' class="center" />

- the input current in the primary circuit is AC, so its going to produce a small amount of AC current in the secondary circuit for you. You need to figure out how to use this. There are many way to deal with this. My strategy was to use a voltage divider and ensure that the output volatage is never negative as I dont want to give my ADC negative voltage. 

- you must have a load resistor attached on the secondary circuit of your Current transformer. Not having this is like having two live wires dangling around each other with nothing connected to it. The value of the load resistor is important as it will decide how much resolution you get in your secondary circuit. Here is a handy [calculator](https://tyler.anairo.com/projects/open-energy-monitor-calculator) to choose an appropriate value.

- Since we are reading values of an analog sensor and are reading them over a sine wave, many values could be wrong and we need to compensate for that either in software or hardware. What i mean is that when device is OFF, everytime we read the value it should be 0. And if its ON, we should read a 1. But sometimes or shall i say many times we may read a wrong value. We could get something like 0,0,0,0,1,0,0,0 when the machine is off. We dont want 1 wrong value to trigger our alert. You can compensate for this in software by maintain a queue of recently read values and only making a decision when all values in the queue are the same and make no change when all values are not the same. This way if you have queue of 10 values which are being read 1 second apart, you will need 10 consistent 0's to establish that the machine is OFF and 10 consistent 1's to establish that the machine is ON.
   
- Most laundry machines will actually not draw any current while they are in certain parts of the wash cycle like filling up water, rinsing before spinning etc. So its important to deal with them in your code. I handled this by doing wrapping my call to send the text message in a `setTimeout` of 5 minutes and clearing the timeout when the machine became ON. This way the text message was delayed by 5 minutes, but only sent if the machine had been off for more than 5 minutes. This is a heuristic and your mileage may vary(YMMV).

- To handle incoming twilio notifications of peopple who want to subscribe & unsubscribe I would need to know the IP address of my raspberry pi and then connect that in twilio. Since most homes dont have a fixed IP, I went with spliting the code and putting the twilio related code on heroku which gives me a fixed address. Now twilio can send text message notifications there and my raspberry pi can also notify this service when the washing machine turns off

## Extras

- Since the current in the secondary circuit is proportional to the current flowing in the primary circuit of the CT, code could be written to track how much electricity is being consumed by the washing machine.

- I attached an LED to my Raspberry pi and I am driving it from the value i receive from the CT. So when the machine is ON, it writes a 1 to the LED and the LED turns ON. Similarly when machine goes OFF, it writes a 0 and LED goes off. It also very neatly demonstrates the issue i mentioned, where wrong values are randomly read. This is rather neat and can be seen by the LED momentarily going off while the machine is running. 

## Parts List
- [Raspberry Pi 4](https://amzn.to/2modkn2)
- [Pi Open Case with room for PCB](https://amzn.to/2modkn2)
- [ADC Chip](https://amzn.to/2ndX5JM)
- [CT Transformer](https://amzn.to/2nZ9LnN)
- [Resitors](https://amzn.to/2oLwqV7)
- [Single Core Wire](https://amzn.to/2nXuVmg)
- [LEDs](https://amzn.to/2oKKQEX)
- [Connectors](https://amzn.to/2o3Vki9)
- [Extension Cord with ease to seperate wires](https://amzn.to/2oLG2iF)

## More Pictures

<img src='/assets/laundry/pic2.jpg' /> <br/><br/>
<img src='/assets/laundry/pic4.jpg' /> <br/><br/>
<img src='/assets/laundry/pic5.jpg' /> <br/><br/>
<img src='/assets/laundry/pic6.jpg' /> <br/><br/>

## Referneces

- [BrewBot](https://github.com/command-tab/brewbot)
- [CT Sensors](https://learn.openenergymonitor.org/electricity-monitoring/ct-sensors/yhdc-sct-013-000-ct-sensor-report)
- [PI Wiring Diagrams](https://docs.microsoft.com/en-us/windows/iot-core/learn-about-hardware/pinmappings/pinmappingsrpi)
- [ADC Wiring Diagram](https://learn.adafruit.com/raspberry-pi-analog-to-digital-converters/mcp3008)

## Disclaimer

This article has affiliate links. As an Amazon Associate I earn from qualifying purchases.

