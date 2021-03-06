# Maretron_MPower (CLMD)

DC Load Controller Module

* [Documentation](docs)
* [Contents of USB DISK shipped with CLMD12](https://thayermahan-my.sharepoint.com/:f:/p/jcorcoran/EhA2ZhF0i9JKsjyyqnFF2ZcBiykFItC3XT-6K0nYPWfogw?e=kfqcGf)
  This includes documentation, training materials, and installers (N2KView, N2KAnalyzer, N2KExtractor, DSM250 Emulator) 
* [Photos of device internals (teardown)](https://thayermahan-my.sharepoint.com/:f:/p/jcorcoran/Ek5BNGozaKVOlN0toJKLDpABl3fM2MS0jYa-kdxh5eg-kQ?e=UJaBeo)

[[_TOC_]]

## Configuration

  * Plug in Maretron USB100 gateway
  * Open [N2KAnalyzer](https://www.maretron.com/products/N2KAnalyzer.php)
  * Select the CLMD12 entry  
    ![n2kanalyzer gui](img/N2KAnalyzer.png)

  * Right click the CLMD12 entry, select `Configure Device`

  * From the `General` tab, configure output channels:
    Of particular note, current rating does not default to the max per channel, and `Inrush delay` defaults to 200 ms,
    which could high power loads or brownouts for some loads. 

    ![configure output channels](img/output_channel_settings.png)

  * From the `Discrete I/O` tab, configure how and if the input channls should control the output channels:

    By default "on" states of the inputs toggle the output channels on/off.

    ![discrete i/o settings](img/io_settings.png)

  * From the `Inputs` tab, configure the `OnLevel` for each digital input channel.

    Note by default the "on" state corresponds to the input being pulled either high __or__ low. Off is floating. 

    ![inputs tab](img/inputs.png)

  * From the `Advanced` tab, set as follows:  
    ![n2kanalyzer configuration > advanced](img/n2k_advanced.png)


## Wiring

To establish N2K comms you need to:
  * J3 - wire CAN-H, CAN-L, GND, 12VDC
  * J2 - pin 8 needs to connect to ground
  * Wire 6.5 - 32 VDC to the "+" lug

## NMEA 2000 Messags

### DC Voltage/Current Status message (PGN 127751)

Sent periodically every 15 seconds.

Contains the voltage and current status of each output channel.

```
  can0  19F30790   [8]  FF 00 71 00 32 00 00 FF
  can0  19F30790   [8]  FF 01 71 00 00 00 00 FF
  can0  19F30790   [8]  FF 02 00 00 00 00 00 FF
  can0  19F30790   [8]  FF 03 00 00 00 00 00 FF
  can0  19F30790   [8]  FF 04 00 00 00 00 00 FF
  can0  19F30790   [8]  FF 05 00 00 00 00 00 FF
  can0  19F30790   [8]  FF 06 00 00 00 00 00 FF
  can0  19F30790   [8]  FF 07 00 00 00 00 00 FF
  can0  19F30790   [8]  FF 08 00 00 00 00 00 FF
  can0  19F30790   [8]  FF 09 00 00 00 00 00 FF
  can0  19F30790   [8]  FF 0A 00 00 00 00 00 FF
  can0  19F30790   [8]  FF 0B 00 00 00 00 00 FF

  can0  19F30790   [8]  FF 00 71 00 BC 00 00 FF
  can0  19F30790   [8]  FF 01 71 00 00 00 00 FF
  can0  19F30790   [8]  FF 02 00 00 00 00 00 FF
  can0  19F30790   [8]  FF 03 00 00 00 00 00 FF
  can0  19F30790   [8]  FF 04 00 00 00 00 00 FF
  can0  19F30790   [8]  FF 05 00 00 00 00 00 FF
  can0  19F30790   [8]  FF 06 00 00 00 00 00 FF
  can0  19F30790   [8]  FF 07 00 00 00 00 00 FF
  can0  19F30790   [8]  FF 08 00 00 00 00 00 FF
  can0  19F30790   [8]  FF 09 00 00 00 00 00 FF
  can0  19F30790   [8]  FF 0A 00 00 00 00 00 FF
  can0  19F30790   [8]  FF 0B 00 00 00 00 00 FF
```

In the above data capture we can see that output channels 1 and 2 are turned on (non-zero voltage output).
Channel 1 is sourcing current, while channel two is sourcing not (zero draw). All other channels are off.

#### Example decomp:

```
can0  19F30790   [8]  FF 00 71 00 BF 01 00 FF
```

__Contents of the CAN message header:__

| Field name | length (bits) | Value | Notes |
| :--------- | :-----------: | :---- | :---- |
| Priority | 3 | 0b110 | Defined by the source device.
| PGN | 18 | 0x1F307 |
| Source address | 8 | 0x90 | Unique ID. _Note: not the same as the device instance in photo above_

__Contents of the data frame:__

| Field name    | length (bits) | Value | Notes |
| :------------ | :-----------: | :---- | :---- |
| Sequnce ID    | 8 | 0xFF | Hard coded per [Maretron documentation](https://www.maretron.com/support/manuals/CLMD16UM_1.9.pdf)
| Connection ID | 8 | 0x00 | The output channel 0x0 - 0xB -> Output 1 - 12 
| DC Voltage    | 16 | 0x7100 = 113 <br> LSB = 0.1V <br> 113 * 0.1 = 11.3V | Byte swap. Note this will report a voltage when the breaker is tripped.
| DC Current    | 16 | 0xBF01 = 447 <br> LSB = 0.01A <br> 447 * 0.01 = 4.47A | Byte swap. 
| Unknown       | 8 | 0x00 | Unused / unknown
| Reserved      | 8 | 0xFF | Hard coded 0xFF per Maretron documentation


### Circuit Breaker State status message (PGN ):

Sent once every 15 seconds, and in response to state changes.

Each circuit breaker state is encoded as a two bit value:

| Bit 0 | Bit 1 | Value | Description |
| :---: | :---: | :---- | :---------- |
| 1 | 0 | 0x1 | Breaker `ON`
| 0 | 1 | 0x2 | Breaker `Tripped`
| 0 | 0 | 0x0 | Breaker `OFF` and not `Tripped`
| 1 | 1 | 0x3 | Breaker `ON` and `Tripped`


Each input state is encoded as a two bit value:
| Bit 0 | Bit 1 | Value | Description |
| :---: | :---: | :---- | :---------- |
| 0 | 0 | 0x0 | Input off (per device configuration, by default this is a floating wire)
| 1 | 0 | 0x1 | Input on (per device configuration, by default this is low OR pulled high)
| 0 | 1 | 0x2 | Unused
| 1 | 1 | 0x3 | Unused

#### Example decomp:

```
  can0  0DF20D90   [8]  20 52 00 05 05 C1 FF FF
```

__Contents of the data frame:__

| Field number  | Field name    | length (bits) | Value | Notes |
| :-----------: | :------------ | :-----------: | :---- | :---- |
| 1 | Bank instance | 8 | 0x20 = 32 | Field value as programmed on CLMD (above)
| 2 | Breaker channel #1 | 2 | 0x2 | Breaker on and tripped
| 3 | Breaker channel #2 | 2 | 0x0 | Breaker off
| 4 | Breaker channel #3 | 2 | 0x1 | Breaker on
| 5 | Breaker channel #4 | 2 | 0x1 | Breaker on
| 6 | Breaker channel #5 | 2 | 0x0 | Breaker off
| 7 | Breaker channel #6 | 2 | 0x0 | Breaker off
| 8 | Breaker channel #7 | 2 | 0x0 | Breaker off
| 9 | Breaker channel #8 | 2 | 0x0 | Breaker off
| 10 | Breaker channel #9 | 2 | 0x1 | Breaker on
| 11 | Breaker channel #10 | 2 | 0x1 | Breaker on
| 12 | Breaker channel #11 | 2 | 0x0 | Breaker off
| 13 | Breaker channel #12 | 2 | 0x0 | Breaker off
| 14 | Input channel #1 | 2 | 0x1 | Input on
| 15 | Input channel #2 | 2 | 0x1 | Input on
| 16 | Input channel #3 | 2 | 0x0 | Input off
| 17 | Input channel #4 | 2 | 0x0 | Input off
| 18 | Input channel #5 | 2 | 0x1 | Input on
| 19 | Input channel #6 | 2 | 0x0 | Input off
| 20 | Input channel #7 | 2 | 0x0 | Input off
| N/A | Reserved | 2 | 0x3 | upper-most bits of byte 6
| N/A | Reserved | 16 |0xFFFF |

Field numbers above correspond to commanded paramter numbers used in `126208`

### Command messages (PGN 126208)

The CLMD12 allows for control of circuit breaker states through PGN 126208 (Request/Command/Acknowledge Group Function).

This PGN message has a number of different uses which are conveniently (unintentionally?) publicly described
in this pdf from the NMEA organization: https://www.nmea.org/Assets/20140109%20nmea-2000-corrigendum-tc201401031%20pgn%20126208.pdf


According to the N2K documentation, there are seven different ways this PGN can be used/formatted... which is kind of
obnoxious in my (James) oppinion. These seven different message intrpretations are distinguished by the first byte of the message:

  * 0 = Request Message,
  * 1 = Command Message,
  * 2 = Acknowledge Message,
  * 3 = Read Fields,
  * 4 = Read Fields Reply,
  * 5 = Write Fields,
  * 6 = Write Fields Reply

The Maretron docs indicate a `command message` should be sent, implying the first byte == `1`.

#### Command Group Function message - PGN 126208 (0x1ED00)

The format of this message is as follows [per N2K documentation](https://www.nmea.org/Assets/20140109%20nmea-2000-corrigendum-tc201401031%20pgn%20126208.pdf).

The payload data of this message identifies the PGN for which parameters will be changed, as kind of
a generic interface for manipulating settings of arbitrary messages.  
In our case the PGN of interest for the circuit breakers is 127501 (Binary Status Report)
per the maretron documents (See notes in section below).  

Following the PGN identification, we also define the quantity of filed id/field value pairs. i.e. the
data in the _commanded_ PGN that we want to change.
As such, this is a variable length message. FIeld ID lengths are fixed at 8 bits. The lengths of the
associated `Values` is defined by the message definition for the PGN we are _commanding_.
Individual fields in the data frame are padded to 1 byte boundaries (per NMEA spec). 

The contents of this message is longer than 8bytes, and as a result must span multiple packets.
The provision for this in NMEA2K is called `fast packet` and is described in more detail
[here](https://gitlab.tmpriv.com/thayermahan/documentation/can_notes).


Data frame contents:

| Field name | length (bits) | Value | Notes |
| :--------- | :-----------: | :---- | :---- |
| Fast packet header | 8 | 0xE0 | This message will end up spanning multiple packets (longer than 8 bytes) this header flags that a sequence of messages will follow.
| Message Length | 8 | 0x0A | The number of bytes that follow, the length of data fields to that follow, this is only present in the first fast packet.
| Command group function code | 8 | 0x01 | Identify this is a `Command Message`
| Commanded PGN | 24 | 0x1F20D (127501) <br> byte swapping leads to 0x0DF201 | 
| Priority Setting | 4 | 0x8 | Allows changing the priority of the Commanded PGN <br> 0x8 = Do not change priority <br> note this is the lower nibble of the byte
| Reserved | 4 | 0xF | Not this is the upper nibble of the byte
| Number of Pairs of Commanded Parameters to follow | 8 | 0x2 | 
| - | - | - | We've reached the data frame 8byte limit. Start next packet
| Fast packet header | 8 0xE1 | indicates the next frame of the `fast packet` 
| Field number of first commanded parameter | 8 | 0x01 | Bank instance field in PGN 127501
| Value field| 8 <br> (2bit field padded to byte boundary) | (default 0x20) | Bank instance field value as programmed on CLMD (above), as reported in PGN 127501. <br> default: 32 = 0x20  
| Parameter field | 8 | 0x02 - 0x0D | which breaker to command, 1 - 12 => 0x2 - 0x0D
| Value Field | 8 <br> (2bit field padded to byte boundary) | 0x0 or 0x1 | breaker state, `0x0` = `OFF`, `0x1` = `ON`


Examples:
  * turn on channel 1:  
    ```
    cansend can0 0DED9050#E00A010DF201F802; cansend can0 0DED9050#E101200201FFFFFF
    ```
  * turn off channel 1:  
    ```
    cansend can0 0DED9050#E00A010DF201F802; cansend can0 0DED9050#E101200200FFFFFF
    ```
  * turn on channel 2:  
    ```
    cansend can0 0DED9050#E00A010DF201F802; cansend can0 0DED9050#E101200301FFFFFF
    ```
  * turn off channel 2:  
    ```
    cansend can0 0DED9050#E00A010DF201F802; cansend can0 0DED9050#E101200300FFFFFF
    ```


### CLMD12 NMEA 2000 Periodic Data Transmitted PGNs

From [CLMD12UM_1.8 Appendix A](docs/CLMD12UM_1.8.pdf).


* PGN 127500 (0x1F20C) ??? Load Controller Connection State/Control

  The CLMD12 uses this PGN to transmit the state of each of the breakers. A separate occurrence
  of this message will be transmitted for each breaker. The state of each breaker may be controlled
  by issuing a 126208 NMEA Command for this message addressed to this device.

  * Field 1: Sequence ID ??? This field is transmitted with a value of 255.
  * Field 2: Connection ID ??? This field identifies the output channel (breaker) whose status is being reported in this message.   
    The value of this field will be in the range of 0 (Breaker #1) through 11 (Breaker #12).
  * Field 3: State ??? This field indicates the state of the solid-state breaker.
  * Field 4: Status ??? This field indicates the status of the solid-state breaker.
  * Field 5: Operational Status & Control ??? This field is used to lock and unlock the solid-state breaker.
  * Field 6: PWM Duty Cycle ??? This field is used to control and report the PWM duty cycle of the
solid-state breaker.
  * Field 7: TimeON ??? This field is used to report the ON time if the solid-state breaker is running
under the control of a Flash Map.
  * Field 8: TimeOFF ??? This field is used to report the OFF time if the solid-state breaker is running
under the control of a Flash Map.

* PGN 127501 (0x1F20D) ??? Binary Status Report

  The CLMD12 uses this PGN to transmit the state of each of the breakers and connected switch
  inputs. The state of the breakers may be controlled by issuing a 126208 NMEA Command for
  this message addressed to this device.

  * Field 1: Indicator Bank Instance ??? This field identifies the particular switch bank to which this
PGN applies. Please refer to Section 3.5.1.2 for instructions on how to program the
value of this field.
  * Field  2: Indicator #1 ??? This field indicates the state of the solid-state breaker on output channel #1. The state will be one of the following values:
    * ???OFF??? ??? The breaker is open ??? no current is supplied to the load.
    * ???ON??? ??? The breaker is closed ??? current is supplied to the load.
    * ???Error??? ??? The breaker is open due to an error condition
  * Field 3: Indicator #2 ??? This field indicates the state of the solid-state breaker on output channel #2.
  * Field 4: Indicator #3 ??? This field indicates the state of the solid-state breaker on output channel #3.
  * Field 5: Indicator #4 ??? This field indicates the state of the solid-state breaker on output channel #4.
  * Field 6: Indicator #5 ??? This field indicates the state of the solid-state breaker on output channel #5.
  * Field 7: Indicator #6 ??? This field indicates the state of the solid-state breaker on output channel #6.
  * Field 8: Indicator #7 ??? This field indicates the state of the solid-state breaker on output channel #7.
  * Field 9: Indicator #8 ??? This field indicates the state of the solid-state breaker on output channel #8.
  * Field 10: Indicator #9 ??? This field indicates the state of the solid-state breaker on output channel #9.
  * Field 11: Indicator #10 ??? This field indicates the state of the solid-state breaker on output channel #10.
  * Field 12: Indicator #11 ??? This field indicates the state of the solid-state breaker on output channel #11.
  * Field 13: Indicator #12 ??? This field indicates the state of the solid-state breaker on output channel #12.
  * Field 14: Indicator #13 ??? This field indicates the state sensed by the digital input on input channel #1. The state will be one of the following values:
    * ???OFF??? ??? The digital input voltage level is outside the range(s) programmed for ???ON??? levels
    * ???ON??? ??? The digital input voltage level is inside the range(s) programmed for ???ON??? levels
      Please refer to section 3.5.4.3.1 for details
  * Field 15: Indicator #14 ??? This field indicates the state sensed by the digital input on input channel #2.
  * Field 16: Indicator #15 ??? This field indicates the state sensed by the digital input on input channel #3.
  * Field 17: Indicator #16 ??? This field indicates the state sensed by the digital input on input channel #4.
  * Field 18: Indicator #17 ??? This field indicates the state sensed by the digital input on input channel #5.
  * Field 19: Indicator #18 ??? This field indicates the state sensed by the digital input on input channel #6.
  * Field 20: Indicator #19 ??? This field indicates the state sensed by the digital input on input channel #7.
