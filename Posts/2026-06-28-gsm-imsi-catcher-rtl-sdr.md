# The GSM IMSI Catcher: Your Passive Mobile Phone Tracker with RTL-SDR

![RTL-SDR Dongle](https://i.redd.it/7lditiy5ktx91.jpg)


An IMSI Catcher is a device that captures the unique International Mobile Subscriber Identity (IMSI) numbers from mobile phones in your vicinity. Using a cheap RTL-SDR dongle and open-source software like gr-gsm, you can build a *passive* IMSI catcher that simply listens to GSM (2G) network traffic. It's like a digital sniffer for phones - it quietly logs identifiers as nearby devices communicate with cell towers, revealing which phones are in the area without ever transmitting a signal.

## What is a Passive IMSI Catcher?

A passive IMSI catcher is a surveillance tool that:
- **Monitors 2G GSM frequencies** to capture phone identifiers
- **Records IMSI numbers** - unique SIM card identifiers that reveal country and carrier
- **Shows TMSI, LAC, and Cell ID** for network location tracking
- **Works without transmitting** - unlike "Stingray" devices, it just listens
- **Can be built for under $30** using cheap RTL-SDR hardware

## How GSM IMSI Catchers Work

### The Privacy Problem

When a mobile phone connects to a cellular network, it must identify itself to the network using its IMSI number. To protect privacy, the network assigns a temporary TMSI (Temporary Mobile Subscriber Identity) for all subsequent communications. The IMSI is only exposed during the initial connection handshake.

**The issue:** A passive IMSI catcher can listen for these handshakes and capture IMSI numbers whenever phones move between cell towers. Once you have someone's IMSI, you can potentially track their movements by monitoring for that identifier across multiple locations.

### Active vs. Passive IMSI Catchers

| Feature | Passive IMSI Catcher | Active IMSI Catcher ("Stingray") |
|---------|---------------------|----------------------------------|
| Transmits | No, just listens | Yes, impersonates a cell tower |
| Legality | Often legal to use | Usually illegal without authorization |
| Hardware needed | RTL-SDR (RX only) | HackRF/USRP (TX capable) |
| Can force connections? | No | Yes, forces phones to connect |
| Detects stationary phones? | No - only phones moving between towers | Yes |
| Cost | ~$30 | $600+ |

## Hardware Required

### Essential Components
- **RTL-SDR Dongle** (RTL2832U based) - ~$15-30
  - Can receive frequencies up to about 1.7GHz (GSM 900/1800 MHz)
  - The cheap TV tuner dongles work perfectly
- **Antenna** - the included telescopic antenna works, but a proper GSM antenna improves range
- **A Computer** running Linux (Ubuntu 20.04 LTS recommended)
  - 4GB+ RAM recommended for compiling gr-gsm
  - Can also run on Raspberry Pi 4 using DragonOS

### Optional Accessories
- **HackRF One** - for more advanced frequency scanning
- **GPS Module** - for location tagging captured IMSIs
- **Battery Pack** - for portable operation
- **Better antenna** - a GSM 900/1800 MHz external antenna

## Software Setup

### Step 1: Install GNU Radio and Dependencies

```bash
# Install core dependencies
sudo apt update
sudo apt install -y cmake autoconf libtool pkg-config build-essential \
  python-docutils libcppunit-dev swig doxygen liblog4cpp5-dev \
  gnuradio-dev gr-osmosdr libosmocore-dev liborc-0.4-dev swig \
  python3-numpy python3-scipy python3-scapy
```

### Step 2: Install gr-gsm

```bash
# Clone gr-gsm
git clone -b maint-3.8 https://github.com/velichkov/gr-gsm.git
cd gr-gsm
mkdir build
cd build
cmake ..
make -j 4
sudo make install
sudo ldconfig
```

### Step 3: Install IMSI-Catcher Script

```bash
# Clone the IMSI catcher tool
git clone https://github.com/Oros42/IMSI-catcher.git
cd IMSI-catcher
```

## Finding GSM Frequencies

### Using grgsm_scanner

The first step is finding active GSM frequencies in your area:

```bash
grgsm_scanner
```

This will show output like:

```
ARFCN:  974, Freq:  925.0M, CID:     2, LAC:   1337, MCC: 208, MNC:  20, Pwr: -41
ARFCN:  976, Freq:  925.4M, CID:  4242, LAC:   1007, MCC: 208, MNC:  20, Pwr: -45
```

Take note of the frequencies - you'll use one of these for monitoring.

### Understanding the Output

| Field | Meaning |
|-------|---------|
| ARFCN | Absolute Radio Frequency Channel Number - identifies the specific GSM channel |
| Freq | The center frequency to tune to |
| CID | Cell Identifier - unique ID for the tower |
| LAC | Location Area Code - identifies the general area |
| MCC | Mobile Country Code - country identifier |
| MNC | Mobile Network Code - identifies the carrier |
| Pwr | Signal strength in dBm |

### Alternative: Using kalibrate-rtl

If grgsm_scanner doesn't work well, install kalibrate-rtl:

```bash
sudo apt install build-essential libtool automake autoconf librtlsdr-dev libfftw3-dev
git clone https://github.com/steve-m/kalibrate-rtl
cd kalibrate-rtl
./bootstrap && CXXFLAGS='-W -Wall -O3' ./configure
make
sudo make install

# Scan for GSM frequencies
kal -s GSM900
```

## Capturing IMSI Numbers

### Step 1: Start grgsm_livemon

Open a terminal and start the GSM live monitor on one of the frequencies discovered:

```bash
grgsm_livemon -f 925.4M
```

If you see hex output like this, it's working:

```
15 06 21 00 01 f0 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b
25 06 21 00 05 f4 f8 68 03 26 23 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b
49 06 1b 95 cc 02 f8 02 01 9c c8 03 1e 57 a5 01 79 00 00 1c 13 2b 2b
```

### Step 2: Run the IMSI Catcher

Open a second terminal:

```bash
cd IMSI-catcher
sudo python3 simple_IMSI-catcher.py -s
```

The `-s` flag tells it to sniff on the loopback interface where grgsm_livemon is sending data.

### Step 3: View Captured IMSIs

After a moment, the script will display captured IMSI numbers:

```
IMSI: 208201234567890, Country: France, Brand: Orange
IMSI: 208012345678901, Country: France, Brand: SFR
```

Each IMSI reveals:
- **MCC** (first 3 digits): Country code
- **MNC** (next 2-3 digits): Mobile network code (carrier)
- **MSIN** (remaining digits): Unique subscriber ID

### Optional: Analyze with Wireshark

For deeper analysis, capture GSMTAP packets:

```bash
sudo wireshark -k -Y '!icmp && gsmtap' -i lo
```

This lets you inspect individual GSM packets.

## Advanced: Tracking and Geolocation

### Finding Tower Location

With the LAC and Cell ID from your scan, you can locate the tower on a map:

1. Visit a cell tower database like https://cellidfinder.com
2. Enter MCC, MNC, LAC, and Cell ID
3. The website will show the tower's approximate location

### Logging to Database

To save captured IMSIs to a file:

```bash
sudo python3 simple_IMSI-catcher.py -s --txt captured_imsi.txt
```

Or to an SQLite database:

```bash
sudo python3 simple_IMSI-catcher.py -s --sqlite imsi_data.db
```

## Troubleshooting

### No Data Appearing?
- Try a different frequency - GSM signals vary by location
- Ensure you have GSM 900/1800 MHz coverage in your area (GSM is being phased out in some countries)
- Check antenna connection
- Verify you're not using Python 3.9 (has a ctypes bug)

### gr-gsm Installation Fails?
- For Python 3.11+, you may need to replace `imp` with `importlib` in the code
- Consider using Docker: `docker pull atomicpowerman/imsi-catcher`
- The main gr-gsm repo may not work with newer GNU Radio versions - use the `maint-3.8` branch

### Running on Raspberry Pi?
- DragonOS Pi64 comes preconfigured with gr-gsm and all dependencies
- The RTL-SDR works well on Pi 4, but compilation can be slow

## Security Considerations

> ⚠️ **WARNING**: This tutorial is for educational purposes only. Building and using an IMSI catcher may be illegal in your jurisdiction. Always:
> - Only use on networks you have permission to test
> - Respect privacy laws and local regulations
> - Understand that passive IMSI catchers don't transmit, making them less regulated than active devices

### Important Notes

- **GSM is outdated** - Many carriers have phased it out in favor of 3G/4G/5G
- **This is passive** - Unlike Stingrays that impersonate cell towers and force connections, passive catchers just listen
- **IMSI reveals your carrier, not your identity** - But it can be linked to personal info with additional data
- **Privacy risk** - This demonstrates how easily phones leak unique identifiers

## Fun Facts

- Motherboard magazine showed you can build this "with $20 of gear from Amazon in 30 minutes"
- Traditional IMSI catchers used by law enforcement cost thousands of dollars - this proves the hardware is simple
- The same technique works for 3G and some 4G phones when they fall back to 2G
- You can also use HackRF One instead of RTL-SDR for more advanced capabilities

## Additional Resources

- Oros42 IMSI-catcher GitHub: https://github.com/Oros42/IMSI-catcher
- gr-gsm GitHub: https://github.com/ptrkrysik/gr-gsm
- DragonOS: Ready-to-use SDR Linux distro
- Cell tower locator: https://cellidfinder.com
