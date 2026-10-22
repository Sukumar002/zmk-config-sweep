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


## ZMK Studio compatibility requirement

Studio compatibility is a release requirement, not a later polish item.

- The firmware must support the user's custom ZMK Studio deployment for on-the-go remapping and supported feature access.
- It must also remain compatible with the official ZMK Studio client wherever the firmware uses the official ZMK Studio RPC protocol and a supported transport.
- Keep firmware RPC/protobuf behavior standard. Do not introduce custom protocol changes that require the custom frontend unless there is a documented, unavoidable extension; any extension must not break official Studio's standard functions.
- Validate Studio transport and role explicitly: USB serial on the device that exposes the Studio endpoint; BLE Studio only where the official firmware/client/platform combination supports it. Split-central/peripheral and dongle-central roles may not expose identical transports.
- Verify the physical-layout metadata, key positions, layer metadata, unlock behavior, and keymap settings persistence. Test live remapping, reconnect, lock/unlock, and stock-settings restore.
- Include enough reserved layers in the compiled keymap for intended Studio layer expansion. Studio can rename/enable layers already present, but cannot add arbitrary new layers beyond those compiled into the firmware.
- Ensure keymap changes through Studio persist as expected. Once users customize the keymap in Studio, source keymap edits generally will not replace the saved live keymap without restoring stock settings.

Official references:
- https://zmk.dev/docs/features/studio
- https://zmk.dev/docs/config/studio
- https://zmk.dev/docs/development/studio-rpc-protocol

## Porting challenges to track

1. **ZMK fork/API compatibility:** custom features may depend on events, Kconfig symbols, behaviors, or APIs not present in upstream ZMK v0.3. Pin and audit the firmware/module combination together.
2. **Shield and matrix adaptation:** each board has its own matrix transform, physical-layout metadata, scanner nodes, GPIO mapping, and left/right role definitions. A dongle build has no local key matrix and must not accidentally inherit active half-scanner behavior.
3. **Controller and wiring differences:** Pro Micro-compatible boards may differ in battery ADC, charge detection, I2C pin aliases, reset/boot behavior, regulator configuration, and available RAM. A compatible footprint does not prove identical electrical behavior.
4. **Display differences:** OLED dimensions, I2C aliasing, rotation/offset, fonts, and widget placement require per-shield configuration and visual validation.
5. **BLE role and connection capacity:** central/peripheral roles, maximum connections, pairing slots, advertising, reconnect policy, and host switching must be validated at runtime, especially when a dongle connects to both halves and supports more than one keyboard identity.
6. **Battery and charging telemetry:** peripheral battery reporting is not the same as the dongle's own battery source. Some clones expose USB voltage as a misleading battery reading or lack a usable charge-status signal; hide unsupported telemetry rather than display false values.
7. **Persistent settings and migration:** Studio remaps, timeout selections, display preferences, and connection identities may survive firmware upgrades. Changing settings keys or identity layout needs explicit migration/reset behavior.
8. **Studio memory/transport footprint:** Studio increases firmware RAM/flash use. The official Studio RPC endpoint must be available on the intended central device, with USB/BLE transport configured correctly. A successful compile alone does not establish client compatibility.
9. **Studio feature boundaries:** the official Studio client edits compiled keymap structures and behaviors; it does not dynamically define arbitrary new behaviors or create unlimited layers. Reserve intended layers and keep custom-only controls accessible without making standard keymap editing incompatible.
10. **Feature interactions and regression:** combos, mod-taps, conditional layers, layout shift, timeout cycling, display toggles, sleep, and battery events can interact. Preserve original Sweep bindings and test behavior under both standalone BLE and dongle modes.
11. **Build artifact provenance:** each output must clearly identify board, role, connectivity mode, display, and source commit. Build green is compile evidence only; pairing, reconnect, remapping, battery, and OLED behavior need hardware checks.

## Validation order

1. Get the five current build targets compiling from the pinned dependencies.
2. Confirm keymap and physical-layout metadata are accepted by the Studio-enabled central target.
3. Test official ZMK Studio over the configured transport.
4. Test the custom Studio against the same firmware and standard remapping operations.
5. Validate the custom display/settings features and BLE/dongle behavior on hardware.
6. Add other controllers only after this baseline is reproducible.
