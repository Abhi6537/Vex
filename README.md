<div align="center">

<a href="https://www.youtube.com/watch?v=kzHCAnwe_4Q">
  <img src="https://img.shields.io/badge/%E2%96%B6%20WATCH%20THE%20DEMO-FF0033?style=for-the-badge&logo=youtube&logoColor=white" alt="Watch the Vex demo on YouTube">
</a>

<br><br>

<pre>
██╗   ██╗███████╗██╗  ██╗
██║   ██║██╔════╝╚██╗██╔╝
██║   ██║█████╗   ╚███╔╝
╚██╗ ██╔╝██╔══╝   ██╔██╗
 ╚████╔╝ ███████╗██╔╝ ██╗
  ╚═══╝  ╚══════╝╚═╝  ╚═╝
</pre>

<h3>ONE HAND · FULL CONTROL</h3>
<p><strong>A motion-powered wearable gamepad for players who need their second hand for something else.</strong></p>

<p>
  <a href="https://github.com/Abhi6537/Vex"><img src="https://img.shields.io/github/stars/Abhi6537/Vex?style=flat-square&color=7C3AED" alt="GitHub stars"></a>
  <a href="https://github.com/Abhi6537/Vex/blob/main/LICENSE"><img src="https://img.shields.io/badge/license-MIT-00C2A8?style=flat-square" alt="MIT License"></a>
  <a href="https://platformio.org/"><img src="https://img.shields.io/badge/platform-PlatformIO-F59E0B?style=flat-square" alt="PlatformIO"></a>
  <a href="https://www.espressif.com/en/products/socs/esp32"><img src="https://img.shields.io/badge/board-ESP32-2563EB?style=flat-square" alt="ESP32"></a>
</p>

</div>

<table>
<tr>
<td bgcolor="#17122B" width="50%">
<h3>◈ THE PROBLEM</h3>
Most controllers assume two free hands. That breaks down for streamers, makers, crowded demo spaces, long sessions, and players with limited mobility.
</td>
<td bgcolor="#062B2A" width="50%">
<h3>◆ THE VEX APPROACH</h3>
Vex moves the control surface onto a glove: a joystick handles movement while wrist motion handles aiming.
</td>
</tr>
</table>

## ⚡ What is Vex?

**Vex** is an open hardware, one-handed wearable controller built around an ESP32. It turns finger inputs, an analog joystick, and an inertial measurement unit into a wireless Bluetooth gamepad.

The goal is not to replace every controller. The goal is narrower and more useful:

> **When you do not have two hands to give a game, you should not have to give up control.**

<table>
<tr>
<td align="center" bgcolor="#111827"><h3>🎙️ STREAM</h3><sub>Keep a mic or second device in your free hand.</sub></td>
<td align="center" bgcolor="#1E1B4B"><h3>♿ ACCESS</h3><sub>A legitimate one-handed input path.</sub></td>
<td align="center" bgcolor="#172554"><h3>🖐️ COMFORT</h3><sub>Reduce reach and sustained gripping.</sub></td>
</tr>
</table>

## 🧠 How it works

| Input layer | Vex translation | Output |
|:--|:--|:--|
| Analog joystick | Centered, dead-zone-filtered axes | **Left stick** |
| Wrist tilt | Complementary-filtered IMU orientation | **Right stick** |
| Joystick press | Digital switch | **Gamepad button** |
| Finger buttons | Digital contact points | **Expandable controls** |

The ESP32 performs the full pipeline on-device. There is no phone app, cloud service, or desktop translation layer in the loop.

## 🧩 Hardware stack

<table>
<tr><th align="left">Part</th><th align="left">Purpose</th><th align="left">Status</th></tr>
<tr><td><code>ESP32 DevKit</code></td><td>Processing + Bluetooth Low Energy HID</td><td>Core</td></tr>
<tr><td><code>Analog joystick</code></td><td>Primary movement input</td><td>Core</td></tr>
<tr><td><code>MPU6050 / MPU6500-class IMU</code></td><td>Wrist orientation and aiming</td><td>Core</td></tr>
<tr><td><code>Digital finger inputs</code></td><td>Wearable button/contact points</td><td>Prototype</td></tr>
<tr><td><code>Resistors + wiring</code></td><td>Signal conditioning and integration</td><td>Required</td></tr>
<tr><td><code>Glove base</code></td><td>Mechanical mounting and interaction</td><td>Custom</td></tr>
</table>

## 🎛️ Current pin map

