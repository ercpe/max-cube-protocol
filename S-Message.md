# The S Message

* The s command is to send settings to the connected devices.
* The S message contains the response.

## The s Command

The outgoing 's' command looks like this:

    s:AARAAAAAB5EAAWY=\r\n

Like other messages the parameter is base64 encoded.

Converted to hex, this is: 

    0000000000: 00 04 40 00 00 00 07 91  00 01 66 

The first part of the s command  consists of following fields:

    Description        Length      Example Value
    =====================================================================
    Base String        6           000440000000
    RF Address         3           0FDAED
    Room nr            1           01

### Base String

The base string is preceding the detailed settings and determines what parameter will be set.
	
	000440000000  for temperature and mode setting
	000410000000  for program data setting
	000011000000  for eco mode temperature setting

	000412000000  for config valve functions
	000020000000  for add link partner
	000021000000  for remove link partner
	000022000000  for set group address
	00??23000000  for remove group address

	Note other commands exist

## the s Temperature and Mode setting command

    Description        Length      Example Value
    =====================================================================
    Base String        6           000440000000
    RF Address         3           0FDAED
    Room nr            1           01
    Temperature  &     1           66
	Mode (similar to the L message)
	Date Until		   1           04  Note: Only in case of vacation mode setting. Otherwise this byte is omitted

	
	
### Temperature & mode

    01100110:                          66

The temperature indicates the configured temperature and the mode of a device. It can be decoded as following:

    hex:  |    66     |
    dual: | 0110 1100 |
            |||| ||||
            ||++-++++-- temperature: 10 1100 -> 38 = temp * 2
            ||                     (to get the temperature, the value must be divided by 2: 38/2 = 19)
            ||
            ||
            ++--------- mode: 00=auto/weekly program
                        01=manual
                        10=vacation
                        11=boost
					
In case of a vacation program

### Date Until

    0000000000:                             9d 0b

Date until indicates to which date the given temperature is set. It can be decoded as following:

               +-++++--------------- day: 1 1101 -> 29
               | ||||  
    dual: | 1001 1101 | 0000 1011 | (9D0B)
            |||          | | ||||
            |||          | +-++++--- year: 0 1011 -> 11 = year - 2000
            |||          |                 (to get the actual year, 2000 must be added to the value: 11+2000 = 2011)
            |||          |
            +++----------+---------- month: 1000 -> 8

In this example the temperature is set till Aug 29, 2011.

### Time Until


Time until indicates to which date the given temperature is set. In this example it is set to 2:00 (04 * 0,5 hours = 2:00)

## s command program setting

	s:AAQQAAAAD8OAAQJASUxuQMtNIE0gTSBNIA==

Again the parameter is base64 encoded.
w
Converted to hex, this is: 

    00 04 10 00 00 00 0f c3  80 01 02 40 49 4c 6e 40
    cb 4d 20 4d 20 4d 20 4d  02

It is decoded as following:

    Description        Length      Example Value
    =====================================================================
    Base String        6           000410000000
    RF Address         3           0FC380
    Room Nr            1           01
    Day of week        1           02
    Temp and Time      2           4049
    Temp and Time (2)  2           4c6e
    Temp and Time (3)  2           40cb
    Temp and Time (4)  2           4d20
    Temp and Time (5)  2           4d20
    Temp and Time (6)  2           4d20
    Temp and Time (7)  2           4d02

### Day of week

    hex:  |    02     |
    dual: | 0000 0010 |
                 ||||
                 |+++-- day: 000: saturday
                 |           001: sunday
                 |           010: monday
                 |           011: tuesday
                 |           100: wednesday
                 |           101: thursday
                 |           110: friday
                 |
                 +----- telegram: 1: set
                                  0: not set
                                  
The meaning of telegram is unclear at the moment.

### Temperature and Time

    hex:  |    40     |    49     |
    dual: | 0100 0000 | 0100 1001 |
            |||| ||||   |||| |||| 
            |||| |||+---++++-++++-- Time: 001001001: 06:05
            |||| |||
            |||| |||+-------------- Temperature: 0100000: 16
This 16 bit word contains the temperature on the 7 MSB and Time until that temperature is set on the 9 LSB.
Temperature value has to be divided by 2.

    20 (hex) =  32 (decimal) -> 32/2 = 16

Time is the value * 5 minutes since midnight.

    49 (hex) = 73 (decimal) -> 73*5 = 365 -> 6:05

## s command eco temperature  setting

	s:AAARAAAAD8OAACshPQkHGAM=

Again the parameter is base64 encoded.

Converted to hex, this is: 

    00 00 11 00 00 00 0f c3  80 00 2b 21 3d 09 07 18
    03

