# Beekeep Load-Cell Hardware Notes

## Hardware identified

- Four small 3-wire half-bridge load cells
- Each load cell has red, black, and white wires
- One HX711 load-cell amplifier/24-bit ADC module
- No separate four-load-cell combinator board

## HX711 module connections

The load-cell side of the module is labeled:

- `E+` and `E-`: bridge excitation
- `A+` and `A-`: primary differential signal input (gain 128 in ESPHome)
- `B+` and `B-`: secondary input channel; not planned for this scale

The controller side includes:

- `VCC`: power
- `GND`: ground
- `DT`: data output to the ESP32
- `SCK`: clock input from the ESP32

Planned ESP32 connections in `configuration.yaml`:

- HX711 `DT` to ESP32 `GPIO21`
- HX711 `SCK` to ESP32 `GPIO22`
- HX711 `VCC` to ESP32 `3.3V`
- HX711 `GND` to ESP32 `GND`

## Load-cell wiring status

The four 3-wire cells can be wired manually as a complete Wheatstone bridge, so a combinator board is optional. Do not assign wire functions from color alone because load-cell color conventions vary.

Before finalizing the four-cell wiring, measure one disconnected load cell with a multimeter in resistance/ohms mode:

1. Red to black
2. Red to white
3. Black to white

The wire shared by the two approximately half-resistance readings is the center tap. The remaining pair should measure approximately twice that resistance. Use those readings to determine the exact connections to HX711 `E+`, `E-`, `A+`, and `A-`.

## ESPHome status

`configuration.yaml` contains an HX711 sensor in raw calibration mode:

- Sensor name: `Hive Scale Raw`
- ID: `hive_scale_raw`
- Gain: 128 (channel A)
- Samples every 5 seconds
- Median filter: 7-sample window, publishes every 6 samples

After wiring and firmware upload, record the stabilized raw reading with the platform empty and again with a known weight. Those two points will be used with ESPHome's `calibrate_linear` filter to report actual hive weight in pounds.

