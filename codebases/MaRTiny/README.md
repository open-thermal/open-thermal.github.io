# MaRTiny

**Firmware for MaRTiny, the SHaDE Lab's low-cost biometeorological sensor that also counts how many people stand in sun and in shade.**

[SHaDE Lab](https://shadelab.asu.edu/tools-data/) · Arizona State University · License: [CERN-OHL-P-2.0](LICENSE)

MaRTiny is "MaRTy's tiny sibling". It measures globe temperature, air temperature, humidity, UV and wind, and photographs the space it watches, so that a server can estimate mean radiant temperature and count pedestrians in sun and in shade. It was deployed in City of Tempe parks from 2021 to 2023. An interactive 3D model of the installation is on the [OpenThermal website](../../index.html#martiny).

This folder holds the **firmware of its two microcontrollers**, the **server software** that stores and analyses what they send (image analysis with YOLO and shadow detection, and the MongoDB database), the **models**, and the tests.

## How it works

```
 sensors ──► Arduino Uno ──serial 115200──► ESP32-CAM ──Wi-Fi──► your server
 DS18B20 x2  averages them      [Send]  ◄──                    photo: multipart POST
 DHT22       and answers   ───► JSON line                      data:  HTTP GET ?data=…&mac=…
 UV, wind                       [ResetESP32] ◄──
 battery
```

Every cycle the ESP32-CAM takes a photo and uploads it, sends `[Send]` to the Arduino, reads back one JSON line of averages, uploads that, and restarts. The Arduino keeps measuring in between and averages everything since the last `[Send]`. If no `[Send]` arrives for 10 minutes, the Arduino cuts the ESP32-CAM's power for 20 seconds and turns it back on. Both boards run a watchdog.

The JSON line looks like this (values are averages; `null` means no valid reading in that window):

```json
{"Temperature":31.50,"Humidity":22.00,"UV_Intensity":7.20,"Dallas1":30.10,"Dallas2":54.90,"Wind_Speed":3.10,"Battery_Voltage":12.40}
```

## What is in this folder

| Path | What it is |
|---|---|
| [`firmware/MaRTiny-Arduino/`](firmware/MaRTiny-Arduino) | Sketch for the Arduino Uno, plus `martiny_sensors.h` (sensor maths, JSON message, command parser) |
| [`firmware/MaRTiny-ESP32/`](firmware/MaRTiny-ESP32) | Sketch for the AI-Thinker ESP32-CAM, plus `martiny_net.h` (URL encoding, JSON collector, upload header), `config.example.h` and `select_pins.h` |
| [`server/`](server) | The image server (Python), the data collector (Node) and the MongoDB management tool. See [`server/README.md`](server/README.md) |
| [`models/`](models) | Which models are used, why the weights are not in the repository, a checksum verifier and a template for the shadow network. See [`models/README.md`](models/README.md) |
| [`tests/`](tests) | C++ tests of the two firmware headers, run from Python |
| [`LICENSE`](LICENSE), [`CITATION.cff`](CITATION.cff) | CERN-OHL-P-2.0 text and citation metadata |

The web viewer (`MaRTiny-view`) is a separate public project, linked from the [MaRTiny wiki](https://github.com/SHaDE-Lab/MaRTiny). Mechanical drawings, a bill of materials and wiring diagrams are not part of this folder.

**Two licensing points before you reuse the models:** YOLOv8 is AGPL-3.0, and the BDRAR shadow network has no stated license. The weights are therefore not stored here; [`models/README.md`](models/README.md) explains the options, including a permissive alternative detector.

## Hardware and wiring

Arduino Uno pins, as used by the sketch:

| Pin | Connected to |
|---|---|
| A0 | ML8511 UV sensor output |
| A1 | 3.3 V reference (read to scale the UV signal) |
| A4 | Cup anemometer, analog voltage |
| A2 | Battery voltage through a 1:2 divider |
| D3 | DHT22 humidity sensor, hanging in one of the two white cups under the board |
| D4 | Two DS18B20 temperature probes on a OneWire bus: one in the globe, the other in the second white cup under the board. The sketch reports them in bus order as `Dallas1` and `Dallas2`; check which is which when you wire yours. |
| D6, D7 | Power control of the ESP32-CAM (LOW = off, HIGH = on) |
| TX, RX | Serial line to the ESP32-CAM, 115200 baud |

The ESP32-CAM is the AI-Thinker board, whose camera pins are in `select_pins.h`.

## Setting it up

1. **Libraries.** In the Arduino IDE (or `arduino-cli`) install: *DHT sensor library*, *OneWire* and *DallasTemperature* for the Uno; for the ESP32-CAM install the *esp32* board package.
2. **Arduino Uno.** Open `firmware/MaRTiny-Arduino/MaRTiny-Arduino.ino`, select *Arduino Uno*, and upload.
3. **Calibrate the anemometer.** `MT_WIND_V_MIN` in `martiny_sensors.h` is the voltage the sensor gives with no wind, and it differs for every sensor. Hold the cups still, read the voltage, and set it a little above that value. The full-scale values (2.0 V = 32 m/s) are those of the sensor used in the paper.
4. **ESP32-CAM.** In `firmware/MaRTiny-ESP32/`, copy `config.example.h` to `config.h` and enter your Wi-Fi name and password and your server's host name and ports. The sketch will not compile without `config.h`, and `config.h` is in `.gitignore`, so your credentials are never committed. Open `MaRTiny-ESP32.ino`, select *AI Thinker ESP32-CAM*, and upload (with GPIO 0 held low while uploading, as for every ESP32-CAM).
5. Wire the boards together (TX to RX, RX to TX, common ground), as in the table above.

## The server side in one picture

```
ESP32-CAM --photo--> image server --> MongoDB <-- data collector <--sensor JSON-- ESP32-CAM
                     (YOLO: who is there, BDRAR: where the shadow is)    '--> the viewer reads /getLatest ...
```

The image server finds people and shadows in each photo and stores how many stand in shade. The data collector stores the sensor messages and serves them. `server/tools/db_admin.py` handles stats, indexes, pruning, removing stored photos and export. Photographs are **not stored by default**. Details, endpoints, the database layout and what changed from the lab's working server are in [`server/README.md`](server/README.md).

## Tests

The logic that can go wrong without anyone noticing is in the two firmware headers and in the server. For the firmware, the tests compile the headers on your computer and check them:

```bash
cd codebases/MaRTiny
python -m pytest                           # firmware logic and the credential checks (needs a C++ compiler)
cd server && python -m pytest              # 74 tests of the image server, database layer, admin tool and model manifest
cd server/edge && npm install && npm test  # 9 tests of the data collector
```

They need a C++ compiler (g++, clang++, or Visual Studio Build Tools on Windows) and are skipped without one. They check, among other things:

* the UV, wind and battery conversions against the formulas, including the edges (below the zero-wind voltage, above full scale, a missing reference reading);
* that the message is **valid JSON** that Python parses, and that the averages are right, including that a failed DHT22 read or a disconnected DS18B20 (−127 °C) no longer drags an average down;
* the serial command parser: noise around a command, a command split across reads, restarts, over-long commands;
* URL encoding round-trips through Python's decoder, and an Arduino message survives the whole trip to the URL;
* the multipart upload header, and that no Wi-Fi password, private server address, error-reporting key, database address, or model weight file appears anywhere in this folder.

What the tests do not cover: the sketches themselves were not run on the boards, and the ESP32-CAM sketch was not compiled for the ESP32 here, because that needs the board package (the Arduino sketch was compile-checked for the Uno). The server tests use an in-memory stand-in for MongoDB, not a real MongoDB. Test on your hardware and database before relying on a deployment.

## Changes from the lab's working version

This is the lab's firmware, prepared for publication. Behaviour is the same except for these deliberate changes:

* **Secrets removed.** The Wi-Fi name and password and the server address are no longer in the code; they come from `config.h`. If the working version's Wi-Fi password is still in use anywhere, change it: it was in the lab's source files.
* **Better averages.** A failed DHT22 read or a disconnected DS18B20 used to count as a sample (the −127 °C placeholder was added to the sum). They are now skipped, and a sensor with no valid reading reports `null` instead of a wrong number.
* **Valid JSON.** The original line could contain `nan`, which is not JSON; it now contains `null`.
* **Fixed-size buffers** replace growing `String` objects in the command parser, the JSON collector and the message, which is safer on a small microcontroller.
* **`select_pins.h` added** (the original sketch included it but it was not in the repository).
* The sensor maths and the serial protocol moved into the two headers so they can be tested; the formulas are unchanged.

## Security notes

The upload as written sends photos and data over plain HTTP with no authentication, and the servers have no login, so anyone who knows the address could post to your server or read the traffic. For a real deployment put the servers behind HTTPS and a reverse proxy that requires a token, keep the server address out of public code, and restrict who can reach MongoDB. The server's own fixes (input checks, photographs off by default, no open CORS) are listed in [`server/README.md`](server/README.md).

## Citation

Kulkarni, K. K., Schneider, F. A., Gowda, T., Jayasuriya, S., & Middel, A. (2022). MaRTiny: A Low-Cost Biometeorological Sensing Device With Embedded Computer Vision for Urban Climate Research. *Frontiers in Environmental Science*, 10, 866240. <https://doi.org/10.3389/fenvs.2022.866240>

## License

[CERN Open Hardware Licence Version 2 - Permissive (CERN-OHL-P-2.0)](LICENSE) for the files in this folder. The rest of the OpenThermal repository is under the Apache License 2.0. Arduino, ESP32 and the sensor libraries keep their own licenses.
