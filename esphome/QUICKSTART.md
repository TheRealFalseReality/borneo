# Quick Start Guide - Borneo ESPHome

Get your Borneo LED controller connected to Home Assistant in under 30 minutes!

## Prerequisites

- [ ] Borneo BLC06MK1 LED controller board
- [ ] USB-C cable for initial programming
- [ ] Home Assistant installed and running
- [ ] WiFi network (2.4GHz)
- [ ] Computer with Python 3.7+

## Step 1: Install ESPHome (2 minutes)

### Option A: Via Home Assistant (Recommended)
1. Open Home Assistant
2. Go to **Settings** → **Add-ons** → **Add-on Store**
3. Search for "ESPHome"
4. Click **Install**

### Option B: Via Command Line
```bash
pip3 install esphome
```

## Step 2: Prepare Configuration (5 minutes)

1. **Download configuration files** from this repository:
   ```bash
   git clone https://github.com/borneo-iot/borneo.git
   cd borneo/esphome
   ```

2. **Create your secrets file**:
   ```bash
   cp secrets.yaml.template secrets.yaml
   ```

3. **Edit secrets.yaml** with your information:
   ```yaml
   wifi_ssid: "YourWiFiName"
   wifi_password: "YourWiFiPassword"
   ap_password: "FallbackPassword123"
   ota_password: "OTAPassword123"
   latitude: "37.7749"    # Your latitude
   longitude: "-122.4194" # Your longitude
   ```

4. **Generate API key**:
   ```bash
   esphome encryption-key-generate
   ```
   Copy the output into `secrets.yaml` as `api_encryption_key`

## Step 3: Validate Configuration (1 minute)

```bash
esphome config borneo-lyfi.yaml
```

You should see: `INFO Configuration is valid!`

## Step 4: Initial Flash via USB (10 minutes)

1. **Connect your board** via USB-C
2. **Put board in download mode** (if needed - depends on your board)
3. **Flash the firmware**:
   ```bash
   esphome run borneo-lyfi.yaml
   ```
4. **Select your USB port** when prompted
5. **Wait for compilation and upload** (5-10 minutes first time)

## Step 5: Add to Home Assistant (2 minutes)

1. **Wait for device to connect** to WiFi (status LED should be solid)
2. **Open Home Assistant**
3. **Go to** Settings → Devices & Services
4. **ESPHome integration** should auto-discover your device
5. **Click "Configure"** and enter your API encryption key from secrets.yaml
6. **Done!** Your device is now integrated

## Step 6: Test Basic Functions (5 minutes)

### Turn on a light
1. Go to **Settings** → **Devices & Services** → **ESPHome**
2. Click on "Borneo Aquarium LED"
3. Toggle **"LED Channel 0 (Red)"** to ON
4. Adjust brightness slider

### Check temperature
- Find **"Board Temperature"** sensor
- Should show current board temperature in °C

### Test fan
1. Enable **"Enable Thermal Control"** switch
2. Adjust **"Target Temperature"** to below current temp
3. Fan should start spinning

## Step 7: Create Your First Automation (5 minutes)

Let's create a simple sunrise simulation:

1. Go to **Settings** → **Automations & Scenes**
2. Click **"+ Create Automation"**
3. Click **"Create new automation"** (not from blueprint)
4. Configure trigger:
   - Type: **Time**
   - At: **08:00:00**
5. Add action:
   - Type: **Call service**
   - Service: **Light: Turn on**
   - Target: Select all LED channels
   - Data:
     ```yaml
     brightness_pct: 100
     transition: 3600  # 1 hour gradual increase
     ```
6. **Save** as "Morning Aquarium Sunrise"

## You're Done! 🎉

Your Borneo LED controller is now integrated with Home Assistant!

## What's Next?

### Learn More
- 📖 [Full README](README.md) - Complete documentation
- 🔄 [Migration Guide](MIGRATION.md) - Coming from ESP-IDF?
- 🤖 [Automation Examples](home-assistant-automations.yaml) - Copy & paste ready automations
- 📊 [Summary](SUMMARY.md) - What's included in this conversion

### Customize Your Setup

#### Create a Dashboard
1. Go to **Overview** in Home Assistant
2. Click **"Edit Dashboard"**
3. Add a new card:
   ```yaml
   type: entities
   title: Aquarium LED Control
   entities:
     - light.led_channel_0_red
     - light.led_channel_1_green
     - light.led_channel_2_blue
     - sensor.board_temperature
     - fan.cooling_fan
   ```

#### Set Up Temperature Alerts
1. Go to **Settings** → **Automations**
2. Create automation with trigger:
   - Type: **Numeric state**
   - Entity: **sensor.board_temperature**
   - Above: **50**
3. Add action to send notification

#### Configure Daily Schedule
Check [`home-assistant-automations.yaml`](home-assistant-automations.yaml) for ready-to-use examples:
- Morning sunrise (8 AM)
- Midday peak (12 PM)
- Evening sunset (6 PM)
- Night moonlight (10 PM)

## Troubleshooting Quick Fixes

### Device won't connect to WiFi
```yaml
# In borneo-lyfi.yaml, add:
wifi:
  ssid: "YourSSID"
  password: "YourPassword"
  
  # Add this to see more debug info:
  enable_on_boot: true
  
  # Optional: Use static IP
  manual_ip:
    static_ip: 192.168.1.100
    gateway: 192.168.1.1
    subnet: 255.255.255.0
```

### Can't find device in Home Assistant
1. Check device logs:
   ```bash
   esphome logs borneo-lyfi.yaml
   ```
2. Look for "API connection successful"
3. If not, manually add integration:
   - **Settings** → **Integrations** → **+ Add Integration**
   - Search "ESPHome"
   - Enter device IP address

### Temperature reading is wrong
The NTC thermistor calibration might need adjustment. In `borneo-lyfi.yaml`, find the `ntc` sensor and adjust:
```yaml
calibration:
  b_constant: 3950  # Try 3435 or 4200
  reference_temperature: 25°C
  reference_resistance: 10kOhm
```

### LED channels don't work
1. Check if you're in Manual mode:
   ```yaml
   select.led_control_mode → "Manual"
   ```
2. Set brightness values:
   ```yaml
   number.manual_ch0_brightness → 2048
   ```

## Support

Need help? Here's where to go:

1. **Check the logs first**:
   ```bash
   esphome logs borneo-lyfi.yaml
   ```

2. **Search existing issues**: [GitHub Issues](https://github.com/borneo-iot/borneo/issues)

3. **Ask the community**:
   - [Borneo Discord](https://discord.gg/EFJTm7PpEs)
   - [GitHub Discussions](https://github.com/borneo-iot/borneo/discussions)

4. **Open a new issue**: Include:
   - ESPHome version
   - Home Assistant version
   - Board model
   - Relevant logs
   - What you've tried

## Success Stories

Once you're up and running, share your setup:
- Post in [GitHub Discussions](https://github.com/borneo-iot/borneo/discussions)
- Join the [Discord community](https://discord.gg/EFJTm7PpEs)
- Tag `#BorneoLED` on social media

Happy reefing! 🐠🪸
