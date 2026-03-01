# Borneo ESPHome Configuration

## Overview

This directory contains ESPHome configuration files for converting the Borneo aquarium LED controller from ESP-IDF to ESPHome for seamless Home Assistant integration.

## Features

The ESPHome implementation provides:

### Core Features
- **6-Channel PWM LED Control**: Precise 12-bit PWM control (0-4095) for each LED channel
- **Home Assistant Integration**: Native ESPHome API for real-time control
- **Thermal Management**: PID-controlled fan system with NTC thermistor temperature monitoring
- **Multiple Control Modes**: Manual, Scheduled, and Sun Simulation modes
- **Brightness Correction**: Support for Linear, Logarithmic, Exponential, Gamma, and CIE1931 correction methods
- **Status Monitoring**: WiFi signal, uptime, temperature sensors
- **OTA Updates**: Over-the-air firmware updates via ESPHome
- **Web Interface**: Built-in web server for standalone control

### Hardware Support (BLC06MK1)
- **MCU**: ESP32-C3
- **LED Channels**: 6 channels (GPIOs: 3, 4, 5, 10, 18, 19)
- **Fan Control**: Dual PWM + voltage regulator control (GPIOs: 6, 7)
- **Temperature**: NTC thermistor on GPIO2
- **Status LED**: GPIO8

## Getting Started

### Prerequisites

1. **Install ESPHome**:
   ```bash
   pip install esphome
   ```

