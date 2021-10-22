# Maretron_MPower

DC Load Controller Modules

* [Documentation](docs)
* [Contents of USB DISK shipped with CLMD12](https://thayermahan-my.sharepoint.com/:f:/p/jcorcoran/EhA2ZhF0i9JKsjyyqnFF2ZcBiykFItC3XT-6K0nYPWfogw?e=kfqcGf)
  This includes documentation, training materials, and installers (N2KView, N2KAnalyzer, N2KExtractor, DSM250 Emulator) 


## Wiring

To establish N2K comms you need to:
  * J3 - wire CAN-H, CAN-L, GND, 12VDC
  * J2 - pin 8 needs to connect to ground
  * Wire 6.5 - 32 VDC to the "+" lug

## NMEA 2000 Messags

From [CLMD12UM_1.8 Appendix A](docs/CLMD12UM_1.8.pdf).

__CLMD12 NMEA 2000® Periodic Data Transmitted PGNs__

* PGN 127500 (0x1F20C) – Load Controller Connection State/Control

  The CLMD12 uses this PGN to transmit the state of each of the breakers. A separate occurrence
  of this message will be transmitted for each breaker. The state of each breaker may be controlled
  by issuing a 126208 NMEA Command for this message addressed to this device.

  * Field 1: Sequence ID – This field is transmitted with a value of 255.
  * Field 2: Connection ID – This field identifies the output channel (breaker) whose status is being reported in this message.   
    The value of this field will be in the range of 0 (Breaker #1) through 11 (Breaker #12).
  * Field 3: State – This field indicates the state of the solid-state breaker.
  * Field 4: Status – This field indicates the status of the solid-state breaker.
  * Field 5: Operational Status & Control – This field is used to lock and unlock the solid-state breaker.
  * Field 6: PWM Duty Cycle – This field is used to control and report the PWM duty cycle of the
solid-state breaker.
  * Field 7: TimeON – This field is used to report the ON time if the solid-state breaker is running
under the control of a Flash Map.
  * Field 8: TimeOFF – This field is used to report the OFF time if the solid-state breaker is running
under the control of a Flash Map.

* PGN 127501 (0x1F20D) – Binary Status Report

  The CLMD12 uses this PGN to transmit the state of each of the breakers and connected switch
  inputs. The state of the breakers may be controlled by issuing a 126208 NMEA Command for
  this message addressed to this device.

  * Field 1: Indicator Bank Instance – This field identifies the particular switch bank to which this
PGN applies. Please refer to Section 3.5.1.2 for instructions on how to program the
value of this field.
  * Field  2: Indicator #1 – This field indicates the state of the solid-state breaker on output channel #1. The state will be one of the following values:
    * “OFF” – The breaker is open – no current is supplied to the load.
    * “ON” – The breaker is closed – current is supplied to the load.
    * “Error” – The breaker is open due to an error condition
  * Field 3: Indicator #2 – This field indicates the state of the solid-state breaker on output channel #2.
  * Field 4: Indicator #3 – This field indicates the state of the solid-state breaker on output channel #3.
  * Field 5: Indicator #4 – This field indicates the state of the solid-state breaker on output channel #4.
  * Field 6: Indicator #5 – This field indicates the state of the solid-state breaker on output channel #5.
  * Field 7: Indicator #6 – This field indicates the state of the solid-state breaker on output channel #6.
  * Field 8: Indicator #7 – This field indicates the state of the solid-state breaker on output channel #7.
  * Field 9: Indicator #8 – This field indicates the state of the solid-state breaker on output channel #8.
  * Field 10: Indicator #9 – This field indicates the state of the solid-state breaker on output channel #9.
  * Field 11: Indicator #10 – This field indicates the state of the solid-state breaker on output channel #10.
  * Field 12: Indicator #11 – This field indicates the state of the solid-state breaker on output channel #11.
  * Field 13: Indicator #12 – This field indicates the state of the solid-state breaker on output channel #12.
  * Field 14: Indicator #13 – This field indicates the state sensed by the digital input on input channel #1. The state will be one of the following values:
    * “OFF” – The digital input voltage level is outside the range(s) programmed for “ON” levels
    * “ON” – The digital input voltage level is inside the range(s) programmed for “ON” levels
      Please refer to section 3.5.4.3.1 for details
  * Field 15: Indicator #14 – This field indicates the state sensed by the digital input on input channel #2.
  * Field 16: Indicator #15 – This field indicates the state sensed by the digital input on input channel #3.
  * Field 17: Indicator #16 – This field indicates the state sensed by the digital input on input channel #4.
  * Field 18: Indicator #17 – This field indicates the state sensed by the digital input on input channel #5.
  * Field 19: Indicator #18 – This field indicates the state sensed by the digital input on input channel #6.
  * Field 20: Indicator #19 – This field indicates the state sensed by the digital input on input channel #7.
