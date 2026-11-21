# MaRTy

**The mobile human-biometeorological station of the SHaDE Lab: the datalogger program that turns its sensors into mean radiant
temperature, and the field guide for running it.**

[SHaDE Lab](https://shadelab.asu.edu/tools-data/) · Arizona State University · License: [CERN-OHL-P-2.0](LICENSE)

MaRTy (Mean Radiant Temperature cart) is a wheeled cart carrying a mast with twelve radiation sensors, a weather probe, an
ultrasonic anemometer and a GPS. Walked through a place, it records what a pedestrian is exposed to every 2 seconds and computes mean
radiant temperature (MRT) on board. Its measurements are the training data behind [WebMRT](../webmrt) and a validation reference for
the lab's other models. An interactive 3D model of the cart is on the [OpenThermal website](../../index.html#marty).

## What is in this folder

| File | What it is |
|---|---|
| [`2022-0518_MaRTy2.CR3`](2022-0518_MaRTy2.CR3) | The CRBasic program that runs on MaRTy 2's Campbell Scientific **CR3000** datalogger: reads every sensor, computes the radiation budget and MRT, and writes a 2-second table to the memory card |
| [`MaRTy_cheatSheet_2024_0511.pdf`](MaRTy_cheatSheet_2024_0511.pdf) | Two-page field guide (April 2024): what to check before fieldwork, how to assemble the cart, take measurements and disassemble it |
| [`LICENSE`](LICENSE) | CERN Open Hardware Licence Version 2, Permissive (official text, unmodified) |
| [`CITATION.cff`](CITATION.cff) | How to cite this work |

This folder holds the **logger program and the field guide**. It does not contain mechanical drawings, a bill of materials, wiring
diagrams or the calibration certificates of the instruments (see [Limitations](#limitations)).

## The instruments

| Measurement | Sensor | How the program reads it |
|---|---|---|
| Shortwave and longwave radiation, six directions | Three Hukseflux **NR01** four-component net radiometers, about 1.3 m above the ground | Differential voltages: four per radiometer (shortwave in and out, longwave in and out) plus its Pt100 body temperature |
| Air temperature and relative humidity | **EE181** probe in a radiation shield | Single-ended voltages on channels 1 and 2 (temperature: 0.1 °C per mV with a −40 °C offset; humidity: 0.1 % per mV) |
| Wind speed and direction | **Gill WindSonic** two-dimensional ultrasonic anemometer | RS-232 at 9600 baud on `Com1`, with a checksum and diagnostic code |
| Position, speed, course, altitude | **Garmin GPS16X** | NMEA on `Com2`, clock in UTC−7 (Arizona, no daylight saving); speed converted from knots to m/s |
| Logger housekeeping | CR3000 | Battery voltage and panel temperature |

The 2019 paper lists a Rotronic HygroClip2 probe for temperature and humidity; the current setup and the cheat sheet use an EE181.

### Which radiometer looks where

| Radiometer | Looks | Angular weight | Mounted |
|---|---|---|---|
| NR01 no. 1 | Left and right | 0.22 each | End of the short pole, which runs in line with the cart handle |
| NR01 no. 2 | Back and front | 0.22 each | One end of the long pole, perpendicular to the short pole and centred |
| NR01 no. 3 | Up and down | 0.06 each | The other end of the long pole |

NR01 no. 3 is read through an AM16/32 multiplexer (enable on control port 6, clock on port 5) so its four signals share one
differential channel. The cheat sheet's diagram shows this layout; check which radiometer goes where each time you assemble the cart.

## What the program computes

Every 2 seconds, for each of the three radiometers:

* **Net shortwave and longwave**, `K_in − K_out` and `L_in − L_out`, the **albedo** (reflected over incoming shortwave) and the total
  radiation arriving from each side.
* **Corrected longwave.** The pyrgeometer signal is relative to the instrument body, so the body's own emission is added back using
  its Pt100 temperature: `L = L_signal + σ·T_body⁴`.
* **Surface temperature** seen by each sensor, from the corrected longwave: `T = (L / σ)^¼ − 273.15`.

Then the **mean radiant temperature** from the six directions, with angular factors for a standing person
(Höppe 1992; VDI 1998) and absorption coefficients `a_k = 0.70` for shortwave and `a_l = 0.97` for longwave:

```
MRT = [ Σ  W_i · ( a_k·K_i + a_l·L_i ) / ( a_l·σ ) ] ^ ¼  −  273.15        σ = 5.67×10⁻⁸ W m⁻² K⁻⁴
        i=1..6                                                              W = 0.22 (left, right, front, back), 0.06 (up, down)
```

This is equation (1) of Middel and Krayenhoff (2019), and the last line of the program computes it.

### Output

The `Output` table is written to the memory card every 2 seconds, overwriting the oldest data when the card is full. It holds the logger
battery and panel temperature, air temperature, humidity, wind speed and direction, GPS position, speed, course and altitude, the radiation
terms of all three radiometers and `MRT`. A second table, `Table2`, keeps the daily minimum battery voltage.

## Using the program

1. Open `2022-0518_MaRTy2.CR3` in Campbell Scientific's CRBasic Editor (part of LoggerNet) and send it to a CR3000 datalogger.
2. **Check the calibration multipliers first.** Each `VoltDiff` line for a radiometer carries a number such as `55.30973`, marked
   `'adjusted calibration` in the file. It converts millivolts to W m⁻² and belongs to the specific radiometer that was on the cart in
   May 2022. For any other instrument, replace it with the value from that instrument's calibration certificate, or the radiation values
   and MRT will be wrong.
3. Check that the wiring matches the channel numbers in the program and the table above.
4. Follow the cheat sheet for the field routine. In short: charge the battery the night before, bring the poles, tools and memory card,
   attach the cross poles and then the radiometers (check which is up/down, left/right, back/front), switch the logger on and wait for it to
   say "running", and check the real-time output tables for plausible values.

### Measuring well

From the cheat sheet and the paper:

* For stop-and-go transects, stop at the site and step away for at least one minute so the sensors settle to their response time, and keep
  a log of stop times.
* For stationary measurements, stay away from the cart and angle the handle to the side of the back/front radiometer, so it does not
  disturb the up/down one.
* In the 2019 study the first 10 and last 5 two-second records at each stop were discarded, the rest averaged, and values detrended to the
  hour for comparison across sites.
* Wipe the radiometers with lens wipes, keep cables close to the poles, and carry the cart with two people, holding the bottom and not the
  red grid.

## Limitations

* **This is the logger program and the field guide only.** There are no mechanical drawings, bill of materials, wiring diagrams or
  calibration certificates here, so the folder does not, by itself, let someone build a MaRTy. Adding those would make it a complete open
  hardware release.
* The calibration multipliers in the program are specific to the instruments that were on the cart and must be replaced for any other
  radiometers.
* The program could not be run or tested here: it needs a CR3000 and the sensors. The MRT line follows the published equation.
* In the program's `Units` section the albedo is labelled `W/m^2`; it is a ratio and has no unit.
* The 3D model on the website is illustrative and is not a construction drawing.

## Citation

Middel, A., & Krayenhoff, E. S. (2019). Micrometeorological determinants of pedestrian thermal exposure during record-breaking heat in
Tempe, Arizona: Introducing the MaRTy observational platform. *Science of The Total Environment*, 687, 137–151.
<https://doi.org/10.1016/j.scitotenv.2019.06.085>

See also `CITATION.cff`.

## License

[CERN Open Hardware Licence Version 2 - Permissive (CERN-OHL-P-2.0)](LICENSE), applied to the files in this folder. The rest of the
OpenThermal repository is under the Apache License 2.0. The CRBasic program was created with Campbell Scientific's Short Cut and
modified by Ariane Middel (2019); Campbell Scientific, Hukseflux, Gill and Garmin product names belong to their owners.