The active mapping lives in [`src/main.cpp`](src/main.cpp).

| Function | GPIO | Function | GPIO |
|:--|--:|:--|--:|
| Joystick X | `36` | Joystick Y | `34` |
| Joystick press | `5` | IMU SDA | `27` |
| Digital input 1 | `4` | IMU SCL | `14` |
| Digital input 2 | `2` | Digital input 3 | `22` |
| Digital input 4 | `23` | — | — |

## 🌀 Signal pipeline

```text
  ┌─────────────┐     ┌──────────────┐     ┌─────────────┐
  │  Joystick   │────▶│              │────▶│             │
  └─────────────┘     │              │     │  BLE HID   │──▶ Game
                      │    ESP32     │     │  Gamepad   │
  ┌─────────────┐     │              │     │             │
  │  MPU6050    │────▶│  Calibrate   │────▶└─────────────┘
  └─────────────┘     │  Filter     │
  ┌─────────────┐     │  Map        │
  │ Finger I/O  │────▶│              │
  └─────────────┘     └──────────────┘
```

1. **Calibrate on boot** — joystick center and stationary IMU bias are sampled.
2. **Fuse motion data** — gyro responsiveness is combined with accelerometer stability.
3. **Clean the signal** — dead zones, clamping, and axis mapping remove unwanted drift.
4. **Speak HID** — the host sees a standard Bluetooth gamepad.

> 🟡 **Calibration note:** Keep the wearable still during startup. Recalibrate after changing the IMU mounting angle.

## 🛠️ Build and flash

### Requirements

- [PlatformIO](https://platformio.org/)
- ESP32 DevKit
- Arduino framework
- BLE-capable host device

```bash
git clone https://github.com/Abhi6537/Vex.git
cd Vex
pio run
pio run --target upload
pio device monitor
```

The serial monitor uses **115200 baud**. Pair with the Bluetooth device named **ESP32 Gyro Gamepad**.

<table>
<tr><td bgcolor="#3B1D1D">
⚠️ <strong>Hardware safety:</strong> Verify your board pinout and electrical levels before wiring. GPIO assignments belong to the current prototype and may evolve.
</td></tr>
</table>

## 🧪 Prototype lessons

<details>
<summary><strong>Flex sensors need a wider bend radius</strong></summary>
<br>
Early sensors failed when fabric forced them over one sharp crease. The mounting was redesigned to spread the bend across a wider curve, with solder strain relief and testing under real hand movement.
</details>

<details>
<summary><strong>BLE axis calls are not universal</strong></summary>
<br>
Raw sensor logs were separated from BLE output to isolate the fault. Host/library differences were the issue, so the firmware uses the axis calls that behaved consistently, including the Android-specific right-stick path.
</details>

<details>
<summary><strong>Drift requires two sensors, not one</strong></summary>
<br>
Raw gyro data drifted and raw accelerometer data jittered. A complementary filter plus startup bias calibration made the aiming signal stable and responsive.
</details>

## 🗺️ Roadmap

| Status | Milestone |
|:--:|:--|
| ☐ | Expand finger-contact and button mappings |
| ☐ | Add configurable control profiles |
| ☐ | Improve enclosure and glove mounting |
| ☐ | Add visual calibration feedback |
| ☐ | Test across more operating systems and game genres |
| ☐ | Publish a detailed wiring diagram and build photos |

## ▶️ Demo

<div align="center">

[![Watch the Vex demo](https://img.youtube.com/vi/kzHCAnwe_4Q/hqdefault.jpg)](https://www.youtube.com/watch?v=kzHCAnwe_4Q)

<a href="https://www.youtube.com/watch?v=kzHCAnwe_4Q"><img src="https://img.shields.io/badge/▶%20OPEN%20ON%20YOUTUBE-FF0033?style=for-the-badge&logo=youtube&logoColor=white" alt="Open Vex demo on YouTube"></a>

</div>

## 🤝 Contributing

Hardware revisions, firmware improvements, accessibility testing, and control ideas are welcome. Open an issue with reproducible details or submit a focused pull request with relevant wiring notes.

## 📄 License

Released under the [MIT License](LICENSE).

<div align="center">

<br>

<pre>╭────────────────────────────────────────╮
│  VEX  /  WEARABLE INPUT EXPERIMENT     │
│  BUILD SMALL. PLAY FREELY.             │
╰────────────────────────────────────────╯</pre>

<sub>Built with C++ · ESP32 · sensors · stubborn iteration</sub>

</div>
