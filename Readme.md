# Fronius Solar Monitor 🌞⚡

This Python script monitors solar power data from a **Fronius Inverter** and sends Telegram alerts when the battery is full or no longer full. It can also control smart plugs based on battery status.

## Table of Contents
- [Overview](#overview)
- [How it Works](#how-it-works)
- [System Requirements](#-system-requirements)
- [Installation & Setup](#-installation--setup)
- [Data Monitoring & API](#-data-monitoring--api)
- [Additional Resources](#-additional-resources)
- [Troubleshooting](#️-troubleshooting)
- [FAQ](#-faq)

## Overview

**Fronius Solar Monitor** helps you optimize your solar energy consumption by alerting you when your battery reaches full capacity or drops below it. Perfect for regions where excess energy fed to the grid isn't compensated beyond certain limits.
You can also make changes and use it as part of your larger home automation system.

The system can automatically control Tuya-compatible smart plugs based on battery status, allowing you to automatically power devices (like water heaters, AC units, or pool pumps) when excess solar energy is available.

### How it Works
1. **Fronius Inverter** provides real-time data via the **Solar API** (e.g., battery status, power consumption, etc.).
2. **Fronius Solar Monitor Script** retrieves this data using the API and evaluates the battery status.
3. Based on the readings, the script sends **Telegram notifications** (battery full or not) to the user.
4. **Alerts are customizable** based on frequency and consecutive checks, reducing false positives.
5. **Smart Plug Control** (optional) automatically manages connected devices based on battery status and/or PV production.


## 📋 System Requirements

- **Supported Inverters**: Fronius GEN24 series
- **Operating System**: Any Python-compatible OS (Linux recommended, Windows also ok)
- **Python**: 3.6 or higher
- **Network**: Local network access to Fronius inverter
- **Smart Plugs** (optional): Any Tuya-compatible smart plug


## 🔧 Installation & Setup

### Prerequisites

- Python 3.x
- A Fronius inverter with Solar API
- Telegram account
- Tuya-compatible smart plug (optional)

### Step 1: Install Dependencies

Install the required Python libraries using `pip`:

```bash
pip install requests  # Required for making API requests to the Fronius inverter
pip install tinytuya  # Required for smart plug control
```

> **Note**: The `tinytuya` library is only required if you plan to use the smart plug feature.

### Step 2: Enable the Fronius Solar API

The **Solar API** must be enabled on **Fronius GEN24** devices to avoid 404 errors.

#### How to Enable:

1. **Access the Fronius Web Interface**
   - Enter your Fronius inverter's IP address in a browser.
   - Find the IP through:
     - Your router's device list
     - The IP configured during installation
     - Network scanning tools
   
   ![Router](docs/router.jpg)

2. **Enable the API**
   - Navigate to **Communication → Solar API**
   - Toggle the **Solar API** setting to enabled.
   - Save your changes.
   
   ![Solar API](docs/pv.jpg)

### Step 3: Create a Telegram Bot

A more detailed tutorial is available [here](https://core.telegram.org/bots/tutorial#obtain-your-bot-token).

1. **Create Your Bot**
   - Open Telegram and search for **@BotFather**
   - Send `/newbot` and follow the instructions.
   - Save the **Bot Token** provided.

2. **Configure Chat ID**
   - For a **group chat**: Add your bot to a group.
   - For a **private chat**: Start a conversation with your bot.
   - Send any message to the bot.
   - Get your **Chat ID** by opening:
     ```
     https://api.telegram.org/bot<TOKEN>/getUpdates
     ```
   - **Note**: Group chat IDs are negative numbers; private chat IDs are positive.

   ![Response](docs/tgchatid.png)

### Step 4: Configure the Application
(more on the smart plug settings further down at **Setting Up TinyTuya**)
Rename `_config.json` to `config.json` and configure it with your settings:

```json
{
  "telegram_token": "6467835642:AAAAAl99Ue14-e2cPqF79KSdOol5-aTr123",
  "chat_id": "-1048737232455",
  "solar_api_ip": "192.168.1.131",
  "check_interval_min": 1,
  "consecutive_full_checks": 1,
  "consecutive_not_full_checks": 4,
  "language": "en",
  "smart_plug_mode": 1,
  "pv_threshold": 500,
  "smart_plug": {
    "enabled": true,
    "dev_id": "bfa03f282eff246980zuioe",
    "address": "192.168.1.230",
    "local_key": "?quej6(u$e<1<#<<",
    "version": 3.5
  }
}
```

#### Configuration Parameters:

| Parameter | Description | Recommended Value |
|-----------|-------------|-------------------|
| `telegram_token` | Your Telegram bot token | - |
| `chat_id` | Target chat ID for notifications | Negative for groups, positive for private chats |
| `solar_api_ip` | IP address of Fronius inverter | Local network IP of inverter |
| `check_interval_min` | Time between checks (minutes) | 1-5 |
| `consecutive_full_checks` | Readings needed before "battery full" alert | 1-3 |
| `consecutive_not_full_checks` | Readings needed before "battery not full" alert | 2-5 |
| `language` | Notification language | "en" or "de" |
| `smart_plug_mode` | Control mode for smart plug | 1 or 2 (see below) |
| `pv_threshold` | PV production threshold in watts | 500-1000 (depends on your devices) |
| `smart_plug` | Configuration for smart plug control | See below |

> **Note**: Higher `consecutive_checks` values reduce false alerts but increase notification delay.

> **Smart Plug Tip**: Any Tuya-compatible smart plug will work with this system. These are widely available on AliExpress for around $5, making it an affordable addition to your solar monitoring setup.

##### Setting Up TinyTuya

To find your device ID and local key for the smart plug configuration, you can use the TinyTuya library. Visit the official TinyTuya repository for detailed setup instructions:

[https://github.com/jasonacox/tinytuya](https://github.com/jasonacox/tinytuya)

The repository provides tools to scan your network for Tuya devices, obtain device IDs, and retrieve local keys needed for the configuration.

#### Smart Plug Configuration

The smart plug feature allows you to automatically control a Tuya-compatible smart plug based on selected control mode.

##### Smart Plug Control Modes:

1. **Mode 1: Battery Status Only** (Default)
   - The smart plug will turn ON when the battery is full
   - The smart plug will turn OFF when the battery is no longer full
   - This is the original behavior

2. **Mode 2: Battery Status + PV Production**
   - The smart plug will turn ON only when:
     - The battery is full AND
     - Current PV production is above the threshold set in `pv_threshold`
   - The smart plug will turn OFF when either condition is not met
   - This mode helps ensure devices only run when there's sufficient ongoing solar production

##### Smart Plug Parameters:

| Parameter | Description | Example |
|-----------|-------------|--------|
| `enabled` | Enable/disable smart plug functionality | `true` or `false` |
| `dev_id` | Device ID of your Tuya smart plug | bfa5c4d187d5ab1234abcd |
| `address` | IP address of your smart plug | 192.168.1.230 |
| `local_key` | Local key for device authentication | a1b2c3d4e5f6g7h8 |
| `version` | Protocol version | 3.5 |

#### Testing TinyTuya Setup

To test if your smart plug is configured correctly, you can use the included test script that toggles the smart plug ON and OFF every 5 seconds:

```bash
python test_smart_plug.py
```

This is useful for verifying your smart plug connection and configuration before running the main monitoring script.

### Step 5: Run the Monitor

#### Manual Execution

To run the script manually:

```bash
python solar_monitor.py
```


#### Automatic Startup on Raspberry Pi (DietPi)

For a headless setup on Raspberry Pi running DietPi or (maybe) also other Linux distributions:

1. **Install DietPi**: Download from [dietpi.com](https://dietpi.com/docs/install/).

2. **Access via SSH in Terminal**:
   ```bash
   ssh root@192.168.1.132  # Replace with your Pi's IP
   ```

3. **Install Git and Python** via `dietpi-software` (or your package manager if using a different Linux distribution).
   
4. **Clone the repository** and navigate to the project directory:

   ```bash
   git clone https://github.com/Persie0/Fronius-Solar-Monitor.git
   cd Fronius-Solar-Monitor
   ```

5. **Verify the project directory**:
   To ensure you're in the correct directory, run the following command:
   
   ```bash
   pwd
   ```
   
   If the output is **not** `/root/Fronius-Solar-Monitor`, you will need to update the `PROJECT_DIR` variable in the setup configuration. Edit the script to reflect the correct path.

   Example:
   ```bash
   nano /otherpath/Fronius-Solar-Monitor/setup_autostart.sh
   ```

   Update the line:
   ```bash
   PROJECT_DIR="/otherpath/Fronius-Solar-Monitor"
   ```
Then save with Ctrl+O, Enter and Ctrl+X.

6. **Setup Auto-Start**:
   Simply run the setup script included in the repository:
   ```bash
   sudo bash setup_autostart.sh 
   ```

7. **Configure the bot**:
   Edit the `config.json` file to include your bot's token, chat ID, and inverter's IP address.
   ```bash
   nano config.json
   ```
   then copy and paste your configured config.json into it. Then Ctrl+O, Enter and Ctrl+X.

8. **Restart to test**:
   ```bash
   sudo systemctl restart solar_monitor
   ```
   now you should already get a TG notification. 
   > **Note**: If you did not get an notification, to check for errors, run:
   ```bash
   tail -f /root/Fronius-Solar-Monitor/solar_monitor.log
   ```
   And if you really want to test it run:
   ```bash
   sudo reboot
   ```

8. **Update to latest**:
To update to the latest version run in the Fronius-Solar-Monitor folder:
   ```bash
   git pull origin main
   ```
   After updating, restart the service:
   ```bash
   sudo systemctl restart solar_monitor
   ```


---

## 📡 Data Monitoring & API

### Example API Request

To verify connectivity with your Fronius inverter, you can test the API endpoint directly in your web browser:
(Replace *192.168.1.131* with your inverter's IP)
```
http://192.168.1.131/solar_api/v1/GetPowerFlowRealtimeData.fcgi
```
### Sample API Response:

```json
{
  "Body": {
    "Data": {
      "Inverters": {
        "1": {
          "Battery_Mode": "battery full",
          "P": 4201.64,
          "SOC": 100
        }
      },
      "Site": {
        "P_Grid": 740.7,
        "P_Load": -4942.34,
        "P_PV": 4298.14,
        "rel_Autonomy": 85.01,
        "rel_SelfConsumption": 100
      }
    }
  },
  "Head": {
    "Status": { "Code": 0 },
    "Timestamp": "2025-02-18T14:50:13+00:00"
  }
}
```

### Data Visualization

#### Web Interface
![Web Interface](docs/webui.jpg)

#### API Response Values Explained

| Field               | Description |
|---------------------|-------------|
| `Battery_Mode`      | Battery state (e.g., "battery full") |
| `SOC`               | Battery state of charge (percentage) |
| `P_Grid`            | Power from/to the grid (positive = importing, negative = exporting) |
| `P_Load`            | Power consumption (negative values mean energy usage) |
| `P_PV`              | Solar power generation |
| `rel_Autonomy`      | Percentage of energy used from your own sources |
| `rel_SelfConsumption`| Percentage of solar energy used locally |

---

## 📚 Additional Resources

For detailed API documentation:

- [fronius-json-tools](https://github.com/akleber/fronius-json-tools)
- [Fronius Solar API Documentation (PDF)](docs/docs.pdf)

---

## 🛠️ Troubleshooting

| Issue | Solution |
|-------|----------|
| 404 Error when accessing the Solar API | Ensure the Solar API is enabled on your Fronius inverter and you're using the correct IP. |
| No Telegram message received | Double-check the **Bot Token** and **Chat ID** in `config.json`. Ensure the bot is correctly added to the group or conversation. |
| Data not updating | Make sure the inverter's IP is correct and accessible from your Python script. Test with `ping` or use `curl` to manually check if the API is responding. |
| Script Crashes | Run the script in debug mode by adding `-v` for more verbose logging. Check the logs in `/var/log/solar_monitor.log` for detailed errors. |

---


## 💡 FAQ

### **Q: Can I use the Solar Monitor with other inverters or smart plugs?**
A: Currently, the Solar Monitor is designed for **Fronius GEN24** inverters and Tuya-compatible smart plugs. However, you can adapt the code for other devices that provide API access:

For other inverters:
- Check if your inverter model has an available API
- Find the API documentation and endpoint structure
- Use the API URL and response format along with the solar_monitor.py script to modify the code (you can use AI tools like ChatGPT to help with the conversion)

For other smart plugs:
- Verify if your smart plug has an API or protocol that can be accessed via Python
- Find the required libraries and documentation for your smart plug
- Modify the smart plug integration code accordingly (AI tools can help with this adaptation)

---

**Happy Solar Monitoring! 🌞⚡**

