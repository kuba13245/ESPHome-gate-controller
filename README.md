# ESPHome Gate Controller for Home Assistant

A smart, ESP8266-based gate/garage door controller integrated natively with Home Assistant. This project provides real-time state tracking using dual limit switches and a custom state machine to accurately determine the gate's dynamic phases (Opening / Closing / Idle).

## Features
* **Native Home Assistant Integration:** Uses the ESPHome API for seamless, local, and fast communication.
* **Apple HomeKit Compatibility:** Easily exposed to HomeKit via the Home Assistant bridge integration, allowing for native control and status monitoring directly from Apple devices.
* **Dual Limit Switch Support:** Monitors both fully open and fully closed states.
* **Custom State Machine:** Infers dynamic states (`OPENING` or `CLOSING`) when the gate is moving between the limit switches by evaluating the `gate_last_state`.
* **Hardware Signal Filtering:** Utilizes internal pull-up resistors and a 2-second debounce filter (`delayed_on`) to prevent false triggers from sensor bouncing.
* **Failsafe Mechanisms:** Includes a Captive Portal fallback for Wi-Fi recovery and safe reboot timeouts.

## Hardware Requirements
* **Microcontroller:** ESP8266 (NodeMCU v2)
* **Actuator:** 1x Relay Module (to simulate the push-button remote / dry contact)
* **Sensors:** 2x Magnetic Reed Switches / Limit Switches (Normally Open or Normally Closed)
* **Power Supply:** 5V step-down buck converter (if powered directly from the gate motor controller)

## Pinout & Wiring

| ESP8266 Pin | Component / Function | Logic Configuration |
| :--- | :--- | :--- |
| **D1** | Open Limit Switch | Input, Internal Pull-up, Active-Low (Inverted) |
| **D2** | Close Limit Switch | Input, Internal Pull-up, Active-Low (Inverted) |
| **D5** | Relay Control (Motor) | Output (Triggers a 3-second pulse) |

*Note: Limit switches are configured with internal pull-up resistors and inverted logic, meaning they trigger when pulled to GND.*

## Logic & Operation
The controller does more than just actuate a relay. It runs a 1-second interval loop to continuously evaluate the physical state of the gate:
1. **Fully Closed:** The close limit switch (D2) is triggered. State updates to `CLOSED`, and the `gate_last_state` is saved as `0`.
2. **Opening:** The relay pulses for 3 seconds. The close switch is released. The loop detects that both switches are OFF and the last state was `0`, automatically setting the Home Assistant cover state to `OPENING`.
3. **Fully Open:** The open limit switch (D1) is triggered after a 2-second software debounce. State updates to `OPEN`, and `gate_last_state` becomes `1`.
4. **Closing:** The relay pulses again. Both switches are OFF, and the last state was `1`, updating the cover state to `CLOSING`.

## Installation & Setup
1. Clone this repository or copy the `gate.yaml` file.
2. Rename `secrets.yaml.example` to `secrets.yaml` and fill in your network credentials and API keys.
3. Flash the firmware to your NodeMCU using the ESPHome dashboard or CLI.
4. Add the device to Home Assistant via the Integrations page.

---
**License:** GNU General Public License v3.0 (GPLv3)
