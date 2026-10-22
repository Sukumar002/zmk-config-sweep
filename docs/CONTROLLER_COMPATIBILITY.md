# Controller compatibility and dual-firmware plan

This repository is intended to be a reference port of reusable keyboard and dongle features. It must follow a dual-firmware model: generate both standalone BLE firmware and dongle-based firmware from the same board configuration and preserved keymap.

## Firmware modes

| Mode | Required outputs | Architecture |
| --- | --- | --- |
| Standalone BLE | Left and right keyboard halves | The split halves communicate over BLE; no custom host dongle is required. |
| Dongle | Left half, right half and central dongle | The halves connect to the dongle, which provides the host connection and applicable dongle-side features. |

Connectivity mode is a build dimension, not a separate keymap or a separate manually maintained project. The build matrix must explicitly generate and label artifacts for each mode. Hardware-specific features must be enabled only for targets that support them.

## Controller families to support

| Controller family | Intended role | Integration notes |
| --- | --- | --- |
| nice!nano v2 / compatible Pro Micro-form-factor nRF52840 | Split halves and dongle | Establish the baseline target first. Verify the exact board definition and any clone-specific hardware differences. |
| SuperMini nRF52840 | Split halves and dongle where supported | Treat as a distinct board target. Confirm the selected Zephyr/ZMK board definition, USB behavior, pin mapping, battery sensing and available peripherals before enabling features. |
| XIAO nRF52840 | Optional / less common target | Keep in scope as an optional target. Its form factor, pin mapping and board definition differ from Pro Micro-style controllers, so do not assume wiring or shield compatibility. |

These are planned compatibility targets, not a claim that every controller or feature has already been built or hardware-tested. Controller choice and connectivity mode must be independently represented in the build configuration, but only valid combinations should be enabled.

## Configuration principles

1. Keep the original Sweep keymap, layers, combos and bindings as the starting point.
2. Generate both BLE and dongle firmware from the same board configuration and keymap.
3. Keep controller choice, connectivity mode, display variant and optional features as separate build dimensions.
4. Use explicit build-matrix entries for each supported controller + mode + role combination. Do not assume that a successful build for one nRF52840 board validates another.
5. Gate hardware-dependent features (OLED, battery telemetry, charging status, USB behavior and GPIO-specific functionality) on verified board capabilities.
6. Share reusable feature code where practical; isolate pin assignments, devicetree overlays, board definitions and other hardware-specific integration.
7. Pin the ZMK revision and module dependencies used by the reference port, and document any custom ZMK patches required.
8. Preserve a known-good baseline and validate changes in small steps on the feature branch before considering a merge to main.

## Validation gates

For each supported controller and firmware mode:

- [ ] Confirm the exact Zephyr/ZMK board target and build environment.
- [ ] BLE mode: build left and right halves with the preserved keymap.
- [ ] Dongle mode: build left and right halves plus the central dongle target.
- [ ] Build each intended OLED variant only where the display and wiring are supported.
- [ ] Verify battery and charging telemetry only when the hardware exposes reliable signals.
- [ ] Hardware-test USB HID, split connectivity, pairing, reconnects and multi-board switching.
- [ ] Ensure artifacts are clearly named by board, controller, mode, role and display variant where applicable.
- [ ] Record build artifacts, commit SHA, test results and known limitations.

A passing compile is necessary but does not prove runtime or hardware compatibility.

## Current scope

The initial port remains on the hana-feature-port branch. The first task is a compatibility audit and reproducible baseline; controller-specific build entries should be added only after their board identifiers and dependencies have been confirmed. The default main branch is to remain untouched during this work.
