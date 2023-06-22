---
layout: post
title: TM4313 GPSDO Teardown
date:   2023-06-16 00:00:00 -1000
categories:
---

* TOC
{:toc}

# Introduction

It's a generally accepted truism that once you've acquired your first frequence
counter, you slowly get sucked into the Cult of 
[Time Nutts](http://www.leapsecond.com/time-nuts.htm), 
and the never ending quest of increasingly accurate time measurements.

In my [previous blog post](/2023/06/16/Frequency-Counting-with-Linear-Regression.html)
about measuring frequency with high precision, I mentioned the need for an 
accurate time reference to measure time accurately.

Such time reference can be derived from different kind of devices.

# Different Precision Oscillators 

**Crystal oscillator**

You can find crystal oscillators in pretty much any modern electronic device.
They are dirt cheap, their high volume price is just a few cents, but their
accuracy is not great, and usually measured in hundreds of ppm.

That's good enough for anything that doesn't require a super accurate clock
but it will limit the accuracy of a frequency counter to just 3 or 4 digits.

One reason for their limited accuracy is the temperature dependent behavior
of the oscillating crystal.

**Voltage controller crystal oscillator**

A voltage controlled oscillator (VCXO) has an input that add external control of the
output frequency. Changing the voltage on the input up or down will change the
output frequency. The range over which the output frequency can be controlled is
call the [absolute pull range (APR)](https://www.renesas.com/us/en/document/apn/847-vcxo-absolute-pull-range). 
VCXOs can have an APR from 10 to 2000ppm.

You could use a VCXO to statically trim the oscillator against a known good timing
reference to correct the worst systematic errors of a regular crystal oscillator, but 
VCXO are almost always used as part of a dynamic feedback loop, against some reference, as
part of some PLL.

All the oscillators below are have a VCXO in them.

**Temperature controlled crystal oscillator**

A temperature controlled crystal oscillator (TCXO) takes the logical step of adjusting
the output frequency based on the ambient temperature of the oscillator. TXCO consists
of an embedded temperature sensor, a control circuit, and a VCXO. The control circuit
measures the temperature and adjust the input voltage of the VCXO so that inherent temperature
dependency of the crystal oscillator is counteracted.

The result is a much more stable output. TXCOs can have temperature stability
of 2ppm down to 0.2ppm.

Note that TCXOs will still have a separate input control pin for the user, so that
the user can design an additional circuit around the TXCO to obtain even better accuracy.

**Oven controlled crystal oscillator**

Oven controlled crystal oscillators (OXCO) take things to the next level: instead of
adjusting the input control voltage to compensate for changing ambient temperature,
they embed the VCXO in an oven: a metal enclosure with a heating element, 
usually a transistor, and insulation material to better shield the inside of the
oven from outside temperature changes. There's still a temperature sensor, but
instead of adjusting the control input of the VCXO, it is used to control the
heating element to maintain a constant temperature inside the oven.

The typical temperature stability of an OXCO ranges from 0.1 all the way down to
0.001ppm.

A frequency counter with an OXCO that is well calibrated can have a precision of
9 digits, or higher when the counter itself located in a room with a constant
temperature.

**Double-oven controlled crystal oscillator**

The double-oven controlled oscillators (DOXCO) has, you've guessed it, not
one but two ovens! This way, the temperature of the crystal can be controlled
even better, often within a range of just 0.10C.

**Rubidium Clocks**


# GPS Disciplined Oscillators


# The TM4313 GPSDO 

I stumbled onto the TM4313 while searching for GPSDOs on eBay. I hadn't seen these before
and with a nice metal enclosure and one of the lowest GPSDO prices around, they seemed
attractive.

There is almost no information about the TM4313: when you google for just the "TM4313" terms,
there's tons of GPSDO related results, but none of them actually discuss the TM4313, except
for this one in-depth review paper: 
[Comparison of Inexpensive 10 MHz GNSS Disciplined Oscillators](https://reeve.com/Documents/Articles%20Papers/Reeve_GDOComp.pdf).

'It judged the TM4313 to a good option, so I placed an order. Shipping time was supposed to 
be a few weeks, but it arrived only a few days after ordering.

The non-descript box contained the following items:

[XXX]

* the GPSDO unit itself
* a GPS window antenna with a long cable
* a USB-A to micro-USB cable
* a 5V power brick

The unit itself has a good looking aluminum enclosure with front and
back plate that are aluminum as well.

The front plate has 4 blue LEDs for power status, time tracking status,
GPS lock, and serial communication status.

There's a micro-USB port to connect the GPSDO to a PC and extract time information. And
there's the 5V power plug. I would have prefered the power plug to be in the back but
there wasn't any room left there.

The back plate has 3 SMA connectors: the GPS antenna input, a 10MHz reference clock
outputs, and 1 pulse-per-second (PPS) output.

The 10MHz reference output of a GPSDO is either a sine wave or a digital square wave.
The benefit of a sine wave is the lack of strong harmonics in the signal. The TM4313
doesn't come with a specification sheet or manual, and there is no mention anywhere
about the type of output. When put on a scope, I can see 10MHz sinewave with an
amplitude of +-1.1V into a 50 Ohm termination resistor.

# Inside the TM4313

Opening the TM4313 is a simple matter of removing 2 Torx T6 screws and sliding out the PCB.

Here are the main components:

* an HBFEC OC5SC25 OXCO

    This seems to be a clone of a CTi OC5SC25, which can be found for just a few dollars
    on eBay or AliExpress. Tony Albus has the [following diagram](https://www.tonyplaza.nl/download/YT079/CTi_OSC5A2B02-CTi_OC5SC25.pdf)
    about it.

* an unmarked GPS module

* an NXP LPC1752 microcontroller

* an FTDI232RL USB-to-serial converter

* power regulators and a few transistors

Is that all there is? No!

The OCXO is not soldered flush against the PCB, but floats a few mm above it.
And there's 2 active components soldered in between them!


# References

* [Bliley - Crystal Oscillators: The Beginner's Guide (OCXO, TCXO, VCXO, & Clocks)](https://blog.bliley.com/quartz-crystal-oscillators-guide-ocxo-tcxo-vcxo-clocks)
* [Comparison of Inexpensive 10 MHz GNSS Disciplined Oscillators - W.D. Reeve](https://reeve.com/Documents/Articles%20Papers/Reeve_GDOComp.pdf)


