# Sweep dongle port audit

## Scope

- Work only on `hana-feature-port`; `main` remains unchanged.
- First controller target: `nice_nano//zmk`, the custom ZMK board target for nice!nano / compatible Pro Micro-form-factor nRF52840 controllers.
- Preserve the original Sweep keymap and its four-layer behavior.
- Build standalone BLE and dongle modes from the same Sweep configuration.
- First dongle display target: 128x64 SSD1306.

## Dependency pins

The manifest pins the custom ZMK, shared keyboard-feature module, and dongle display module to the revisions used by the existing feature implementation. These pins are intentional: upstream ZMK v0.3 does not include the custom timeout-feedback events needed by the current feature set.

## Build matrix

| Artifact | Role |
| --- | --- |
| `sweep_ble_left_central` | Left half, standalone BLE central |
| `sweep_ble_right_peripheral` | Right half, standalone BLE peripheral |
| `sweep_dongle_left_peripheral` | Left half, dongle mode peripheral |
| `sweep_dongle_right_peripheral` | Right half, dongle mode peripheral |
| `sweep_dongle_central_128x64` | nice!nano dongle central with SSD1306 |

## Initial validation gates

1. Confirm the custom ZMK revision and module pins resolve in GitHub Actions.
2. Confirm all five firmware targets compile.
3. Inspect the resulting artifacts and check for unintended keymap changes.
4. Hardware-test BLE split behavior and dongle mode independently.
5. Verify split pairing/reconnection and battery telemetry on hardware before declaring the port fully functional.

A green build is not proof of Bluetooth or display runtime behavior. Advanced key-triggered settings are added only after the initial matrix compiles, using unused Sweep key positions so existing bindings remain intact.
