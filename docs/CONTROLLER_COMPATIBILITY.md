# Controller compatibility plan

This repository is intended to be a reference port of reusable keyboard and dongle features. Controller selection must remain independent of the original Sweep keymap wherever the hardware and ZMK board support allow it.

## Controller families to support

| Controller family | Intended role | Integration notes |
| --- | --- | --- |
| nice!nano v2 / compatible Pro Micro-form-factor nRF52840 | Split halves and, where wiring permits, dongle | Establish the baseline target first. Verify the exact board definition and any clone-specific hardware differences. |
| SuperMini nRF52840 | Split halves and, where wiring permits, dongle | Treat as a distinct board target. Confirm the selected Zephyr/ZMK board definition, USB behavior, pin mapping, battery sensing and available peripherals before enabling features. |
| XIAO nRF52840 | Optional / less common target | Keep in scope as an optional target. Its form factor, pin mapping and board definition differ from Pro Micro-style controllers, so do not assume wiring or shield compatibility. |

These are planned compatibility targets, not a claim that every controller or feature has already been built or hardware-tested.

## Configuration principles

1. Keep the original Sweep keymap, layers, combos and bindings as the starting point.
2. Keep board/controller selection separate from connectivity mode and optional features.
3. Use explicit build-matrix entries for each supported controller + shield/role combination. Do not assume that a successful build for one nRF52840 board validates another.
4. Gate hardware-dependent features (OLED, battery telemetry, charging status, USB behavior and GPIO-specific functionality) on verified board capabilities.
5. Share reusable feature code where practical; isolate pin assignments, devicetree overlays, board definitions and other hardware-specific integration.
6. Pin the ZMK revision and module dependencies used by the reference port, and document any custom ZMK patches required.
7. Preserve a known-good baseline and validate changes in small steps on the feature branch before considering a merge to main.

## Validation gates

For each controller target:

- [ ] Confirm the exact Zephyr/ZMK board target and build environment.
- [ ] Build left and right halves with the preserved keymap.
- [ ] Build the dongle target if that controller is being used as a dongle.
- [ ] Build each intended OLED variant only where the display and wiring are supported.
- [ ] Verify battery and charging telemetry only when the hardware exposes reliable signals.
- [ ] Hardware-test USB HID, split connectivity, pairing, reconnects and multi-board switching.
- [ ] Record build artifacts, commit SHA, test results and known limitations.

A passing compile is necessary but does not prove runtime or hardware compatibility.

## Current scope

The initial port remains on the hana-feature-port branch. The first task is a compatibility audit and reproducible baseline; controller-specific build entries should be added only after their board identifiers and dependencies have been confirmed. The default main branch is to remain untouched during this work.
