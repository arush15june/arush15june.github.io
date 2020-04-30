---
published: true
title: "Bits & Signals: Qatar International Cybersecurity Competition"
date: 2019-11-24T00:00:00.000Z
categories: ["competition"]
tags: ["qicc", "qatar", "ctf", "hardware"]
---

* *TL;DR*: Al Capwn goes international! We won the second prize hacking hardware with our 15$ (1000 INR) SDR and Logic Analyzer in Doha, Qatar!

In October 2019, Members of Al Capwn flew from New Delhi, India to Doha, Qatar to attend [Qatar International Cybersecurity Contest](https://www.hbku.edu.qa/en/qicc) at Hamad Bin Khalifa University. We are thankful of HBKU for sponsoring the complete trip. Specfically, We participated in the Qatar International Hacking Contest, a hardware hacking competition.

This post is going to document the hardware hack, how we conducted security research on the device's presented to us during the competition. 

## Travelog

Being my first international flight, Going for the red wine during the Qatar Airways flight turned out to be a great choice. The first thing we did after landing was get on hand some $$$ to spend, for this I had a [Niyo Global Card](https://www.goniyo.com/). They promise zero fee conversions via their international Visa card, and it worked just as good as a local debit card at both ATMs and POS terminals. Compared to international cards from big banks which charge fees on every transaction, We got zero fees on all our transactions while having control via their decently built application. 

Apart from offline payments, The card supports international online payments. Now, you can transact at international ecommerce like AliExpress easily without having a credit card, A problem I often experience as a student with just a debit card. Though, One major problem I faced was that PNB (my savings account) didn't allow NEFT/IMPS transfer to the DCB Bank account registered with the Niyo card.

![Premier Inn, Education City](/img/premier_inn.jpg)

We stayed at the Premier Inn in Education City during the competition. On the first night, we discovered that Zomato operated there and delivered pizza. Though, I wasn't able to make use of my Zomato Gold :P. We visited the Mall Of Qatar (a very big mall, as the name suggests) for a little while which turned out to be a huge nerdgasm. For many my favourite materialistic pleasures, including Sneakers and Video Games all there to play with.

![Playstation Classic](/img/PSClassic.jpg)

The best part was that during the whole week of stay, in this country full of sand, my white sneakers just as white as they were when I left home. This is the future.

## Qatar International Hacking Contest

A week before the competition we were notified of the 12 devices that we would be given access to at the event, but the specific versions of the devices weren't given to us two days until the competition. We had research done on the devices and strategized to target a few devices which we demmed were attackable. This included a restaurant pager, Ring Video Doorbell camera, a wifi based home security system, and a wifi security camera. The 12 devices had some big shot devices included two Echo Speakers, the latest Bose NCH700 headphones and the Ring wireless alarm kit which worked over Z-Wave. Along with the devices, The organizer's provided us with a HackRF SDR, an iFixit toolkit and an Alfa wireless dongle with monitor mode.

![Hardware](/img/hardware.jpg)

The whole event was divided into 4 sections of 2 hours each over 2 days which was annoying to produce any concrete results, on top of that we were, at first, not allowed to dismantle the devices but were allowed on the condition that the device's integrity be maintained. We opened the devices and packed them back again neatly before the end of the competition.

Our toolkit which we brought from home consisted of Multimeter's, cheap Arduinos, a CP2102 based USB FTDI board, a Cypress FX2 based el cheapo [logic analyzer](https://robu.in/product/usb-logic-analyze-24m-8ch-mcu-arm-fpga-dsp-debug-tool/) having 8 channels paired with the fantastic open source software [Sigrok](https://sigrok.org/) and [PulseView](https://sigrok.org/wiki/PulseView), a variant of the RTL-SDR, and a FOSSASIA [PSLab](https://pslab.io/). All my love to [Open Tech Lab](https://www.youtube.com/channel/UCeF7JKNXOy0jpMOxpgbZcpg) for its immensely information videos from which I learned so much about SDRs, electronics and using Sigrok and PulseView.

[This](https://labs.portcullis.co.uk/blog/hardware-hacking-how-to-train-a-team/) also turned out to be very useful.

### [Nadamoo Restaurant Pager System](http://www.nadamoo.cn/products/PagerSystem/SquareStyle/2017-09-14/11.html)
This device consists of a base station and a receiver, The receiver is given to a customer and when their food gets ready the employees ring up the receiver by sending signals to it from the base station.

![Nadamoo Restaurant Pager System](/img/nadamoo.jpg)

Our initial research on Restaurant Pagers revealed various articles and blog posts on the internet by amazing hackers. [This](https://hackaday.com/2019/07/30/teardown-catel-ctp300-restaurant-pager/), [this](http://www.windytan.com/2013/09/the-burger-pager.html), and [this](https://www.rtl-sdr.com/using-a-hackrf-to-reverse-engineer-and-control-restaurant-pagers/) turned out be to very useful for the event. After setting up the device and using the pagers with the base station for a brief, instead of listening to the frequency written at the back of the base station (433.92MHz) I opened up the base station and deduced/guessed from the circuit board and the components that a shift register was used to serialize the data protocol generated from an IC and sent over to the RF component. I hooked my logic analyzer to the pins connecting the number board of the base station to the RF board. Using PulseView, we analyzed the signal transmitted after sending a command to one of the receivers. On one of the lines, We found a digital signal and started reverse engineering the signal.

![Nadamoo Logic Analyzer](/img/logic.jfif)

The complete generated signal consisted of long bursts repeated continuously for a second. Inspecting one of the bursts revealed that they contained the same repeating pattern. This pattern was the protocol which identified the receiver for which the signal was meant.

![Reverse Engineered Nadamoo](/img/reverse-nadamoo.png)

* The signal is an On-Off Keyed digital signal with a structure consisting of 28 bits.

* Preamble (17 bits)
  + 10101011110110001
    - This also contained the identifier of the base station linked with the receiver. 

* Sync (2 bits)
  + 00
  
* Pager Identifier (8 bits)
  + The pagers are identified by the bitstring in these 8 bits.  The bits are represented in Little Endian.
  
* Terminator (1 bit)
  + Last bit terminates the message with a 0 bit.

**<p align='center'>10101011110110001 00 11000000 0</p>**<p align='center'>For pager numbered 3.</p>

![Reverse Engineered Nadamoo](/img/nadamoo-table.png)
*<p align='center'>Pager ID's: 3, 4, and 5</p>*

We confirmed the reverse engineered protocol by tuning the HackRF to 433.92 MHz and listened to the signals using [Universal Radio Hacker](https://github.com/jopohl/urh) which worked like a charm on Windows!

![URH and Pager](/img/URHpager.png)

The demodoulated PWM signal in URH correctly represents the reverse engineering protocol. Due to its fixed nature, the protocol is easily replayable using an inexpensive 433MHz Transceiver module and an Arduino or an expensive Tx-capable SDR (like the HackRF we had).

### [Fortress Security System](https://www.fortresssecuritystore.com/wireless-alarm-system-bundles/s03-wifi-alarm-system-kits.html)

It is a security system which lets you rig your house with battery powered sensors communicating with the base station over wireless radio. The base station exposes an interface over wifi to allow control via an Android App. We were given the Fortress S03 kit which only has wifi, they also have other kits which are able to communicate over 3G and 4G.

The first step, use and understand the device. The S03 kit contains a base station and many peripherals, this included: a Remote Key Fob, a Panic Button, a Door Contact Sensor, and a PIR Sensor. This means you put the PIR and Contact sensors all over your house and arm the system which rings a high-pitched noise whenever someone passes them and wake you up. 

![Fortress S03 PCB](/img/fortresspcb.jfif)

The product website and manual wasn't very descriptive of the hardware used in the system. So we opened up the base station and the peripherals, inside we found familiar chips and exposed pads. The main CPU was a Mediatek MT6261, a trusty ESP8266 for WiFi, and a 433MHz **receiver** (SYN510R), no transmitter. This revealed how the peripherals communicated to the base station and a new attack surface for us to research.

The PCB had 5 conveniently marked and exposed pads near the ESP8266 named V, G, IO, **RX, TX**. Hoping it to be a raw UART to the ESP8266, I connected the FTDI board and opened Putty in the search of its baud rate. I tried the common baud rates found online with ESP8266 projects, 9600, 115200, etc. Didn't work. Finally, I opened up the Arduino IDE and it's serial monitor and scrolled through its list of conveniently listed common baud rates and found ASCII logs of the wifi connection at 250000 baud/s. Unfortunately, The ESP8266 didn't accept any input over the UART port (and/or we weren't able to find a way to trigger any mode if there was any).

![Putty ESP8266 UART Logs](/img/putty-esp-uart.png)
**Debug logs from the ESP8266**

We knew the peripherals communicated over 433MHz RF. Using [Universal Radio Hacker](https://github.com/jopohl/urh) and the HackRF, I looked at the signals sent out by the peripherals like the Key Fob and the Panic Button. Suspecting them to be common protocols instead of decoding them by my own, so me googling suggested a popular library [RTL_433](https://github.com/merbanan/rtl_433) for the RTL-SDR which decoded protocols from many generic 433MHz devices. Setting up RTL_433 on my Manjaro VM was a breeze and it decoded the protocols from the peripherals: The key fob, panic button (which was the same device as the key fob essentially), PIR Sensor and Door Contact Sensor. All of them were decoded by RTL_433 and corresponded correctly to the actions performed. 

![Keyfob](/img/keyfob.jfif)
**Keyfob**

![Keyfob Decode](/img/keyfob-code.png)
**Decoded data from the Keyfob**

![PIR Decode](/img/PIR-Sensor.png)
**Decoded PIR sensor protocol**

![PIR Sensor](/img/PIR.jfif)
**PIR Sensor**

The protocols used by the peripherals were fixed key codes and were easily replayable. The keyfobs are able to control the arming and disarming of the base station, replaying its signals an attacker can disarm or arm the system at will with a 433 MHz trasnmitter or an SDR.

## Conclusion
We reported all our findings from the Nadamoo Restaurant Pager, Ring Video Doorbell, Lefun Wireless Camera, Letscom Activity Tracker and the Fortress Security to the organizers. Turns out we fared well compared to the competitors and were awarded the second prize at the competition! We met many phenomenal people of the hacker community and had an amazing trip in Qatar.