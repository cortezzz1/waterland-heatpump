# Pool Heat Pump ESPHome Interface (CC173C-V3.0)

Reverse engineered ESPHome interface for Chinese OEM swimming pool heat pumps using the **CC173C-V3.0** display board.

This project allows monitoring and control of the heat pump from Home Assistant using an ESP32.

The communication bus between the display and the main controller has been partially decoded, allowing temperatures, modes and status information to be read directly from the heat pump.

Control is currently performed using relay-based button emulation.

---

# Features

## Monitoring

The following values are decoded from the display communication bus:

- Water In Temperature
- Water Out Temperature
- Ambient Temperature
- Coil Temperature
- Exhaust Temperature
- Setpoint
- Operating Mode
- Flow Status
- Maximum Setpoint
- Heat Pump Running Status

## Control

The following controls are available in Home Assistant:

- Power
- Mode
- Temperature Up
- Temperature Down

Control is achieved by electrically simulating physical button presses on the display PCB.

---

# Hardware

## Required Components

- ESP32 DevKit (ESP32-WROOM-32)
- Bidirectional Logic Level Shifter
- 4-Channel Relay Module (Opto-Isolated)
- CC173C-V3.0 Display Board

---

# Display Board

Tested with:

**Display PCB: CC173C-V3.0**

The display communicates with the main controller using a proprietary serial communication bus.

No public protocol documentation could be found during development.

---

# Display Connector

The following wiring was identified on the display board:

| Wire Color | Function |
|------------|----------|
| Red / Brown | +12V Supply |
| Yellow | GND |
| Blue | NET Data Bus |

Measured values:

- Supply voltage ≈ 12V
- NET idle voltage ≈ 4V
- Communication pulses present during operation

---

# Reading the Bus

The NET signal is connected through a bidirectional logic level shifter before being connected to the ESP32.

## Wiring

| Heat Pump | Level Shifter | ESP32 |
|------------|------------|------------|
| Blue (NET) | HV Side | GPIO34 |
| Yellow (GND) | GND | GND |
| Red/Brown (+12V) | Not Connected | — |

### ESP32 Connections

| GPIO | Function |
|--------|----------|
| GPIO34 | NET Bus Input |
| GPIO25 | Power Relay |
| GPIO26 | Mode Relay |
| GPIO27 | Temperature Up Relay |
| GPIO32 | Temperature Down Relay |

---

# Relay-Based Control

Although communication from the display to the controller could be decoded successfully, reliable command injection was not achieved.

The following methods were tested:

- Frame replay
- Complete button sequence replay
- Modified frame injection
- Direct command injection
- Dynamic frame reconstruction

The heat pump ignored injected commands.

The final solution uses relay outputs connected in parallel with the display buttons.

This behaves exactly like pressing the physical buttons.

The relays are controlled by ESPHome and exposed to Home Assistant as buttons.

---

# Decoded Data

Currently decoded from the communication bus:

| Value | Status |
|---------|---------|
| Water In Temperature | ✅ |
| Water Out Temperature | ✅ |
| Ambient Temperature | ✅ |
| Coil Temperature | ✅ |
| Exhaust Temperature | ✅ |
| Setpoint | ✅ |
| Maximum Setpoint | ✅ |
| Operating Mode | ✅ |
| Flow Status | ✅ |
| Running Detection | ✅ |

---

# Home Assistant Entities

## Sensors

- WP Water In
- WP Water Out
- WP Omgeving
- WP Coil
- WP Uitlaatgas
- WP Setpoint
- WP Max Setpoint
- WP Delta Water
- WiFi RSSI

## Binary Sensors

- WP Flow OK
- WP No Flow
- WP Actief

## Text Sensors

- WP Mode
- WP Display Pagina

## Buttons

- WP Druk Power
- WP Druk Mode
- WP Temp Omhoog
- WP Temp Omlaag

---

# Example Automations

This project can be integrated with Home Assistant automations such as:

### Solar Forecast Heating

Enable the heat pump automatically when sufficient solar production is predicted.

### Scheduled Pool Heating

Run the heat pump only during selected hours.

### Temperature Monitoring

Monitor pool temperatures directly in Home Assistant dashboards.

### Energy Optimization

Combine solar forecasts, electricity prices and pool temperature targets.

---

# Reverse Engineering Notes

The display bus was monitored using ESPHome's `remote_receiver`.

Example frames captured during operation:

```text
D1 44 14 45 45 05 55 11 ...
D2 88 28 8A 8A 0A AA 22 ...
DD 16 17 0A 08 10 ...
C4 04 11 45 51 45 45 ...
C8 08 22 8A A2 8A 8A ...
```

Several temperatures, status bits and operating modes were successfully identified.

Native protocol write support remains under investigation.

---

# Current Status

## Working

✅ Communication bus monitoring

✅ Temperature decoding

✅ Setpoint decoding

✅ Maximum setpoint decoding

✅ Operating mode decoding

✅ Flow detection

✅ Heat pump running detection

✅ Home Assistant integration

✅ Relay-based button control

## Experimental

⚠ Direct protocol write support

⚠ Native command injection

⚠ Full protocol decoding

---

# Compatibility

Confirmed working with:

- CC173C-V3.0 display boards

Likely compatible with multiple Chinese OEM pool heat pumps using the same display hardware.

Possible compatible brands may include:

- Waterland
- Aquaforte
- Eco+
- Generic Chinese OEM pool heat pumps

Compatibility with other display revisions has not yet been verified.

---

# Development History

The project started as an attempt to directly control the heat pump through the communication bus between the display and the main controller.

Reading data from the bus proved successful and allowed decoding of temperatures, setpoints and operating modes.

Writing commands to the bus was significantly more difficult. Despite extensive testing of replayed and reconstructed frames, reliable command execution could not be achieved.

The final implementation therefore uses relay-based button emulation, which has proven completely reliable for daily operation.

---

# Future Plans

Possible future improvements:

- Full protocol decoding
- Native setpoint control
- Native mode control
- Native power control
- Additional diagnostics
- Fault code decoding
- Automatic protocol discovery

---

# Disclaimer

This project is not affiliated with the manufacturer of the heat pump or display board.

Use at your own risk.

Always verify wiring before connecting an ESP32 to your equipment.

The author assumes no responsibility for damage caused by incorrect installation or use.

---

# Credits

This project was developed through reverse engineering of the CC173C-V3.0 display communication bus using ESPHome, Home Assistant and extensive protocol analysis.

If you have the same display board and discover additional protocol information, contributions are welcome.