It is decoded as following:

    Description              Length      Example Value
    =====================================================================
    Base String              6           000011000000
    RF Address               3           0FC380
    Room Nr                  1           00
    Temperature Comfort      1           2b
    Temperature Eco          1           21
    Temperature Max          1           3d
    Temperature Min          1           09
    Temperature Offset       1           07
    Temperature Window Open  1           18
    Duration window open     1           03


### Temperature

Temperature comfort, temperature eco, temperature max, temperature min, temperature window open decode the same way.
To get the actual temperature, the value muast be divided by 2.

    2b (hex) = 43 (decimal) -> 43/2 = 21.5


### Temperature Offset

Temperature Offset is decoded as following:

    07 (hex) = 7 (decimal) -> 7/3 - 3.5 = 0

### Duration Window Open

Duration window open is simply the time in minutes

    03 (hex) = 3 (decimal) -> 3 (minutes)


## s command config valve functions
    
	s:AAQSAAAAD8OAATIM/wA=\r\n

Again the parameter is base64 encoded.

Converted to hex, this is: 

	00 04 12 00 00 00 0f c3  80 01 32 0c ff 00
  
It is decoded as following:

	Description        Length      Example Value
	=====================================================================
	Base String        6           000412000000
	RF Address         3           0FC380
	Room Nr            1           01
	Boost              1           32
	Decalcification    1           0C
	Valve Maximum      1           FF
	Valve Offset       1           00

### Boost

    hex:  |    32     |
    dual: | 0011 0010 |
            |||| ||||
            |||+-++++--- valve position: 10010 -> 18 = 18/20 = 90 % (20 is fully open)
            |||
            +++--------- boost duration: 001 -> 1 = duration / 5 minutes
                                       (to get the actual duration, the value must be multiplied by 5: 1 * 5 = 5 minutes)

### Decalcification

    hex:  |    0C     |
    dual: | 0000 1100 |
            |||| ||||
            |||+-++++--- hour: 01100 -> 12
            |||
            +++--------- day: 000: saturday
                              001: sunday
                              010: monday
                              011: tuesday
                              100: wednesday
                              101: thursday
                              110: friday

## s command add link partner
    
    s:AAAgAAAAD8NzAA/a7QE=\r\n

Again the parameter is base64 encoded.

Converted to hex, this is: 

    00 00 20 00 00 00 0f c3  73 00 0f da ed 01

It is decoded as following:

    Description        Length      Example Value
    =====================================================================
    Base String        6           000020000000
    RF Address         3           0FC373
    Room Nr            1           00
    RF Adress Partner  3           0FDAED
    Partner Type       1           01

### Partner Type

Partner type tells what device the partner is.

	Device			Type
	=====================================================================
	Heating Thermostat	1 
	Heating Thermostat Plus 2 
	Wall Mounted Thermostat 3 
	Shutter Contact		4
	Push Button		5


## s command remove link partner

    s:AAAhAAAAD8NzAA/a7QE=\r\n

Again the parameter is base64 encoded.

Converted to hex, this is: 

    00 00 21 00 00 00 0f c3  73 00 0f da ed 01

It is decoded as following:

    Description        Length      Example Value
    =====================================================================
    Base String        6           000021000000
    RF Address         3           0FC373
    Room Nr            1           00
    RF Adress Partner  3           0FDAED
    Partner Type       1           01

### Partner Type

Partner type tells what device the partner is.

	Device			Type
	=====================================================================
	Heating Thermostat	1 
	Heating Thermostat Plus 2 
	Wall Mounted Thermostat 3 
	Shutter Contact		4
	Push Button		5

## s command set group address

    s:AAAiAAAAD8OAAAE=\r\n

Again the parameter is base64 encoded.

Converted to hex, this is: 

    00 00 22 00 00 00 0f c3  80 00 01

It is decoded as following:

    Description        Length      Example Value
    =====================================================================
    Base String        6           000022000000
    RF Address         3           0FC380
    Not Used           1           00
    Room Nr            1           01

## s command remove group address

    s:AAAjAAAAD8OAAAE=\r\n

Again the parameter is base64 encoded.

Converted to hex, this is: 

    00 00 23 00 00 00 0f c3  80 00 01

It is decoded as following:

    Description        Length      Example Value
    =====================================================================
    Base String        6           000023000000
    RF Address         3           0FC380
    Not Used           1           00
    Room Nr            1           01


## The S Message

The incoming 'S' message response looks like this:

	S:00,0,31

This can be decoded as following

	Description        Length      Example Value
	=====================================================================
	Duty Cycle          2           00 
	Command Result      1           0
	Free Memory Slots   2           31

### Duty Cycle

868MHz radio comms is limited to 1% transmission, i.e. 36 seconds in each hour. The cube monitors this, and returns the hex representation of the permitted duty cycle as a percentage. When this percentage reaches 100% it will queue S commands in memory or reject additional commands.

### Command Result

	0:	command processed
	1:	command discarded

### Free Memory Slots

The hex representation of the free memory slots
