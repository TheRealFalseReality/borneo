# Migration Guide: ESP-IDF to ESPHome

This guide will help you migrate from the original ESP-IDF firmware to the ESPHome implementation for Home Assistant integration.

## Overview

The ESPHome version provides seamless integration with Home Assistant while maintaining the core functionality of the Borneo LED controller. While some advanced features move from firmware to Home Assistant automations, you gain:

- Native Home Assistant integration
- Easy configuration via YAML
- Visual control through Home Assistant UI
- Powerful automation capabilities
- Regular updates through ESPHome project

## Pre-Migration Checklist

Before migrating, document your current settings:

- [ ] LED channel brightness levels (0-4095 for each of 6 channels)
- [ ] PWM frequency (default: 19kHz)
- [ ] Brightness correction method (Linear/Log/Exp/Gamma/CIE1931)
- [ ] Thermal management settings:
  - [ ] Target temperature
  - [ ] PID parameters (Kp, Ki, Kd)
  - [ ] Fan control settings
- [ ] LED schedules (times and brightness levels)
- [ ] Sun simulation settings (if used):
  - [ ] Geographic coordinates
  - [ ] Timezone offset
- [ ] Acclimation settings (if active):
  - [ ] Start date
  - [ ] Duration in days
  - [ ] Starting brightness percentage
- [ ] Network settings (WiFi SSID/password)

## Migration Steps

### Step 1: Install ESPHome

On your computer, install ESPHome:

```bash
pip3 install esphome
```

Or use the Home Assistant Add-on:
1. Navigate to **Settings → Add-ons → Add-on Store**
2. Search for "ESPHome"
3. Click Install

### Step 2: Prepare Configuration

1. Copy the ESPHome configuration files:
   ```bash
   cd esphome
   cp secrets.yaml.template secrets.yaml
   ```

2. Edit `secrets.yaml` with your information:
   ```yaml
   wifi_ssid: "YourWiFiSSID"
   wifi_password: "YourWiFiPassword"
   api_encryption_key: "generate_with_esphome"
   ota_password: "YourSecurePassword"
   latitude: "your_latitude"
   longitude: "your_longitude"
   ```

3. Generate API encryption key:
   ```bash
   esphome encryption-key-generate
   ```
   Copy the output to `secrets.yaml`

### Step 3: Validate Configuration

Check that the YAML is valid:

```bash
cd esphome
esphome config borneo-lyfi.yaml
```

Fix any errors before proceeding.

### Step 4: Initial Flash (USB)

For the first flash, you need a USB connection:

1. Connect ESP32-C3 to your computer via USB
2. Put the device in download mode if necessary (depends on your board)
3. Run:
   ```bash
   esphome run borneo-lyfi.yaml
   ```
4. Select the USB serial port when prompted
5. Wait for compilation and upload (5-10 minutes)

### Step 5: Verify Basic Functionality

After flashing:

1. Check logs for successful boot:
   ```bash
   esphome logs borneo-lyfi.yaml
   ```

2. Verify connectivity:
   - Status LED should be solid (connected)
   - Device should appear in Home Assistant

3. Test individual LED channels from Home Assistant

### Step 6: Configure Home Assistant Integration

1. Go to **Settings → Devices & Services**
2. ESPHome integration should auto-discover your device
3. Click "Configure" and enter the API encryption key
4. The device and all entities will be added

### Step 7: Restore Settings

Transfer your documented settings:

#### LED Settings
- Set brightness correction method: `select.brightness_correction`
- Configure control mode: `select.led_control_mode`

#### Thermal Settings
- Target temperature: `number.target_temperature`
- PID parameters:
  - `number.pid_kp`
  - `number.pid_ki`
  - `number.pid_kd`
- Enable thermal control: `switch.enable_thermal_control`

#### Channel Brightness (Manual Mode)
Set each channel's brightness using:
- `number.manual_ch0_brightness` through `number.manual_ch5_brightness`

