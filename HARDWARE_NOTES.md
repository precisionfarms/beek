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

## Noctua exhaust-fan control

The controller supports a Noctua NF-F12 industrialPPC-2000 IP67 PWM fan through ESPHome:

- ESP32 `GPIO25`: 25 kHz PWM command
- ESP32 `GPIO27`: fan tachometer/RPM input
- Home Assistant entity: `Hive Exhaust Fan`, adjustable from 0% to 100%
- Home Assistant sensor: `Hive Exhaust Fan Speed`, reported in RPM

### Fan connector

- Pin 1, black: ground
- Pin 2, yellow: +12 V fan power
- Pin 3, green: tachometer output
- Pin 4, blue: PWM command

Power the fan directly from the protected 12 V input rail, not from the LM2596 output used for the ESP32. Connect the 12 V supply ground, LM2596 ground, ESP32 ground, and fan ground together. The fan is rated for 0.1 A maximum, but fuse and size the incoming supply for the complete controller.

### Required open-drain PWM interface

Do not connect the blue fan PWM wire directly to 12 V. Use a small open-drain interface:

1. Connect `GPIO25` through approximately 100 ohms to the gate of a 2N7000 or 2N7002 N-channel MOSFET.
2. Add a 100k resistor from the MOSFET gate to ground so the transistor stays off during reset.
3. Connect the MOSFET source to ground.
4. Connect the MOSFET drain to the fan's blue PWM wire.
5. Do not add a PWM pull-up; the fan provides its own compatible pull-up.

ESPHome marks the LEDC output inverted to compensate for the transistor, so a requested 100% corresponds to full fan speed and 0% commands stop.

### Tachometer input

The green tachometer wire is an open-collector output with two pulses per revolution:

1. Connect the green wire to `GPIO27`.
2. Add a 10k pull-up from `GPIO27` to the ESP32 3.3 V rail.
3. Never pull the tachometer line up to 12 V.

The configuration divides pulses per minute by two to publish RPM. If RPM feedback is not needed, leave the green wire disconnected and remove or disable the `pulse_counter` sensor.
