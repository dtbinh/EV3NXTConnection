# EV3NXTConnection
Send data from a Mindstorms EV3 to a Mindstorms NXT brick with a modified sensor cable.

## Motivation

While it is very simple to connect EV3 bricks either via USB, Bluetooth or WLAN, no such option 
exists to connect an old NXT to an EV3. Most possibilities currently available require
some intermediary processor (PC, Arduino, etc.) to do the necessary transformation of the
data formats. Other possibilities require reprogramming the firmware of one or both bricks.

I wanted to create at least some kind of connectivity, no matter how slow, to transfer any amount of
data from the EV3 to an NXT by using simple wiring. Possibly with some small electronic elements.

After endless tries and failures, I finally found a way to create a slow-speed 
communication link from EV3 to NXT over such a modified sensor cable. The link is able to send 
an 1-out-of-4 command every 50ms. This could indeed be sufficient for many models.
When more complexity is needed, multiple pulses could be transferred for a single command
(of course at the cost of higher overall transmission times).

## Operation principle
While it is not possible to connect the I2C bus link from the EV3 to the I2C bus of the NXT
(both sides can only work as the single master), it is nevertheless possible to use the I2C 
facility of the EV3 (using standard firmware only) to create some outbound pulses on the 
SDA line (pin 6, the blue wire). When connected correctly to the analog input of the NXT
(pin 1, the white wire), the NXT could somehow sense these pulses.

## Abusing the I2C bus to generate a pulse of defined length
Because the NXT has a quite slow sample rate for its analog input ports (333 Hertz), any
signal must be at least about this long for a reliable detection. 
When trying to send/receive something on the I2C bus, the master sends 8 bit to address a slave
and expects the slave to respond with an acknowledge. Without acknowledge the EV3 stops there.
These 8 bits (including some extra start/stop frame) will take about 1ms to send, which is to
short for a reliable detection on the side of the NXT.
To solve this issue, the cable wiring contains a 10nF capacitor that can hold the data line
low long enough for the EV3 to believe a valid acknowledge was sent. Then the EV3 continues
sending data (always zero bits) to actually generate a low pulse on the DTA wire that is
long enough for the NXT to detect reliably and which can even be varied in length between
5ms and 35ms.

## Detecting the pulse  
On the NXT the pulses arrives at the analog pin which is sampled every 3 ms. The NXT program 
must permanently read the sensor to detect the begin and end of a pulse. 
The timing is not very accurate because of the limited sample rate, but the pulse length
can be measured accurately enough to distinguish 4 different pulse lengths.
These can be interpreted as different commands.

## The cable wiring and more detailed explanation

(also see diagram)

* Pin 3 (red) must be connected through 
	This is the ground pin
* Pin 6 (blue) of the EV3 must be connected via an 68kOhm resistor  to Pin 1 (white) of the NXT.
   This resistor in conjunction with the 10kOhm pull-up inside the NXT forms the required 80kOhm
   pull-up of the I2C data line.   
* A 10nF capacitor must be connected between Pin 3 (red) and Pin 6 (blue) of the EV3.
   When the EV3 sends a byte containing only zero-bits (which is the only thing the program
   will send in this setup) the capacitor will quickly reach GND level which it can keep just long
   enough to make the EV3 believe an acknowledge was sent. The EV3 will then continue sending
   the intended number of zero-bytes.
* No other pins must be connected.

