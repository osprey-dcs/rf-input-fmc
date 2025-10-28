- 2 Independent RF inputs
- 15 Digital inputs
- 1 PPS input


## Clock Inputs

| **parameter**            | value                                 |
| ------------------------ | ------------------------------------- |
| connector type           | Samtec Isorate IP5-02-01-L-S-RA1-L-TR |
| Targeted Input Bandwidth | 50MHz- 200Mhz                         |
| Absolute Input Bandwidth | 50MHz- 3000Mhz                        |
| Input Range              | -5 to +25 dBm                         |
| Absolute Input Range     | -5.8dbm - 25.9 dBm                    |
| Clock Divider            | 1x - (1/8)x                           |
| Input Impedance          | 50 ohm +/- 0.5%                       |


### Input Bandwidth
We are targeting a 50-200MHz input, while the input parts are rated for higher then 200Mhz operation, we will be likely limited by non-ideal effects of the realized circuit.
LMK and front end can handle

| **Part**   | **Frequency(MHz)** |
| ---------- | ------------------ |
| TCP-2-33W+ | 50 - 3000          |
| TCM2-33WX+ | 10 - 3000          |
| LMK01801   | ?- 3100            |
The FPGA Clock input only operate up to 650MHz.
<span style="color: red;">Dividers bringing the final input frequency down to below 650MHz must be used.</span>
Realistically, that layout will limit the bandwidth *we should test this limit*

### Input Range
There will some loss in the splitter transformer:
![[Pasted image 20251021103838.png]]

I'll use 3.5dB

There will some loss in the input transformer
This is the chart from the TCM2-33WX+ Input Coupling transformer

| **Frequency(MHz)** | Insertion Loss(dB) | Return Loss(dB) | Amplitude Unbalance(dB) | Phase Unbalance(deg) |
| ------------------ | ------------------ | --------------- | ----------------------- | -------------------- |
| 10                 | 0.87               | 21.66           | 0.03                    | 0.01                 |
| 100                | 0.78               | 28.58           | 0.03                    | 0.04                 |
| 400                | 0.95               | 21.26           | 0.05                    | 0.82                 |
|                    |                    |                 |                         |                      |
We can add another .8dB for a total of 4.3dB loss before the signal reaches the LMK01801

From LMK01801:
Vin_max = 3.1V p-p
Vin_min = 0.5V p-p

So input range of -5.8dbm - 25.9 dBm 
## Digital Inputs

| **parameter**               | value                                 |
| --------------------------- | ------------------------------------- |
| connector type              | Samtec Isorate *Part Number*          |
| input impedance             | 50 ohm(PPS Input) or high impedance   |
| V<sub>IH</sub>              | > 1.28 V                              |
| V<sub>IL</sub>              | < 0.65 V                              |
| Intended Input Logic Levels | 5.0V, 3.3V, 2.5V                      |
| Absolute Maximum Input      | 7V                                    |
| Maximum Switching Speed     | <span style="color: red;">????</span> |

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

## FMC PINOUT
<span style="color: red;">Add pinout here</span>
** What cannot be tri-stated with other firmware
- sample clk output - Quartz?
