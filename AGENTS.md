# AGENTS.md - ZMK Corne Configuration

This document provides guidance for AI agents operating in this repository.

## Project Overview

This is a **ZMK keyboard firmware configuration** for the Corne split keyboard. ZMK is built on the [Zephyr RTOS](https://www.zephyrproject.org/).

## Build Commands

### Building Firmware

```bash
# From the .zmk/zmk/app directory, build for native_posix testing:
west build -b native_posix_64 -p

# Build for actual hardware (nice_nano_v2):
west build -b nice_nano_v2 -p -- -DSHIELD=corne_left
west build -b nice_nano_v2 -p -- -DSHIELD=corne_right

# Build with custom config:
west build -b nice_nano_v2 -p -- -DSHIELD=corne_left -DCONFIG_ZMK_RGB_UNDERGLOW=y
```

### Running Tests

```bash
# From .zmk/zmk/app directory, run all tests:
./run-test.sh all

# Run a single test:
./run-test.sh tests/conditional-layer/tri-layer

# Run a specific test with verbose output:
ZMK_TESTS_VERBOSE=1 ./run-test.sh tests/combo/basic

# Auto-accept test snapshot changes (after reviewing changes):
ZMK_TESTS_AUTO_ACCEPT=1 ./run-test.sh <path-to-test>
```

### Code Formatting & Linting

```bash
# Format C/C++ code with clang-format (in-place):
clang-format -i src/**/*.c

# Check YAML formatting (from app directory):
npm run prettier:check

# Format YAML files:
npm run prettier:format
```

### Pre-commit Hooks

```bash
# Install pre-commit hooks:
pip3 install pre-commit && pre-commit install

# Run hooks manually:
pre-commit run --all-files
```

## Code Style Guidelines

### C/C++ Source Files

- **Style**: LLVM-based (see `.clang-format`)
  - Indent width: 4 spaces
  - Column limit: 100 characters
  - Includes are NOT sorted (SortIncludes: false)

- **Copyright Header Required** (every source file):
  ```c
  /*
   * Copyright (c) 2020 The ZMK Contributors
   *
   * SPDX-License-Identifier: MIT
   */
  ```

- **Includes Order**:
  1. Zephyr kernel headers (`<zephyr/...>`)
  2. Driver headers (`<drivers/...>`)
  3. ZMK internal headers (`<zmk/...>`)

- **Naming Conventions**:
  - Functions: `snake_case`
  - Constants/Enums: `UPPER_SNAKE_CASE`
  - Types/Structs: `snake_case_t` (suffix `_t`)
  - Variables: `snake_case`
  - Device tree macros: `UPPER_SNAKE_CASE`

- **Device Tree Bindings**:
  ```c
  #define DT_DRV_COMPAT zmk_behavior_key_press
  
  #define KP_INST(n) \
      BEHAVIOR_DT_INST_DEFINE(n, ...);
  
  DT_INST_FOREACH_STATUS_OKAY(KP_INST)
  ```

### YAML Configuration Files

- **Style**: Prettier with config in `.prettierrc.js`
  - End of line: `auto`
  - Trailing commas: `es5`
  - Check with: `npm run prettier:check`

### Devicetree (.keymap, .dtsi)

- **Includes**:
  ```c
  #include <behaviors.dtsi>
  #include <dt-bindings/zmk/keys.h>
  #include <dt-bindings/zmk/bt.h>
  ```

- **Bindings Format** (6+3 layout example):
  ```
  bindings = <
     &kp TAB   &kp Q &kp W &kp E   &kp Y &kp U  &kp I     &kp O   &kp P    &kp BSPC
     &kp LCTRL &kp A &kp S &kp D   &kp H &kp J  &kp K     &kp L   &kp SEMI &kp SQT
                    &kp LGUI &mo 1 &kp SPACE   &kp RET &mo 2 &kp RALT
  >;
  ```

### Error Handling

- Use Zephyr's logging: `LOG_DBG()`, `LOG_ERR()`, `LOG_WRN()`
- Return error codes (negative integers) from functions
- Check return values of `raise_*` functions

### Testing

- Tests use native_posix_64 board emulation
- Each test directory contains:
  - `native_posix_64.keymap` - Test keymap with mock events
  - `keycode_events.snapshot` - Expected output
  - `events.patterns` - Sed patterns to filter output

### Commit Messages

- ZMK follows conventional commits (transitioning)
- Use descriptive titles for PRs
- Include references to fixed issues

## Key Paths

- Config files: `config/`
- ZMK source: `.zmk/zmk/app/`
- Zephyr RTOS: `.zmk/zephyr/`
- Shield definitions: `boards/shields/`
- Build output: `build/` (generated)

## CI/CD

- GitHub Actions workflow: `.github/workflows/build.yml`
- Builds use ZMK's shared workflow: `zmkfirmware/zmk/.github/workflows/build-user-config.yml@v0.3`