### Step 8: Recreate Schedules as Automations

Convert your ESP-IDF schedules to Home Assistant automations. See `home-assistant-automations.yaml` for examples.

Example: Morning lighting at 8 AM
```yaml
automation:
  - alias: "Aquarium Morning"
    trigger:
      - platform: time
        at: "08:00:00"
    action:
      - service: number.set_value
        target:
          entity_id: number.manual_ch0_brightness
        data:
          value: 2048  # 50% brightness
      # Repeat for other channels...
```

### Step 9: Set Up Sun Simulation (Optional)

If you used sun simulation:

1. Enable sun tracking: `switch.enable_sun_simulation`
2. Verify coordinates in `secrets.yaml`
3. Create sunrise/sunset automations (see examples)

### Step 10: Configure Acclimation Mode (Optional)

If you need acclimation:

1. Enable acclimation: `switch.enable_acclimation_mode`
2. Create automation to gradually increase brightness over days
3. Use `input_number` helper to track progress

## Feature Mapping

### Direct Feature Equivalents

| ESP-IDF Feature | ESPHome Equivalent | Notes |
|----------------|-------------------|-------|
| 6-channel PWM | `light.led_channel_*` | Same hardware control |
| Manual brightness | `number.manual_ch*_brightness` | 0-4095 range preserved |
| Temperature sensor | `sensor.board_temperature` | NTC thermistor |
| Fan control | `fan.cooling_fan` | PID-controlled |
| OTA updates | ESPHome OTA | Via Home Assistant |
| WiFi provisioning | ESPHome captive portal | Initial setup |

### Features Moving to Home Assistant

| ESP-IDF Feature | Home Assistant Implementation |
|----------------|------------------------------|
| LED schedules | Time-based automations |
| Sun simulation | Sun integration + automations |
| Acclimation mode | Multi-day automation scripts |
| Cloud effects | Random brightness automations |
| CoAP/CBOR API | Native ESPHome API |

### Configuration Differences

| Setting | ESP-IDF | ESPHome |
|---------|---------|---------|
| Storage | NVS (flash) | Home Assistant database |
| Interface | Custom Flutter app | Home Assistant UI |
| Protocol | CoAP/CBOR | Native API + MQTT (optional) |
| Configuration | Build-time (Kconfig) | Runtime (YAML) |

## Troubleshooting Migration Issues

### Issue: Device Won't Flash

**Symptoms**: Flash fails, timeout errors

**Solutions**:
1. Ensure USB cable supports data (not charge-only)
2. Press and hold BOOT button during power-on (if available)
3. Try slower baud rate: add `--baud 115200` to flash command
4. Use different USB port or cable

### Issue: WiFi Won't Connect

**Symptoms**: Device boots but can't connect to WiFi

