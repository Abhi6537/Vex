# Vex

### A one-handed wearable game controller

**Vex** is an open hardware experiment that turns a glove-mounted ESP32 into a wireless gamepad. It combines an analog joystick, motion-based aiming, and finger-accessible inputs so players can keep one hand free without giving up meaningful control.

> **One hand. Full control.**

Vex is designed for streamers holding a microphone, players using a second device, crowded demo spaces, long sessions, and anyone who cannot comfortably use a conventional two-handed controller. It is not intended to replace every controller for every genre; it focuses on making one-handed play practical.

## Why Vex?

Traditional controllers assume both hands are always available. Vex challenges that assumption by moving the control surface onto a wearable:

- **Motion replaces reach** — wrist tilt drives the right stick, reducing the need to stretch a thumb across a large controller face.
- **A physical joystick stays onboard** — the primary analog stick remains available for familiar movement and navigation.
- **Inputs stay close to the fingers** — contact points and buttons can be operated without changing grip.
- **Complexity lives in firmware** — combinations and mappings can evolve in software instead of adding bulk to the wearable.
- **Accessibility is part of the design** — a one-handed form factor can be a genuine alternative for players who cannot comfortably operate a standard controller.

## Current hardware and firmware

| Component | Role |
| --- | --- |
| ESP32 DevKit | Sensor processing and Bluetooth Low Energy HID |
| Analog joystick | Left-stick movement |
| MPU6050/MPU6500-class IMU | Wrist orientation and right-stick movement |
| Digital finger inputs | On-wearable buttons/contact points |
| Resistors and wiring | Input conditioning and sensor integration |
| Glove or wearable base | Mechanical mounting and hand interaction |

The current firmware:

- Calibrates the joystick center when the device boots.
- Samples the IMU at startup to estimate accelerometer and gyro bias.
- Fuses gyro and accelerometer data with a complementary filter.
- Applies dead zones and range limits for stable stick output.
- Exposes the device as a native BLE gamepad, with no phone app or cloud service required.

## Controls

The current pin mapping is defined in [`src/main.cpp`](src/main.cpp):

| Function | ESP32 pin |
| --- | ---: |
| Joystick X | GPIO 36 |
| Joystick Y | GPIO 34 |
| Joystick press | GPIO 5 |
| Digital input 1 | GPIO 4 |
| Digital input 2 | GPIO 2 |
| Digital input 3 | GPIO 22 |
| Digital input 4 | GPIO 23 |
| IMU SDA | GPIO 27 |
| IMU SCL | GPIO 14 |

In the current mapping, the physical joystick drives the left stick and IMU pitch/roll drive the right stick. The joystick press and one digital input are mapped to gamepad buttons; the remaining inputs are available for extending the control scheme.

## Signal processing

Vex avoids sending raw sensor values directly to the host:

1. **Boot calibration** measures the joystick center and samples the stationary IMU.
2. **Bias correction** removes the initial accelerometer angle and gyro offsets.
3. **Complementary filtering** uses the gyro for responsive movement while the accelerometer corrects long-term drift.
4. **Dead zones and clamping** prevent small sensor noise from becoming unwanted input.
5. **BLE HID output** sends the resulting values as standard gamepad controls.

Keep the wearable still during startup calibration. Recalibrate after changing the IMU mounting angle or significantly changing the mechanical design.

## Build and flash

### Requirements

- [PlatformIO](https://platformio.org/)
- ESP32 DevKit board
- Arduino framework
- BLE-capable host device

### Steps

```bash
git clone https://github.com/Abhi6537/Vex.git
cd Vex
pio run
pio run --target upload
pio device monitor
```

The serial monitor runs at **115200 baud**. After uploading, pair the device named **ESP32 Gyro Gamepad** with the host running the game.

> **Hardware note:** Verify your board's pinout and electrical levels before wiring. GPIO assignments are implementation details of the current prototype and may change as the wearable evolves.

## Lessons from the build

### Making flex sensors survive movement

Sensors that worked on a flat breadboard failed when repeatedly bent at one crease inside fabric. The mounting was redesigned so the sensor follows a wider curve, solder joints receive strain relief, and testing happens under real hand movement rather than only on a desk.

### Debugging BLE axis behavior

Sensor math and BLE output were tested separately by logging raw readings over serial first. This exposed host/library differences in axis calls and led to using the Android-specific right-stick path where it produced the most reliable result.

### Turning drift into usable aiming

Neither raw gyro data nor raw accelerometer data was sufficient by itself. The complementary filter combines their strengths, while startup calibration accounts for unavoidable hardware bias and produces a steadier aiming signal.

## Roadmap

- [ ] Expand finger-contact and button mappings
- [ ] Add configurable control profiles
- [ ] Improve enclosure and glove mounting
- [ ] Add visual calibration feedback
- [ ] Test across more operating systems and game genres
- [ ] Publish a detailed wiring diagram and build photos

## Demo

Watch the project demonstration on YouTube:

[![Watch the Vex demo](https://img.youtube.com/vi/kzHCAnwe_4Q/hqdefault.jpg)](https://www.youtube.com/watch?v=kzHCAnwe_4Q)

## Contributing

Ideas, hardware revisions, firmware improvements, and accessibility testing are welcome. Open an issue with reproducible details for bugs, or submit a pull request with a focused change and wiring/firmware notes where relevant.

## License

Vex is released under the [MIT License](LICENSE).

Copyright (c) 2026 Abhi6537
