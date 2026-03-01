# Copilot Instructions for BorneoIoT

## Repository Overview

This is a **full-stack open-source aquarium LED WiFi controller** project called BorneoIoT. The repository is a monorepo with four main components:

| Directory | Language/Tech | Description |
|-----------|---------------|-------------|
| `fw/` | C (ESP-IDF) | Firmware for ESP32/ESP32-C3/ESP32-C5 LED controllers |
| `client/` | Dart/Flutter | Cross-platform mobile app (iOS, Android, Windows) |
| `borneopy/` | Python | Python client library and CLI tools |
| `hw/` | Horizon EDA | PCB schematics and hardware designs |

The project uses **CoAP + CBOR** as the communication protocol between the app/SDK and firmware.

---

## Firmware (`fw/`)

### Technology Stack
- **ESP-IDF v5.5.2** (required) — ESP32 SDK
- **CMake** build system
- C language with Zephyr-inspired driver abstraction
- Clang-format for code style (`.clang-format` in `fw/`)

### Directory Structure
```
fw/
├── lyfi/           # LED controller firmware (main application)
│   ├── main/src/   # Application source files
│   ├── boards/     # Board-specific config (ESP32 variants)
│   ├── products/   # Product definitions (bst/blc06mk1, bst/ulva6, etc.)
│   └── CMakeLists.txt
├── components/
│   ├── borneo-core/  # Core shared library (WiFi, NVS, OTA, CoAP, RPC)
│   └── drvfx/        # Driver abstraction framework
├── 3rd-components/   # Third-party ESP-IDF components
└── cmake/            # Build helper CMake modules
```

### Building the Firmware
Firmware is built using Docker with the official ESP-IDF image:

```bash
# Build with Docker (same as CI)
docker run -t -v "$(pwd):/app/borneo" -w "/app/borneo/fw/lyfi" espressif/idf:v5.5.2 /bin/bash -c \
  'git config --global --add safe.directory "*" && idf.py build -DPRODUCT_ID=bst/blc06mk1 -DCMAKE_BUILD_TYPE=Release'
```

Available `PRODUCT_ID` values (matrix in CI):
- `bst/ulva6`
- `bst/blc06mk1` (default)
- `bst/c5devkitc1`

If building locally with ESP-IDF installed:
```bash
cd fw/lyfi
idf.py build -DPRODUCT_ID=bst/blc06mk1 -DCMAKE_BUILD_TYPE=Release
```

### Code Style
- Based on **WebKit** style (`.clang-format`)
- Column limit: 120
- Tab width: 4
- Use `BO_TRY_ESP()` macro for ESP-IDF error handling
- Use `BO_MUST()` / `BO_MUST_ESP()` for fatal error conditions

### CI Workflow
- File: `.github/workflows/fw-ci.yml`
- Triggers on pushes/PRs to `master`, `dev`, `dev-*` when `fw/**` files change
- Runs a matrix build across all three product IDs

---

## Mobile App (`client/`)

### Technology Stack
- **Flutter** (channel: stable, version pinned in `client/pubspec.yaml`)
- **Melos** for monorepo package management
- **Dart** SDK ^3.10.0

### Workspace Packages
```
client/
├── lib/                  # Main app source (features, core, shared)
├── packages/
│   ├── lw_wot/           # WebThings protocol abstraction
│   ├── borneo_common/    # Shared utilities
│   ├── borneo_kernel/    # Device management, CoAP/CBOR communication
│   └── borneo_kernel_abstractions/  # Abstract interfaces
├── assets/i18n/          # Localization files
├── test/                 # Unit and widget tests
└── pubspec.yaml
```

### Setup and Development
```bash
cd client

# Install Flutter (use version from pubspec.yaml)
flutter pub get

# Bootstrap all workspace packages with melos
dart pub global activate melos
melos bootstrap

# Generate code (if needed)
flutter packages pub run build_runner build

# Run the app
flutter run
```

### Testing and Linting
```bash
# Run Flutter tests
flutter test --no-pub

# Run Dart package tests
melos run test:dart --no-select

# Analyze code
melos run analyze
# or: flutter analyze --no-pub

# Format code (check only)
melos format --set-exit-if-changed .

# Format code (apply)
flutter format .
```

### Building
```bash
# Android APK
flutter build apk --release --target-platform android-arm64 --no-pub

# iOS (no codesign)
flutter build ios --release --no-codesign

# Windows
flutter build windows --release --no-pub
```

### CI Workflow
- File: `.github/workflows/flutter-ci.yml`
- Triggers on pushes/PRs to `master` when `client/**` files change
- Jobs: `test` → `build-apk`, `build-windows`, `build-ios` → `release`
- Requires secrets: `KEYSTORE_BASE64`, `KEYSTORE_KEY_ALIAS`, `KEYSTORE_PASSWORD` (for signed APK)

### Code Style
- Enforced via `analysis_options.yaml` (strict Flutter lints)
- Localization strings in `client/assets/i18n/`

---

## Python SDK (`borneopy/`)

### Technology Stack
- Python with `aiocoap`, `cbor2`, `aiofiles`
- Async/await throughout

### Setup
```bash
cd borneopy
pip install -r requirements.txt
# or
pip install -e .
```

### Key Dependencies
- `aiocoap>=0.4.12` — CoAP protocol
- `cbor2>=5.8.0` — CBOR encoding/decoding
- `aiofiles` — Async file I/O

### Usage
See `borneopy/README.md` for API docs and examples. The primary class is `LyfiCoapClient`.

---

## Hardware (`hw/`)

- PCB designs in **Horizon EDA** format
- OSHWA certified (CN000017): Model BLC06MK1 (6-channel PWM LED controller)
- PDF schematics are provided; raw EDA source files may be partially excluded
- Licensed under **CERN-OHL-S-2.0**

---

## Communication Protocol

All device communication uses **CoAP** (UDP) with **CBOR** encoding:
- CoAP paths follow the pattern `borneo/<device>/<resource>` (e.g., `borneo/lyfi/state`)
- mDNS used for device discovery
- Defined in `fw/components/borneo-core/` and mirrored in `client/packages/borneo_kernel/`

---

## Contributing Guidelines

- Sign the CLA before submitting PRs (enforced by CLA bot)
- Commit messages: present tense, imperative mood, ≤72 chars first line
- Reference issues/PRs in commit messages
- Follow existing code style (clang-format for C, `flutter format` for Dart)
- Firmware changes go in `fw/`, app changes in `client/`

## Licenses
- Software/Firmware: **GPL-3.0+**
- Hardware: **CERN-OHL-S-2.0**
