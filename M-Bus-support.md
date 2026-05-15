# DCS M-Bus support

DCS supports M-Bus meters connected to an IDC’s RS232 port, normally via an Elvaco unit. 
The IDC doesn’t have any knowledge of the M-Bus protocol and simply passes it transparently 
to the serial port (in the same way it handles intelligent electricity meters).

## Meter Types

A meter type is required with an appropriate M-bus plugin that handles the M-Bus protocol and maps the VIF values to DCS register addresses. 

There is a generic M-Bus plugin that allows for meter types with any of the following register types:

|                      | Register Address | Unit
-----------------------|------------------|-----
Energy                 | 1                | kWh
Power                  | 2                | kW
Flow temperature       | 3                | C
Return temperature     | 4                | C
Temperature difference | 5                | K
Volume                 | 6                | m3
Flow rate              | 7                | m3/hr
Cooling Energy         | 9                | kWh
Heating Energy         | 10               | kWh
Voltage L1-N           | 12               | V
Voltage L2-N           | 13               | V
Voltage L3-N           | 14               | V
Current                | 15               | V
Current L1             | 16               | A
Current L2             | 17               | A
Current L3             | 18               | A
Power Factor           | 19               |  
Frequency              | 20               | Hz

For meters that use extended VIFs with manufacturer specific extensions to the 
standard M-Bus VIFs a specific meter plugin will be need to be developed by Coherent Research.

## Meter setup

When defining a meter the Connection Method will be set to IDC and the remote address will have the form 00:17:BF:XX:XX:XX-COMB.

To use M-Bus primary addressing the Device ID must be set to the M-Bus address of the meter (if no Device ID is set 0 will be used). This value is between 0 and 250. To use M-Bus secondary addressing the Device ID must be set to the 8 digit meter identity. This may or may not be the same as the meter serial number.

The baud rate, data bits, parity and stop bits must be set.

## Polling

Since these meters don’t have internal survey data like intelligent electricity meters DCS is in 
effect polling the current reading each half hour. 
Since DCS is doing this (cf. IDCs polling Modbus meters) the precise time is indeterminant (depends on how many IDCs you have etc). 
Therefore DCS treats these reading a bit differently to other cases: when it polls a reading it sets the timestamp to the end of the current period. 
If there are any readings missing, e.g. if the IDC was offline, DCS fills these readings in as normal readings. 



