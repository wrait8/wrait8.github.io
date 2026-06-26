---
date: 2026-06-28
tags: Wi-Fi Hacking, Hardware
icon: 🍍
cover: https://media.printables.com/media/prints/414795/images/3442031_8f42ed85-56c9-47aa-866b-b8e509e7ae9e/thumbs/inside/1280x960/jpeg/img_7380.webp
---


A Pwnagotchi is a Raspberry Pi Zero device that uses machine learning to capture Wi-Fi handshakes more efficiently. It's like a Tamagotchi pet that needs to capture WPA handshakes to survive, with a cute face that shows its mood on a small e-ink display.

## What is a Pwnagotchi?

A Pwnagotchi is an AI-powered Wi-Fi capturing tool that:
- **Automatically captures WPA/WPA2 handshakes** from nearby networks
- **Uses machine learning** to optimize capture efficiency
- **Has a cute personality** - its mood changes based on how many handshakes it captures
- **Has a built-in web UI** for viewing captured handshakes
- **Syncs with other Pwnagotchis** to share captured data

## Hardware Required

### Essential Components
- Raspberry Pi Zero W or Zero 2 W (~$15-30)
- MicroSD card (16GB minimum) (~$10)
- Waveshare 2.13 inch e-ink display (~$20)
- Battery pack (5000mAh) (~$15)
- USB cable for power
- Optional: PiSugar or battery hat for clean power

### Optional Accessories
- GPS module for location tagging
- RTC (Real Time Clock) module
- RGB LED for status indication
- 3D printed case

## Software Setup

### Step 1: Download the Image

Download the official Pwnagotchi image from the releases page:
```
https://github.com/evilsocket/pwnagotchi/releases
```

### Step 2: Flash the SD Card

**On Windows:**
1. Download BalenaEtcher
2. Select the downloaded image
3. Select your SD card
4. Click "Flash!"

**On Linux:**
```bash
sudo dd if=pwnagotchi-*.img of=/dev/sdX bs=4M status=progress
```

### Step 3: Configure the Device

1. After flashing, the SD card will have a `boot` partition
2. Create a `config.toml` file in the boot partition:

```toml
# config.toml
main.name = "pwnagotchi"
main.lang = "en"
main.whitelist = [
    "YourHomeWiFi",  # Networks to ignore
    "GuestNetwork"
]

# Display configuration
ui.display.enabled = true
ui.display.type = "waveshare_2"

# AI/ML configuration
personality.advertise = true
personality.vibes = true

# Better capture settings
bettercap.handshake_threshold = 50
bettercap.device = "wlan0mon"

# Web UI settings
ui.web.enabled = true
ui.web.address = "0.0.0.0"
ui.web.port = 8080
```

### Step 4: Connect to Network

Create a file named `wpa_supplicant.conf` in the boot partition:

```ini
country=US
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
    ssid="YourWiFiName"
    psk="YourWiFiPassword"
    key_mgmt=WPA-PSK
}
```

### Step 5: Boot and Monitor

1. Insert the SD card into the Raspberry Pi Zero
2. Connect the e-ink display
3. Power on the device
4. The display will show a face and start scanning

## Accessing Your Pwnagotchi

### SSH Access
```bash
ssh pi@10.0.0.2
password: raspberry
```

### Web Interface
Open your browser and navigate to:
```
http://10.0.0.2:8080
```

### View Captured Handshakes
Handshakes are stored in:
```
/root/handshakes/
```

## Using Your Pwnagotchi

### Reading the Face
The Pwnagotchi's face changes based on its mood:

| Face | Meaning |
|------|---------|
| 😊 | Happy - recently captured handshake |
| 😴 | Bored - no networks nearby |
| 🤔 | Thinking - AI processing |
| 😡 | Angry - low battery or errors |
| 🔥 | On fire! - many handshakes captured |

### Understanding the Stats
- **Age**: How long the device has been running
- **Handshakes**: Total handshakes captured
- **Strength**: Current AI confidence
- **Networks**: Networks encountered

## Improving Performance

### AI Training Tips
1. **Multiple Pwnagotchis** work better together
2. **Place in high-traffic areas** for more captures
3. **Use a good antenna** for better range
4. **Keep it on a moving vehicle** to encounter more networks

### Battery Optimization
```toml
# Add to config.toml for longer battery life
main.sleep_time = 20
main.running = true
ui.display.enabled = true
ui.display.type = "waveshare_2"
```

## Troubleshooting

### No Display?
- Check display connections
- Verify display type in config
- Try different display driver

### No Handshakes?
- Check antenna connection
- Ensure WiFi is enabled
- Try different location with more networks
- Check that you're not too close to networks (need some distance for better capture)

### WiFi Not Working?
- Check wpa_supplicant.conf
- Verify network name and password
- Try: `sudo systemctl restart networking`

## Extending Your Pwnagotchi

### Add GPS Support
```toml
# Add to config.toml
gps.enabled = true
gps.device = "/dev/ttyACM0"
gps.baudrate = 9600
```

### Add LED Indicators
```toml
# Add to config.toml
ui.led.enabled = true
ui.led.pin = 18
```

### Sync with Other Pwnagotchis
When multiple Pwnagotchis are in range, they automatically sync handshakes! This is called "memes" - they share captured data with each other.

## Advanced Configuration

### Custom Plugins
Create custom plugins in `/usr/local/lib/python3.7/dist-packages/pwnagotchi/plugins/`:

```python
# Example plugin structure
import pwnagotchi.plugins as plugins

class MyPlugin(plugins.Plugin):
    __author__ = 'Your Name'
    __version__ = '1.0.0'
    __name__ = 'my_plugin'
    
    def on_ready(self, agent):
        print("Plugin loaded!")
```

### Web API
Pwnagotchi exposes a REST API for automation:
```
http://10.0.0.2:8080/api/v1/status
```

## Security Considerations

> ⚠️ **WARNING**: Only use this device on networks you own or have explicit permission to test. Capturing handshakes from networks you don't own may be illegal in your jurisdiction.

### Best Practices
1. **Use only on authorized networks**
2. **Secure your captured handshakes**
3. **Don't share captured data publicly**
4. **Respect privacy and local laws**

## Fun Facts

- A "pwnagotchi" is a play on "Pwnagotchi" from Japanese "Tamagotchi"
- They have "vibes" - AI personality traits that affect behavior
- They "learn" from experience and get smarter over time
- Two Pwnagotchis can "bond" and share data

## Additional Resources

- [Official Pwnagotchi GitHub](https://github.com/evilsocket/pwnagotchi)
- [Pwnagotchi Community](https://discord.gg/pwnagotchi)
- [Raspberry Pi Zero Documentation](https://www.raspberrypi.com/documentation/computers/raspberry-pi-zero.html)
- [Bettercap Framework](https://www.bettercap.org/)
