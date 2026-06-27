![dump1090 Screenshot](https://www.rtl-sdr.com/wp-content/uploads/2013/05/rtlsdr.jpg)

dump1090 is a powerful, open-source Mode S decoder specifically designed for RTLSDR devices that allows you to receive and decode ADS-B (Automatic Dependent Surveillance–Broadcast) signals from aircraft. Written by Salvatore Sanfilippo (antirez) and released under the BSD license, this software transforms a cheap $20 RTL-SDR dongle into a professional-grade aircraft tracking system. With dump1090, you can track planes in real-time, view their positions on a map, and even contribute data to global flight tracking networks.

## What is dump1090?

dump1090 is a software-defined radio (SDR) application that:

- **Decodes Mode S and ADS-B signals** from aircraft at 1090 MHz
- **Displays real-time flight data** including callsign, altitude, speed, and heading
- **Provides a web interface** with Google Maps integration for visualizing aircraft positions
- **Corrects single-bit errors** using the 24-bit CRC checksum for more reliable decoding
- **Supports multiple data formats** including raw hex, AVR, and SBS1/BaseStation formats
- **Can run on various platforms** including Linux, Windows, and Raspberry Pi

## Key Features

### Robust Signal Decoding

dump1090 is renowned for its ability to decode weak signals, often providing better range than other ADS-B decoders. It features:

- **Single-bit error correction** - attempts to fix messages with one flipped bit by checking the CRC
- **Aggressive mode** - tolerates up to two demodulation errors for better weak signal detection
- **Brute-force ICAO decoding** - handles DF formats where the checksum is XORed with the ICAO address
- **DF11 and DF17 message decoding** - captures all essential aircraft data

### Network and Visualization

dump1090 includes powerful networking features:

- **Embedded HTTP server** - serves a web interface showing aircraft on Google Maps at `http://localhost:8080`
- **TCP servers on ports 30001, 30002, and 30003**:
  - **Port 30001** - raw input port for receiving data from other dump1090 instances
  - **Port 30002** - raw output for connecting to display software
  - **Port 30003** - SBS1/BaseStation format for feeding data to flight tracking networks

### Interactive Mode

dump1090 offers an "arcade-style" interactive terminal display that refreshes every second showing all recently detected aircraft with their altitude, flight number, and other key information.

## Installation Guide

### Linux Installation

```bash
# Clone the repository
git clone https://github.com/antirez/dump1090.git
cd dump1090

# Install dependencies
sudo apt-get install build-essential librtlsdr-dev

# Compile
make

# Optional: generate optimized local configuration
make wisdom.local
```

### Windows Installation

1. Download the Windows executable from [this GitHub repository](https://github.com/timseed/Dump1090_Windows)
2. Place `dump1090.exe` and `cygrtlsdr-0.dll` in a folder (e.g., `D:\ADSB\dump1090`)
3. Ensure RTL-SDR drivers are installed via Zadig
4. Open Command Prompt and run:
   ```cmd
   Dump1090.exe --interactive
   ```

### Raspberry Pi Installation

For a dedicated 24/7 aircraft tracker, a Raspberry Pi is the ideal platform:

```bash
# Install dependencies
sudo apt update
sudo apt install git cmake build-essential libusb-1.0-0-dev -y

# Install librtlsdr
git clone https://github.com/steve-m/librtlsdr.git
cd librtlsdr
mkdir build && cd build
cmake ..
make
sudo make install
sudo ldconfig

# Test your SDR
rtl_test

# Install dump1090
cd ~
git clone https://github.com/antirez/dump1090.git
cd dump1090
make
```

## Running dump1090

### Basic Usage

```bash
# Capture directly from RTL device
./dump1090

# Interactive mode with arcade-style display
./dump1090 --interactive

# Interactive with web interface enabled
./dump1090 --interactive --net

# View live map in browser: http://localhost:8080
```

### Advanced Options

```bash
# Disable error correction for raw data
./dump1090 --no-fix

# Use aggressive mode for weak signals
./dump1090 --aggressive

# Set antenna location for accurate positioning
./dump1090 --lat 43.6589 --lon -79.4529

# Decode from captured file
./dump1090 --ifile /path/to/sample.bin

# Network-only mode (hub)
./dump1090 --net-only

# Change web interface port
./dump1090 --net --web-port 8081
```

## Understanding the Output

### Interactive Mode Display

When running in interactive mode, dump1090 displays:

| Field | Description |
|-------|-------------|
| ICAO | Unique aircraft hex code |
| Flight | Callsign/flight number |
| Altitude | Current altitude in feet |
| Speed | Ground speed in knots |
| Heading | Direction in degrees |
| Latitude/Longitude | Position coordinates |

### Raw Message Format

Raw ADS-B messages appear as hex strings starting with `*` and ending with `;`:

```
*8D451E8B99019699C00B0A81F36E;
```

### SBS1/BaseStation Format (Port 30003)

For integration with other software, dump1090 outputs in the SBS1 format:

```
MSG,4,,,738065,,,,,,,,420,179,,,0,,0,0,0,0
MSG,3,,,738065,,,,,,,35000,,,34.81609,34.07810,,,0,0,0,0
```

## Feeding Data to Flight Tracking Networks

dump1090 integrates seamlessly with major flight tracking networks:

### FlightRadar24
```bash
sudo bash -c "$(wget -O - http://repo-feed.flightradar24.com/install_fr24_rpi.sh)"
```

### ADS-B Exchange
```bash
git clone https://github.com/adsbxchange/adsb-exchange.git
cd adsb-exchange
./install.sh
```

### FlightAware (PiAware)
```bash
sudo apt-get install piaware
sudo piaware-config -username YOUR_USERNAME
```

## Troubleshooting

### Device or Resource Busy Error

If you see "Error opening the RTLSDR device: Device or resource busy", you need to blacklist the DVB driver:

```bash
sudo nano /etc/modprobe.d/blacklist-rtl.conf
```

Add these lines:
```
blacklist dvb_usb_rtl28xxu
blacklist rtl2832
blacklist rtl2830
```

Then run:
```bash
sudo update-initramfs -u
sudo reboot
```

### Web Interface Not Loading

- Ensure you're using the `--net` flag
- Check firewall settings for port 8080
- Try accessing via `http://localhost:8080` or your Pi's IP address
- For Windows, ensure the JavaScript files have proper line endings

## Debug Mode

dump1090 includes a debug mode that visualizes signals:

```bash
./dump1090 --debug 1
```

This displays an ASCII-art representation showing the individual magnitude bars sampled at 2MHz, helping you understand the received signals and improve your antenna setup.

## The Importance of a Good Antenna

While dump1090 is excellent software, its performance is highly dependent on your antenna setup. Here's what you need to know:

### Basic Antenna Options

- **Quarter-Wave Ground Plane**: The easiest and most effective DIY option. Simply cut a wire to 69mm (quarter wavelength at 1090 MHz) and mount it vertically above four ground radials of the same length. This simple antenna can outperform the stock whip significantly.

- **Stock Antenna Modification**: The stock antenna from your RTL-SDR kit can be improved by shortening the whip to exactly 69mm. This often provides better reception than the full-length antenna.

### Advanced Antenna Tips

- **Placement Matters**: Mount your antenna as high as possible with a clear line of sight to the horizon. ADS-B signals are horizontally polarized, so vertical mounting is essential.

- **Collinear Antennas**: For maximum range, a collinear antenna with 4-8 elements provides high gain toward the horizon, ideal for receiving distant aircraft.

- **Filters and Amplifiers**: In urban areas, a 1090 MHz bandpass filter can significantly improve reception by blocking interference from cell towers and FM broadcast signals. A low-noise amplifier (LNA) can help overcome cable losses.

- **Coax Cable**: Use quality 50-ohm coax and keep the cable run as short as possible to minimize signal loss.

## Additional Resources

- [dump1090 GitHub Repository](https://github.com/antirez/dump1090)
- [FlightAware dump1090-fa fork](https://github.com/flightaware/dump1090)
- [ADS-B Exchange](https://adsbexchange.com)
- [RTL-SDR Documentation](https://www.rtl-sdr.com)

---

*dump1090 was written by Salvatore Sanfilippo (antirez) and released under the BSD three-clause license. This tutorial is for educational purposes only.
