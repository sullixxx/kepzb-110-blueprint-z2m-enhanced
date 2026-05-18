# PIN-Keypads for Zigbee2MQTT (Complete with Night Mode & Entry Delay)

A comprehensive Home Assistant blueprint that provides full synchronization between Zigbee2MQTT KEPZB-110 keypads and alarm control panels, including complete night mode support and entry delay beeping.

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://github.com/kekstyle/KEPZB-110-BluePrint-Z2M/blob/main/blueprint.yaml)

## 🚀 Features

### Core Functionality
- **Bidirectional sync** between keypad and Home Assistant alarm panel
- **All arming modes**: Away, Home, Night, and Disarm
- **No PIN required for arming** - just press the button (you can activate this functionality directly from alarmo)
- **PIN required for disarming** - secure authentication (user/PIN added previously in alarmo)
- **Entry delay beeping** - gentle reminder when entering through front door to turn disarm
- **Custom PIN actions** - trigger special automations with wrong PINs
- **Panic/SOS support** - emergency button functionality

### Security Features
- **PIN validation** for disarming (alarmo user/PIN)
- **Invalid code handling** with keypad feedback and notify on phone
- **Custom actions** for specific PIN codes
- **Entry sensor monitoring** with configurable delay beeping

## 📱 Supported Devices

### Supported Keypads
- **Frient KEPZB-110** ✅ (Primary target)
- **Develco KEYZB-110** ✅ (Same hardware as Frient)

*Other keypads may work but are not officially supported. The MQTT commands and entry delay functionality are specifically designed for the KEPZB-110/KEYZB-110 models.*

### Compatible Alarm Panels
- **Alarmo** 

## 🔧 Installation

### Prerequisites
- Home Assistant with Zigbee2MQTT
- Zigbee keypad paired to Zigbee2MQTT (see pairing instructions below)
- Alarm control panel configured in Home Assistant

### Pairing Your Frient KEPZB-110 Keypad

#### For Brand New Keypad
1. **Install batteries** in the keypad (4x AA batteries)
2. **Open Zigbee2MQTT** web interface
3. **Enable pairing mode**:
   - Click "Permit join (All)" button
   - Set timeout to 60+ seconds
4. **Activate keypad pairing**:
   - The keypad should automatically enter pairing mode when powered on for the first time
   - Look for blinking LED indicators
   - Wait for Zigbee2MQTT to detect the device

#### For Previously Used Keypad
1. **Reset the keypad**:
   - Remove the keypad from the wall/mounting
   - Locate the small **reset button on the back** of the device
   - Press and hold the reset button for **10+ seconds** while batteries are installed
   - Release the button - LED should start blinking
2. **Enable pairing in Zigbee2MQTT**:
   - Click "Permit join (All)" button
   - Set timeout to 60+ seconds
3. **Complete pairing**:
   - The keypad should now appear in Zigbee2MQTT devices
   - Assign a friendly name (e.g., "Entrance_Keypad")

#### Troubleshooting Pairing
- **No response**: Check battery installation and charge level
- **Won't reset**: Hold reset button longer (15-20 seconds)
- **Still not pairing**: Try moving keypad closer to Zigbee coordinator
- **Pairing fails**: Remove batteries for 30 seconds, reinstall, and try again

### Setup Steps

1. **Download the Blueprint**
   - Save the blueprint YAML file to your Home Assistant
   - Go to Settings → Automations & Scenes → Blueprints
   - Import the blueprint

2. **Find Your Keypad's MQTT Topics**
   - Open **Zigbee2MQTT** web interface
   - Go to the **Devices** tab
   - Find your keypad device and click on it
   - Note the **Friendly Name** (e.g., `Entrance_Keypad`)
   - Your MQTT topics will be:
     - **State Topic**: `zigbee2mqtt/Entrance_Keypad` 
     - **Set Topic**: `zigbee2mqtt/Entrance_Keypad/set`
   
   **Alternative method via MQTT Explorer:**
   - Use an MQTT client/explorer
   - Look for topics under `zigbee2mqtt/` 
   - Find your keypad's friendly name in the topic list

3. **Create Automation from Blueprint**
   - Click "Create Automation"
   - Fill in the required configuration (see below)

4. **Configure Your Keypad**
   - Note your keypad's friendly name in Zigbee2MQTT
   - Ensure it's properly paired and responding

## ⚙️ Configuration

### Required Settings

| Setting | Example | Description |
|---------|---------|-------------|
| **MQTT State Topic** | `zigbee2mqtt/Entrance_Keypad` | Your keypad's MQTT topic (zigbee2mqtt + friendly name) |
| **MQTT Set Topic** | `zigbee2mqtt/Entrance_Keypad/set` | State topic + `/set` |
| **Control Panel** | `alarm_control_panel.home_alarm` | Your alarm panel entity |
| **Notify service** | `alarm_control_panel.home_alarm` | default notify.notify (can add an other notify service)|
### Optional Settings