**Solutions**:
1. Double-check WiFi credentials in `secrets.yaml`
2. Verify WiFi is 2.4GHz (ESP32-C3 doesn't support 5GHz)
3. Check if SSID has special characters - use quotes
4. Try temporary hotspot mode to access web interface

### Issue: Temperature Reading is Wrong

**Symptoms**: Temperature shows unrealistic values

**Solutions**:
1. Verify NTC thermistor type matches configuration
2. Adjust `b_constant` in YAML (common values: 3950, 3435, 4200)
3. Check `reference_resistance` (10kOhm is standard)
4. Verify pull-up resistor value (4.7kOhm from board config)

### Issue: LED Channels Don't Respond

**Symptoms**: Channels stay off or don't change brightness

**Solutions**:
1. Verify GPIO assignments match your board
2. Check if Manual mode is active: `select.led_control_mode`
3. Test with Home Assistant light controls directly
4. Check logs for hardware errors

### Issue: Fan Doesn't Control Temperature

**Symptoms**: Temperature rises, fan doesn't start

**Solutions**:
1. Enable thermal control: `switch.enable_thermal_control`
2. Lower target temperature to trigger fan
3. Check PID parameters are reasonable
4. Verify fan PWM GPIO is correct
5. Test fan manually through `fan.cooling_fan`

### Issue: Can't Find Device in Home Assistant

**Symptoms**: ESPHome integration doesn't discover device

**Solutions**:
1. Check device logs: `esphome logs borneo-lyfi.yaml`
2. Verify API encryption key matches
3. Check Home Assistant and device are on same network
4. Manually add: **Settings → Integrations → Add Integration → ESPHome**
5. Enter device IP address manually

## Rolling Back to ESP-IDF

If you need to revert to ESP-IDF firmware:

1. Rebuild ESP-IDF firmware:
   ```bash
   cd fw/lyfi
   idf.py build
   ```

2. Flash via USB:
   ```bash
   idf.py -p /dev/ttyUSB0 flash
   ```

3. Your previous NVS settings should be preserved (unless you did factory reset)

## Performance Comparison

### Memory Usage
- **ESP-IDF**: ~150KB RAM, custom protocol stack
- **ESPHome**: ~100KB RAM, optimized for Home Assistant

### CPU Usage
- Both implementations use similar CPU for PWM and PID control
- ESPHome has slightly lower overhead (no CoAP server)

### Network Latency
- **ESP-IDF CoAP**: ~50-100ms command response
- **ESPHome API**: ~20-50ms command response (local network)

### Update Frequency
- **ESP-IDF**: Manual updates, project-specific
- **ESPHome**: Regular updates from ESPHome project

## Advanced Migration Topics

### Custom Brightness Correction LUTs

The ESP-IDF version includes custom lookup tables for CIE1931 and other corrections. To implement in ESPHome:

1. Extract LUT from `correction-lut.c`
2. Create ESPHome lambda function with LUT
3. Apply in `led_update_task` script

Example:
```yaml
lambda: |-
  // CIE1931 correction
  static const uint16_t cie1931_lut[] = { /* values from correction-lut.c */ };
  uint16_t corrected = cie1931_lut[(uint16_t)brightness];
```

### Power Monitoring

The ESP-IDF version includes power measurement on ADC channels. To add in ESPHome:

```yaml
sensor:
  - platform: adc
    pin: GPIO0  # Voltage measurement channel
    name: "LED Voltage"
    update_interval: 1s
    filters:
      - multiply: 16.242  # Calibration factor from ESP-IDF
      
  - platform: adc
    pin: GPIO1  # Current measurement channel
    name: "LED Current"
    filters:
      - multiply: 0.235  # Calibration factor from ESP-IDF
```

### Multiple Device Management

For managing multiple Borneo controllers:

1. Copy `borneo-lyfi.yaml` for each device
2. Use unique device names:
   ```yaml
   substitutions:
     device_name: borneo-lyfi-tank1
     friendly_name: "Tank 1 LED"
   ```
3. Each device gets its own `secrets.yaml` or use shared secrets
4. All devices appear in Home Assistant separately

## Support During Migration

If you encounter issues during migration:

1. Check ESPHome documentation: https://esphome.io/
2. Review example automations: `home-assistant-automations.yaml`
3. Open GitHub issue with `esphome` tag
4. Join Discord for real-time help

## Post-Migration Checklist

After successful migration, verify:

- [ ] All 6 LED channels respond correctly
- [ ] Temperature sensor reads accurately
- [ ] Fan control maintains target temperature
- [ ] WiFi connection is stable
- [ ] OTA updates work
- [ ] Schedules run at correct times
- [ ] Home Assistant can control all entities
- [ ] Status LED indicates connection properly
- [ ] Web interface is accessible (optional)

Congratulations! Your Borneo LED controller is now running ESPHome and fully integrated with Home Assistant! 🎉

## Next Steps

- Explore [Home Assistant Automations](home-assistant-automations.yaml) for advanced control
- Set up dashboards in Lovelace for visual control
- Create scenes for different tank conditions
- Integrate with other Home Assistant devices
- Set up mobile notifications for alerts
