# Osprey RF Input FMC Board Design
## RF Input section

- U1 - Splits RF Input for level monitoring
- R1 - the 475R Resistor is specified in the datasheet for TCP-2-33W
- C1 - A DC Blocking Cap and creates a high pass filter. The resistive input impedance is largely dominated by R2 creating a high pass cut off of ~3MHz
- C2&C3 - High Pass Filter input into the Clock Distribution IC

## Clock Divider Chip - LMK01801 [datasheet](https://www.ti.com/product/LMK01801)
Internal Distribution [image](https://www.ti.com/ds_dgm/images/fbd_snas573d.gif)

Power Supply Pins


| Pin | Pin Name            | Description                                     |
| :-- | :------------------ | :---------------------------------------------- |
| 6   | Vcc1_CLKout 0_1_2_3 | Power supply for clock outputs 0, 1, 2, and 3   |
| 15  | Vcc2_CLKin0         | Power supply for clock input 0                  |
| 20  | Vcc3_CLKout 4_5_6_7 | Power supply for clock outputs 4, 5, 6, and 7   |
| 25  | Vcc4_Bias           | Power supply for Bias                           |
| 32  | Vcc5_CLKout8_9_10_1 | Power supply for clock outputs 8, 9, 10, and 11 |
| 37  | Vcc6_CLKin1         | Power supply for clock input 1                  |
| 41  | Vcc7_CLKout 12_13   | Power supply for clock outputs 12, and 13       |
| 46  | Vcc8_DIG            | Power supply for digital                        |

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

## ADC Choice
With the 2kHz bandwidth chosen the design space is open to any ADC capable of measure this.
I chose a simple SPI Dual input ADC for this Role.
We want something with few external parts and that can run it's digital IO at 2.5V to avoid level shifters

Currently in the design I have the ADS7253. This is a 12 bit ADC that is pin compatible with a 14&16bit version. While I think 12 bits is enough for the intended use, have a drop-in replacement with high resolution may be nice for particular community applications.

### TI Products that meet these requirements
| Product or Part number                              | Resolution (Bits) | PDF data sheet                                          | Sample rate (max) (ksps) | Number of input channels | Interface type | Architecture | Input type                       | Multichannel configuration | Rating     | Price\|Quantity (USD) | TI functional safety category | Reference mode    | Package type | Features                   | Power consumption (typ) (mW) | Analog supply voltage (min) (V) | Analog supply voltage (max) (V) | Digital supply (min) (V) | Digital supply (max) (V) |
| --------------------------------------------------- | ----------------- | ------------------------------------------------------- | ------------------------ | ------------------------ | -------------- | ------------ | -------------------------------- | -------------------------- | ---------- | --------------------- | ----------------------------- | ----------------- | ------------ | -------------------------- | ---------------------------- | ------------------------------- | ------------------------------- | ------------------------ | ------------------------ |
| [ADS7253](https://www.ti.com/product/ADS7253)       | 12                | [PDF data sheet](https://www.ti.com/lit/gpn/ADS7253)    | 1000                     | 2                        | SPI            | SAR          | Pseudo-Differential,Single-ended | Simultaneous Sampling      | Catalog    | 3.252                 |                               | External,Internal | TSSOP,WQFN   | Small Size                 | 45                           | 4.5                             | 5.5                             | 1.65                     | 5.5                      |
| [ADS7230](https://www.ti.com/product/ADS7230)       | 12                | [PDF data sheet](https://www.ti.com/lit/gpn/ADS7230)    | 1000                     | 2                        | SPI            | SAR          | Single-ended                     | Multiplexed                | Catalog    | 3.63                  |                               | External          | TSSOP,VQFN   | Daisy-Chainable,Oscillator | 13.7                         | 2.7                             | 5.5                             | 1.65                     | 5.5                      |
| [ADS7947](https://www.ti.com/product/ADS7947)       | 12                | [PDF data sheet](https://www.ti.com/lit/gpn/ADS7947)    | 2000                     | 2                        | SPI            | SAR          | Pseudo-Differential,Single-ended | Multiplexed                | Catalog    | 3.96                  |                               | External          | WQFN         | Small Size                 | 7.5                          | 2.7                             | 5.5                             | 1.65                     | 5.5                      |
| [ADS7946](https://www.ti.com/product/ADS7946)       | 14                | [PDF data sheet](https://www.ti.com/lit/gpn/ADS7946)    | 2000                     | 2                        | SPI            | SAR          | Pseudo-Differential,Single-ended | Multiplexed                | Catalog    | 4.62                  |                               | External          | WQFN         | Small Size                 | 10.5                         | 2.7                             | 5.25                            | 1.65                     | 5.25                     |
| [ADS7280](https://www.ti.com/product/ADS7280)       | 14                | [PDF data sheet](https://www.ti.com/lit/gpn/ADS7280)    | 1000                     | 2                        | SPI            | SAR          | Single-ended                     | Multiplexed                | Catalog    | 5.544                 |                               | External          | TSSOP,VQFN   | Daisy-Chainable,Oscillator | 13.7                         | 2.7                             | 5.5                             | 1.65                     | 5.5                      |
| [ADS7853](https://www.ti.com/product/ADS7853)       | 14                | [PDF data sheet](https://www.ti.com/lit/gpn/ADS7853)    | 1000                     | 2                        | SPI            | SAR          | Pseudo-Differential,Single-ended | Simultaneous Sampling      | Catalog    | 6.3                   |                               | External,Internal | TSSOP,WQFN   | Small Size                 | 45                           | 4.5                             | 5.5                             | 1.65                     | 5.5                      |
| [ADS8328](https://www.ti.com/product/ADS8328)       | 16                | [PDF data sheet](https://www.ti.com/lit/gpn/ADS8328)    | 500                      | 2                        | SPI            | SAR          | Single-ended                     | Multiplexed                | Catalog    | 6.97                  |                               | External          | TSSOP,VQFN   | Daisy-Chainable,Oscillator | 10.6                         | 2.7                             | 5.5                             | 1.65                     | 5.5                      |
| [ADS8330](https://www.ti.com/product/ADS8330)       | 16                | [PDF data sheet](https://www.ti.com/lit/gpn/ADS8330)    | 1000                     | 2                        | SPI            | SAR          | Single-ended                     | Multiplexed                | Catalog    | 8.828                 |                               | External          | TSSOP,VQFN   | Daisy-Chainable,Oscillator | 13.2                         | 2.7                             | 5.5                             | 1.65                     | 5.5                      |
| [ADS8353](https://www.ti.com/product/ADS8353)       | 16                | [PDF data sheet](https://www.ti.com/lit/gpn/ADS8353)    | 600                      | 2                        | SPI            | SAR          | Pseudo-Differential,Single-ended | Simultaneous Sampling      | Catalog    | 10.56                 |                               | External,Internal | TSSOP,WQFN   | Small Size                 | 45                           | 4.5                             | 5.5                             | 1.65                     | 5.5                      |
| [ADS8353-Q1](https://www.ti.com/product/ADS8353-Q1) | 16                | [PDF data sheet](https://www.ti.com/lit/gpn/ADS8353-Q1) | 600                      | 2                        | SPI            | SAR          | Pseudo-Differential,Single-ended | Simultaneous Sampling      | Automotive | 12.461                | Functional Safety-Capable     | External,Internal | TSSOP        |                            | 45                           | 4.5                             | 5.5                             | 1.65                     | 5.5                      |
| [ADS8355](https://www.ti.com/product/ADS8355)       | 16                | [PDF data sheet](https://www.ti.com/lit/gpn/ADS8355)    | 1000                     | 2                        | SPI            | SAR          | Pseudo-Differential,Single-ended | Simultaneous Sampling      | Catalog    | 16.445                |                               | External,Internal | WQFN         | Small Size                 | 60                           | 4.5                             | 5.5                             | 1.65                     | 5.5                      |

### ADS7253[datasheet]()
This ADC has an internal reference 
So this should receive clean 3.3V Power if possible.
This IC also draws a max of 10mA while operating.

### LDO [datasheet](https://www.ti.com/lit/ds/symlink/tps7b91.pdf)
With the low current draw of the ADC in mind, its possible to run this safely off a LDO coming off the 12V input.
Although quite inefficient thermally This provides a clean reference for the ADC
A TPS7B91 was chosen for simplicity for this application.
This LDO has low quiescent current of 1.5uA and can provide 150mA output.
The Data sheet mentions (7.2.2.2) to have a 1uF+ output capacitance for improved stability
This comes in 3.3V and 5V fixed output versions, covering our ADC choices


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

We found the SN74LV6T17-EP has poor speed, so I looked  for another part in TIs library

### Part Comparison
#### TI Voltage translators & level shifters
- **AVC** Family - Requires 2 Supply Voltages
- **TXV** Family - Requires 2 Supply Voltages
- **AXC** Family - Requires 2 Supply Voltages
- **LXC** Family - Requires 2 Supply Voltages
-  **LVC** Family - Requires 2 Supply Voltages
- **ALVC** Family - Requires 2 Supply Voltages
-  **TX** Family - Requires 2 Supply Voltages
- **LSF** Family - While Fast, its bidirectional which could be problematic
- **AUP1T** Family - While works for speed it can only go up to 3.6V inputs 
- **LVxT** Family -See table below

#### Propagation Delay (A → Y) at VCC = 2.5 V, TA = 25 °C

| Part number     | # Bits | Output type                              | Inverted? | CL (pF) | A→Y Low→High (Max ns) | A→Y High→Low (Max ns) | Max Switching Speed (Mhz) | Datasheet                                                                |
| --------------- | :----: | ---------------------------------------- | :-------: | :-----: | :-------------------: | :-------------------: | :-----------------------: | ------------------------------------------------------------------------ |
| SN74LV6T07      |   6    | Open-drain                               |    No     |   15    |         15.9          |         15.9          |           31.45           | [TI SN74LV6T07](https://www.ti.com/lit/gpn/sn74lv6t07)                   |
| SN74LV6T07      |   6    | Open-drain                               |    No     |   50    |         18.8          |         18.8          |           26.6            | [TI SN74LV6T07](https://www.ti.com/lit/gpn/sn74lv6t07)                   |
| SN74LV6T14      |   6    | Push-pull (Schmitt inverter)             |    Yes    |   15    |         11.0          |         11.0          |           45.45           | [TI SN74LV6T14](https://www.ti.com/lit/gpn/sn74lv6t14)                   |
| SN74LV6T14      |   6    | Push-pull (Schmitt inverter)             |    Yes    |   50    |         13.2          |         13.2          |           37.88           | [TI SN74LV6T14](https://www.ti.com/lit/gpn/sn74lv6t14)                   |
| SN74LV6T17      |   6    | Push-pull (Schmitt buffer)               |    No     |   15    |         11.0          |         11.0          |           45.45           | [TI SN74LV6T17](https://www.ti.com/lit/gpn/sn74lv6t17)                   |
| SN74LV6T17      |   6    | Push-pull (Schmitt buffer)               |    No     |   50    |         13.2          |         13.2          |           37.88           | [TI SN74LV6T17](https://www.ti.com/lit/gpn/sn74lv6t17)                   |
| SN74LV1T04      |   1    | Push-pull (inverter)                     |    Yes    |   15    |          7.0          |          7.0          |           71.43           | [TI SN74LV1T04](https://www.ti.com/lit/gpn/sn74lv1t04)                   |
| SN74LV1T04      |   1    | Push-pull (inverter)                     |    Yes    |   30    |          7.5          |          7.5          |           66.67           | [TI SN74LV1T04](https://www.ti.com/lit/gpn/sn74lv1t04)                   |
| SN74LV1T125     |   1    | 3-state push-pull buffer (active-low OE) |    No     |   15    |          6.8          |          6.8          |           71.43           | [TI SN74LV1T125](https://www.ti.com/lit/gpn/sn74lv1t125)                 |
| SN74LV1T125     |   1    | 3-state push-pull buffer                 |    No     |   30    |          7.5          |          7.5          |           66.67           | [TI SN74LV1T125](https://www.ti.com/lit/gpn/sn74lv1t125)                 |
| SN74LV1T126     |   1    | 3-state push-pull buffer (OE)            |    No     |   15    |          6.8          |          6.8          |           71.43           | [TI SN74LV1T126](https://www.ti.com/lit/gpn/sn74lv1t126)                 |
| SN74LV1T126     |   1    | 3-state push-pull buffer                 |    No     |   30    |          7.5          |          7.5          |           66.67           | [TI SN74LV1T126](https://www.ti.com/lit/gpn/sn74lv1t126)                 |
| SN74LV1T34      |   1    | Push-pull buffer                         |    No     |   15    |          6.8          |          6.8          |           71.43           | [TI SN74LV1T34](https://www.ti.com/lit/gpn/sn74lv1t34)                   |
| SN74LV1T34      |   1    | Push-pull buffer                         |    No     |   30    |          7.5          |          7.5          |           66.67           | [TI SN74LV1T34](https://www.ti.com/lit/gpn/sn74lv1t34)                   |
| SN74LV8T240-EP  |   8    | Push-pull buffer                         |    Yes    |   15    |          14           |          14           |           35.71           | [SN74LV8T240-EP](https://www.ti.com/lit/ds/symlink/sn74lv8t240-ep.pdf)   |
| SN74LV8T240-EP  |   8    | Push-pull buffer                         |    Yes    |   50    |          17           |          17           |           29.41           | [SN74LV8T240-EP](https://www.ti.com/lit/ds/symlink/sn74lv8t240-ep.pdf)   |
| SN74LV8T240-Q1  |   8    | Push-pull buffer                         |    Yes    |   15    |          7.7          |          9.8          |           57.14           | [SN74LV8T240-Q1](https://www.ti.com/lit/ds/symlink/sn74lv8t240-q1.pdf)   |
| SN74LV8T240-Q1  |   8    | Push-pull buffer                         |    Yes    |   50    |         10.6          |         12.5          |           43.29           | [SN74LV8T240-Q1](https://www.ti.com/lit/ds/symlink/sn74lv8t240-q1.pdf)   |
| SN74LV8T244-Q1  |   8    | Push-pull buffer                         |    No     |   15    |          8.8          |         11.3          |           49.75           | [SN74LV8T244-Q1](https://www.ti.com/lit/ds/symlink/sn74lv8t244-q1.pdf)   |
| SN74LV8T244-Q1  |   8    | Push-pull buffer                         |    No     |   50    |          12           |         14.3          |           38.02           | [SN74LV8T244-Q1](https://www.ti.com/lit/ds/symlink/sn74lv8t244-q1.pdf)   |
| SN74LV8T540-Q1  |   8    | Push-pull buffer                         |    Yes    |   15    |          7.9          |         10.1          |           55.56           | [SN74LV8T540-Q1](https://www.ti.com/lit/ds/symlink/sn74lv8t540-q1.pdf)   |
| SN74LV8T540-Q1  |   8    | Push-pull buffer                         |    Yes    |   50    |         10.9          |         12.9          |           42.02           | [SN74LV8T540-Q1](https://www.ti.com/lit/ds/symlink/sn74lv8t540-q1.pdf)   |
| SN74LV8T541-Q1  |   8    | Push-pull buffer                         |    Yes    |   15    |          7.7          |          9.8          |           57.14           | [SN74LV8T541-Q1](https://www.ti.com/lit/ds/symlink/sn74lv8t541-q1.pdf)   |
| SN74LV8T541-Q1  |   8    | Push-pull buffer                         |    Yes    |   50    |         10.6          |         12.5          |           43.29           | [SN74LV8T541-Q1](https://www.ti.com/lit/ds/symlink/sn74lv8t541-q1.pdf)   |
| SN74LV8T541-EP  |   8    | Push-pull buffer                         |    Yes    |   15    |         11.4          |         15.3          |           37.45           | [SN74LV8T541-EP](https://www.ti.com/lit/ds/symlink/sn74lv8t541-ep.pdf)   |
| SN74LV8T541-EP  |   8    | Push-pull buffer                         |    Yes    |   50    |         13.9          |         18.7          |           30.67           | [SN74LV8T541-EP](https://www.ti.com/lit/ds/symlink/sn74lv8t541-ep.pdf)   |
| SN74LV8T9541-EP |   8    | Push-pull buffer                         |    Yes    |   15    |         11.8          |          20           |           31.45           | [SN74LV8T9541-EP](https://www.ti.com/lit/ds/symlink/sn74lv8t9541-ep.pdf) |
| SN74LV8T9541-EP |   8    | Push-pull buffer                         |    Yes    |   50    |         14.3          |         23.7          |           26.32           | [SN74LV8T9541-EP](https://www.ti.com/lit/ds/symlink/sn74lv8t9541-ep.pdf) |

The last entry was used in the design but has poor propagation delay. A new part is needed.

#### TI noninverting buffers & drivers

I sorted by chips with: 8 or 16 inputs, over-voltage capable inputs, fast switching speed, and Vcc of 2.5V. 
- **AUC** Family -Very Fast (<2ns) at 2.5V, but max tolerant IO is 3.6V
	- SN74AUC16244
	- SN74AUC244
	- SN74AUCH16244
	- SN74AUCH244
	- SN74AVC16244
- **AVC** Family -Very Fast (<2ns)  at 2.5V, but max tolerant IO is 4.7V
	- SN74AVC16244
- **AHC** Family -  switch speeds in the 10-12ns region; 7V input tolerant
	- SN74AHC244
	- SN74AHC244-Q
	- SN74AHC244-Q1
	- SN74AHC541
	- SN74AHC541-Q1
- **ALVC** Family - Very Fast (<4ns)  at 2.5V, but max tolerant IO is 4.6V
	- SN74ALVC16244A
	- SN74ALVC244
	- SN74ALVCH162244
	- SN74ALVCH16244
	- SN74ALVCH162344
	- SN74ALVCH244
- **LV-A** Family - too slow (>12ns)  at 2.5V Vcc; inputs tolerant to 7V
propagation delays seemed to be optimized around 5V Vcc in this family.
	- SN74LV244A
	- SN74LV244A-Q1
	- SN74LV244A-EP
	- SN74LV541A
	- SN74LV541A-Q1
- **LVC** Family -Very Fast (<6ns)  at 2.5V, max tolerant IO is 6.5V
	- SN74LVC16244A | 3.9ns propagation delay @ 2.5V
	- SN74LVC16244A-Q1  | 5.9ns propagation delay @ 2.5V
	- SN74LVC16244A-EP  | 3.9ns propagation delay @ 2.5V
	- SN74LVC162244A  | 4.3ns propagation delay @ 2.5V


I found the 16bit SN74LVC16244A-EP part is a good fit with a sub 5ns propagation delay.
Part availability is low for the 'EP' part, so maybe we use the SN74LVC16244A.

## EEPROM
This uses the same setup and Quartz

## FMC
Outside of the Clock lines and fixed function pins, the current pin assignments are just placeholders