| Setting | Default | Description |
|---------|---------|-------------|
| **Entry Delay Sensors** | None | Doors that trigger entry delay beep |
| **Entry Delay Duration** | 30 seconds | How long entry beep continues |
| **Custom PIN 1** | `0000` | Special PIN for custom action 1 |
| **Custom PIN 2** | `1111` | Special PIN for custom action 2 |

### Action Settings (All Optional)
- Action Arming
- Action Armed Home
- Action Armed Away  
- Action Armed Night
- Action Disarmed
- Action Pending
- Action Panic
- Action Custom 1
- Action Custom 2

## 🎯 Usage Guide

### Arming the System
**No PIN required if it's not checked on alarmo settings** - just press the button:
- **Away Button** → Arms all zones
- **Home Button** → Arms day zones only
- **Night Button** → Arms night zones only

### Disarming the System
**PIN required** for security:
1. Enter your PIN code (e.g., `1234`)
2. Press **Disarm** button
3. System disarms if PIN is correct

### Entry Delay Behavior
When you've configured entry sensors:
1. **System is armed** (any mode)
2. **Open entry door** → Keypad starts gentle beeping
3. **Walk to keypad** and disarm within the delay period
4. **Enter PIN + Disarm** → Beeping stops, system disarms

### Emergency/Panic
- **Press and hold SOS button** → Triggers panic action
- Customize the panic action in blueprint configuration

### Custom PIN Actions
Configure special PINs that trigger custom automations:
- Enter custom PIN + any arm button → Triggers your custom action
- Useful for guest codes, silent alarms, etc.

## 🔧 Advanced Configuration

### Entry Delay Sensors
Select sensors that should trigger entry delay beeping:
```yaml
# Example sensors to select:
- binary_sensor.front_door
- binary_sensor.back_door
- binary_sensor.garage_door
```

### Custom Actions Examples

**Custom PIN 1 - Guest Mode:**
```yaml
# Trigger when PIN "9999" is entered
action:
  - service: light.turn_on
    target:
      entity_id: light.welcome_lights
  - service: notify.mobile_app
    data:
      message: "Guest has arrived"
```

**Panic Action - Silent Alarm:**
```yaml
action:
  - service: notify.mobile_app
    data:
      message: "EMERGENCY: Panic button pressed"
      title: "🚨 PANIC ALERT"
  - service: camera.record
    target:
      entity_id: camera.front_door
```

## 🚨 Troubleshooting

### Keypad Not Responding
1. **Check MQTT topics** - Verify in Zigbee2MQTT frontend
2. **Test MQTT manually** - Send commands via Developer Tools
3. **Check keypad battery** - Low battery affects responsiveness
4. **Verify pairing** - Re-pair if necessary

### Arming Works But Disarming Doesn't
1. **Check PIN configuration** - Ensure it matches what you're entering
2. **Test alarm panel** - Try manual disarm from HA interface
3. **Check automation traces** - Look for errors in automation history

### No Entry Delay Beeping
1. **Verify sensor selection** - Check binary sensor entities
2. **Test sensor triggers** - Watch in Developer Tools → States
3. **Check automation traces** - Verify trigger conditions are met
4. **Battery check** - Low keypad battery affects beeping

### Invalid Code Behavior
If you enter wrong PIN:
- Keypad shows error feedback
- System remains armed
- Try again with correct PIN

## 📋 MQTT Commands Reference

### Keypad to Home Assistant
```json
{
  "action": "arm_all_zones",
  "action_code": "1234",
  "action_zone": 23,
  "action_transaction": 99
}
```

### Home Assistant to Keypad
```json
{
  "arm_mode": {
    "mode": "arm_all_zones"
  }
}
```

### Valid Modes
- `disarm`
- `arm_day_zones` (Home)
- `arm_night_zones` (Night)
- `arm_all_zones` (Away)
- `entry_delay` (Entry beeping)
- `exit_delay` (Exit countdown)
- `invalid_code`
- `not_ready`
- `in_alarm`

## 🔄 State Synchronization

The blueprint maintains perfect sync between keypad and Home Assistant:

| Home Assistant State | Keypad Display | Description |
|---------------------|----------------|-------------|
| `disarmed` | Disarmed | System off |
| `armed_home` | Day Zones | Home mode |
| `armed_night` | Night Zones | Night mode |
| `armed_away` | All Zones | Away mode |
| `arming` | Exit Delay | Countdown timer |
| `pending` | Entry Delay | Entry countdown + beep |

This blueprint is forked from https://github.com/michaeln64/KEPZB-110-BluePrint-Z2M thanks to him for doing a great job !
