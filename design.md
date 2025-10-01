# Osprey RF Input FMC Board Design

## RF Input section

- U1 - Splits RF Input for level monitoring
- R1 - the 475R Resistor is specified in the datasheet for TCP-2-33W
- C1 - A DC Blocking Cap and creates a high pass filter. The resistive input impedance is largely dominated by R2 creating a high pass cut off of ~3MHz
- C2&C3 - High Pass Filter input into the Clock Distribution IC

## Clock Divider Chip - LMK01801 [datasheet](https://www.ti.com/product/LMK01801)
Internal Distribution [image](https://www.ti.com/ds_dgm/images/fbd_snas573d.gif)

Power Supply Pins
| Pin | Pin Name | Description |
|:---|:---------------------|:----------------------------------------------|
|  6 | Vcc1_CLKout 0_1_2_3 | Power supply for clock outputs 0, 1, 2, and 3  |
| 15 | Vcc2_CLKin0         | Power supply for clock input 0                 |
| 20 | Vcc3_CLKout 4_5_6_7 | Power supply for clock outputs 4, 5, 6, and 7  |
| 25 | Vcc4_Bias           | Power supply for Bias                          |
| 32 | Vcc5_CLKout8_9_10_1 | Power supply for clock outputs 8, 9, 10, and 11|
| 37 | Vcc6_CLKin1         | Power supply for clock input 1                 |
| 41 | Vcc7_CLKout 12_13   | Power supply for clock outputs 12, and 13      |
| 46 | Vcc8_DIG            | Power supply for digital                       |

Zest Grouped the rails as follows:

- VCC1 & VCC3 | This powers outputs 0-7 (Bank A outputs)
- VCC2, VCC4, VCC6, & VCC8 This powers inputs,bias, and digital core
- VCC5 & VCC7 | This powers outputs 8-13 (Bank B outputs)

This separation makes sense and we will keep this grouping.

#### Bias
See section 12.1.1.5 for the inclusion of the 1uF cap between pin25 (Vcc4_Bias) & pin26 (Bias)
I'm running the RF input into both clock banks of the chip. We may want to limit this as it could reduce ICs power draw

#### 12.1.1.2 Unused clock outputs
Leave unused clock outputs floating and powered down.
12.1.1.3 Unused clock inputs
Unused clock inputs can be left floating.

#### 9.4.1 Programmable Mode
When the EN_PIN_CTRL pin is floating (default by internal pull-up/pull-down) then programming is via
MICROWIRE
We want to be in MICROWIRE mode so leave floating

#### 12.1.1.6 In MICROWIRE Mode pins 12,40
SYNC0 and SYNC1 have an internal pullup and may be left as a no-connect if external SYNC is not required.
MICROWIRE SYNC may still be used in this condition.

#### 12.1.1.4 Unused GPIO (CLKoutTYPE_X) pins 11,12,40
Unused GPIO pins can be left floating. Alternatively, unused GPIO pins can individually be pulled to GND


**Zest has pins 12,40 attached to ground; while pin 11 is floating
Maybe we should leave these all floating?**

### Clock Output Connections
For now I am using the same outputs used by Zest going to the similar FMC pins to Zest.
**We should decide this**



## RF Input monitoring

### ADL5501 [datasheet](https://www.analog.com/media/en/technical-documentation/data-sheets/ADL5501.pdf)
The RF is split off and feed into the ADL5501
This is linear device
According to pin definitions: Output is low pass filtered with ~8kohm internal resistance to achieve ~100kHz output.
Working backwards this give the internal capacitance value of ~200pF in this filter.
Zest used a 10nF external cap in parallel with the internal cap lowering the filter further to ~2kHz
This is kept in the design as C6 & C23.

### MCP3202 [datasheet](https://ww1.microchip.com/downloads/aemDocuments/documents/APID/ProductDocuments/DataSheets/21034F.pdf)
With the 2kHz bandwidth chosen the design space is open to any ADC capable of measure this.
I chose a simple SPI Dual input ADC for this Role.
This ADC uses its power supply as an external reference.
So this should receive clean 3.3V Power if possible.
This IC also draws a max of only 550uA while operating.

### LDO [datasheet](https://www.ti.com/lit/ds/symlink/tps7b91.pdf)
With the low current draw of the ADC in mind, its possible to run this safely off a 3.3V LDO coming off the 12V input.
Although quite inefficient thermally This provides a clean 3.3V reference for the ADC
A TPS7B91 was chosen for simplicity for this application.
This LDO has low quiescent current of 1.5uA
The Data sheet mentions (7.2.2.2) to have a 1uF+ output capacitance for improved stability


## Digital Inputs

We would like to take digital inputs at up to 5V level and translate them into the 2.5V level for the FPGA.
There are many parts capable of this translation with a variety of tradeoffs in speed/complexity/input-type(hysteresis)/Logic transition levels

The 6 bit SN74LV6T17-EP was a suggested part.
This offers a fairly simple schmitt trigger input buffer that can be operated at the 2.5V FPGA logic level, but has 5V tolerant inputs.
This part is part of TI's LVxT family

Another part I found in the same family is the SN74LV8T9541-EP [datasheet](https://www.ti.com/lit/ds/symlink/sn74lv8t9541-ep.pdf)
This is another EP (HiRel Enhanced Product) part but with 8 bits in one chip.
This also has some output enable pins that 'can' be used, but I'm not sure they are needed.
This part has slightly better max propagation delay than the SN74LV6T17-EP.


## Level Shifter
This uses the same parts from Zest

## EEPROM
This uses the same setup and Quartz

**Do we want a WP pin header? Maybe make this just pads?**

## FMC
The current pin assignments are just placeholders

Where do we want to put these RF clocks?:
- On the Clock signals for multi-gigabit transceiver data pairs (GBTCLK0_M2C)
- On the multi-gigabit transceiver data pairs (DP0_M2C)
- On the 2 user clocks, differential pairs (CLK0_M2C)
- ON CC pins?
  
How many do we want to put in? 

We could put a given external clock into multiple pins with the LMK chip