2. **Install Home Assistant** (if not already installed):
   - Follow the [Home Assistant installation guide](https://www.home-assistant.io/installation/)

### Configuration

1. **Copy the secrets template**:
   ```bash
   cd esphome
   cp secrets.yaml.template secrets.yaml
   ```

2. **Edit `secrets.yaml`** with your credentials:
   - WiFi SSID and password
   - API encryption key (generate with: `esphome encryption-key-generate`)
   - OTA password
   - Your geographic coordinates for sunrise/sunset calculations

3. **Validate the configuration**:
   ```bash
   esphome config borneo-lyfi.yaml
   ```

### First Installation

For the initial firmware flash, you'll need to connect the ESP32-C3 via USB:

```bash
esphome run borneo-lyfi.yaml
```

Select the USB serial port when prompted. After the initial flash, all future updates can be done over-the-air (OTA).

### OTA Updates

After the initial installation, you can update wirelessly:

```bash
esphome run borneo-lyfi.yaml --device borneo-lyfi.local
```

Or use the Home Assistant ESPHome dashboard for visual management.

## Home Assistant Integration

Once flashed, the device will automatically appear in Home Assistant under:

**Settings → Devices & Services → ESPHome**

### Available Entities

#### Lights
- `light.led_channel_0_red` - Red channel control
- `light.led_channel_1_green` - Green channel control
- `light.led_channel_2_blue` - Blue channel control
- `light.led_channel_3_warm_white` - Warm white channel control
- `light.led_channel_4_uv_purple` - UV/Purple channel control
- `light.led_channel_5_cool_white` - Cool white channel control

#### Sensors
- `sensor.board_temperature` - Board temperature (°C)
- `sensor.wifi_signal` - WiFi signal strength (dBm)
- `sensor.uptime` - Device uptime

#### Controls
- `number.manual_ch0_brightness` through `number.manual_ch5_brightness` - Individual channel brightness (0-4095)
- `number.target_temperature` - Target temperature for thermal control
- `number.pid_kp`, `number.pid_ki`, `number.pid_kd` - PID tuning parameters
- `select.led_control_mode` - Control mode selection
- `select.brightness_correction` - Brightness correction method
- `switch.enable_thermal_control` - Enable/disable thermal management
- `switch.enable_sun_simulation` - Enable sun tracking
- `fan.cooling_fan` - Manual fan control

## Automation Examples

### Sunrise/Sunset Simulation

Create an automation in Home Assistant:

```yaml
automation:
  - alias: "Aquarium Sunrise"
    trigger:
      - platform: sun
        event: sunrise
        offset: "-01:00:00"  # Start 1 hour before sunrise
    action:
      - service: light.turn_on
        target:
          entity_id:
            - light.led_channel_3_warm_white
            - light.led_channel_5_cool_white
        data:
          brightness_pct: 0
          transition: 3600  # 1 hour transition

  - alias: "Aquarium Sunset"
    trigger:
      - platform: sun
        event: sunset
        offset: "+01:00:00"  # End 1 hour after sunset
    action:
      - service: light.turn_off
        target:
          entity_id: all
        data:
          transition: 3600  # 1 hour transition
```

### Scheduled Lighting

```yaml
automation:
  - alias: "Aquarium Day Cycle"
    trigger:
      - platform: time
        at: "09:00:00"
    action:
      - service: number.set_value
        target:
          entity_id: number.manual_ch0_brightness
        data:
          value: 2048  # 50% brightness
```

### Temperature-Based Fan Control

The PID controller automatically manages fan speed based on temperature. You can adjust the target temperature:

```yaml
automation:
  - alias: "Aquarium High Temp Warning"
    trigger:
      - platform: numeric_state
        entity_id: sensor.board_temperature
        above: 50
    action:
      - service: notify.mobile_app
        data:
          message: "Aquarium controller temperature is high!"
```

## Differences from ESP-IDF Firmware

### What's Changed

1. **Communication Protocol**: 
   - ESP-IDF: CoAP/CBOR over WiFi
   - ESPHome: Native Home Assistant API + optional MQTT

2. **Configuration**:
   - ESP-IDF: Flash-based NVS (Non-Volatile Storage)
   - ESPHome: YAML configuration with Home Assistant entities

3. **Control Interface**:
   - ESP-IDF: Custom mobile app (Flutter)
   - ESPHome: Home Assistant UI + optional web interface

4. **Advanced Features** (implemented in Home Assistant):
   - Scheduling: Use Home Assistant automations
   - Sun simulation: Use Home Assistant sun integration
   - Acclimation mode: Create custom automations with gradual brightness increase
   - Cloud effects: Implement as random brightness variations in automations

### What's Preserved

- ✅ 6-channel PWM LED control
- ✅ 12-bit resolution (0-4095)
- ✅ 19kHz PWM frequency
- ✅ PID-based thermal management
- ✅ NTC thermistor temperature monitoring
- ✅ Fan control with dual PWM outputs
- ✅ OTA updates
- ✅ Status LED indicator

### Migration from ESP-IDF

If you're migrating from the ESP-IDF firmware:

1. **Backup your settings**: Note down your LED schedules, brightness levels, and PID tuning values
2. **Flash ESPHome**: Follow the "First Installation" steps above
3. **Recreate schedules**: Use Home Assistant automations to replicate your schedules
4. **Tune PID**: Transfer your PID Kp, Ki, Kd values to the ESPHome configuration
5. **Test thoroughly**: Verify all channels work and thermal control is functioning

## Customization

### Adjusting PWM Frequency

Edit `substitutions` in `borneo-lyfi.yaml`:

```yaml
substitutions:
  pwm_frequency: "25000 Hz"  # Change to desired frequency
```

### Different Board Variants

For different board variants (e.g., BLC05MK2), adjust the GPIO pins in the `substitutions` section:

```yaml
substitutions:
  led_ch0_gpio: "X"  # Update to match your board
  # ... etc
```

### Adding Custom Sensors

You can extend the configuration with additional sensors supported by ESPHome, such as:
- Current sensors (for power monitoring)
- Additional temperature sensors
- Water level sensors
- pH sensors

See the [ESPHome documentation](https://esphome.io/) for available components.

## Troubleshooting

### Device Not Connecting

1. Check WiFi credentials in `secrets.yaml`
2. Verify the device is powered and status LED is blinking
3. Check Home Assistant logs: **Settings → System → Logs**

### Temperature Reading Issues

1. Verify NTC thermistor calibration values match your hardware
2. Check the `b_constant` value in the configuration
3. Adjust `reference_resistance` if using a different thermistor

### LED Channels Not Working

1. Verify GPIO pin assignments match your hardware
2. Check PWM frequency is within acceptable range for your LEDs
3. Test individual channels using Home Assistant light controls

### Fan Not Controlling

1. Check if thermal control is enabled
2. Verify PID parameters are reasonable (Kp=10, Ki=0.5, Kd=1.0 are good defaults)
3. Monitor temperature sensor readings

## Support

For issues specific to the ESPHome conversion:
- Create an issue in the [Borneo GitHub repository](https://github.com/borneo-iot/borneo/issues)
- Tag with `esphome` label

For general ESPHome questions:
- Visit the [ESPHome documentation](https://esphome.io/)
- Join the [ESPHome Discord](https://discord.gg/KhAMKrd)

## License

The ESPHome configuration maintains the same dual-licensing as the original Borneo project:
- Software/Firmware: GPL-3.0+
- Hardware: CERN-OHL-S-2.0

See [LICENSE](../LICENSE) and [LICENSE-HARDWARE](../LICENSE-HARDWARE) for details.

## Credits

- Original ESP-IDF firmware by Wei Li (李维)
- ESPHome conversion maintains feature parity with the original design
- Built on the excellent [ESPHome](https://esphome.io/) project
