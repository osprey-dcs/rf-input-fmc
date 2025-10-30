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
| V<sub>IH</sub>              | > 1.70 V                              |
| V<sub>IL</sub>              | < 0.70 V                              |
| Intended Input Logic Levels | 5.0V, 3.3V, 2.5V                      |
| Absolute Maximum Input      | 6.5V                                  |
| Maximum Switching Speed     | <span style="color: red;">????</span> |


## FMC PINOUT
<span style="color: red;">Add pinout here</span>
** What cannot be tri-stated with other firmware
- sample clk output - Quartz?
-
## Connector
The mating connector is as thin as the right angle part so the right angle connector can sit back on the board.
![[Pasted image 20251030092402.png]]

We will need 28 mm of clarence min above the board for the connector
![[Pasted image 20251030104230.png]]
