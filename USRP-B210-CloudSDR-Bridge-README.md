# USRP B210 CloudSDR Bridge

**Connect your USRP B210 to SpectraVue transparently via CloudSDR protocol emulation**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)

## Overview

This bridge enables **SpectraVue** and other CloudSDR-compatible software to connect to a **USRP B210** as if it were a native CloudSDR device. The bridge handles all protocol translation, provides professional streaming architecture, and unlocks the full capabilities of your B210.

### Key Features

- ✅ **Complete SpectraVue Compatibility** - All 12 sample rates supported (2.048 MHz down to 16 kHz)
- ✅ **Gain Control Integration** - SpectraVue gain controls (0/-10/-20/-30dB) map to optimal B210 settings
- ✅ **Frequency Offset Support** - Access B210's full 6 GHz range beyond SpectraVue's 2 GHz limit
- ✅ **Adaptive Master Clock** - Automatic clock selection for optimal decimation ratios
- ✅ **Professional Streaming** - Double-buffered architecture handles USB burst transfers
- ✅ **Exact Sample Rates** - Perfect digital timing with no approximations

## Requirements

### Hardware
- **USRP B210** with USB 3.0 connection
- **Computer** with USB 3.0 port (USB 2.0 supported with reduced performance)

### Software
- **Python 3.8+**
- **UHD (USRP Hardware Driver)** - Install via:
  ```bash
  # Linux (Ubuntu/Debian)
  sudo apt install uhd-host
  
  # Conda (recommended)
  conda install -c conda-forge gnuradio uhd
  
  # Or build from source: https://github.com/EttusResearch/uhd
  ```
- **Python Dependencies**:
  ```bash
  pip install numpy
  ```

### SpectraVue Software
- **SpectraVue** (any version supporting CloudSDR)
- Configure SpectraVue to connect to `localhost:50000` (default) or your custom host/port

## Installation

1. **Clone or download** the bridge script
2. **Install UHD** and Python dependencies
3. **Connect your B210** and verify with:
   ```bash
   uhd_find_devices
   uhd_usrp_probe
   ```
4. **Run the bridge** (see Usage section)

## Usage

### Basic Operation

```bash
# Standard operation (0-2 GHz range)
python b210_bridge.py

# Custom port
python b210_bridge.py --port 51000

# Verbose logging
python b210_bridge.py --verbose 2

# Quiet mode
python b210_bridge.py --verbose 0
```

### Frequency Offset (Access Full 6 GHz Range)

SpectraVue is limited to 0-2 GHz, but the B210 covers 70 MHz - 6 GHz. Use frequency offset to access the full range:

```bash
# Access 2.4-4.4 GHz range
python b210_bridge.py --offset 2.4e9

# Access 4-6 GHz range  
python b210_bridge.py --offset 4e9

# VHF/UHF operation
python b210_bridge.py --offset 400e6
```

**How it works:**
- SpectraVue shows **100 MHz** → B210 actually tunes **2.5 GHz** (with 2.4 GHz offset)
- SpectraVue shows **1.8 GHz** → B210 actually tunes **4.2 GHz**

### Device Selection

```bash
# Specific B210 by serial number
python b210_bridge.py --device "serial=12345"

# Specific USB address
python b210_bridge.py --device "addr=192.168.10.2"
```

## SpectraVue Configuration

### Connection Setup
1. **Start the bridge** with your desired settings
2. **Launch SpectraVue**
3. **Configure CloudSDR connection**:
   - **Host**: `localhost` (or bridge host IP)
   - **Port**: `50000` (or your custom port)
   - **Device Type**: CloudSDR

### Gain Controls
SpectraVue's gain controls are automatically mapped to optimal B210 settings:

| SpectraVue Setting | B210 Hardware Gain | Use Case |
|-------------------|-------------------|----------|
| **0dB** | 60dB | High sensitivity for weak signals |
| **-10dB** | 45dB | Good sensitivity, reduced noise |
| **-20dB** | 30dB | Medium sensitivity for average signals |
| **-30dB** | 15dB | Low sensitivity for strong signals |

### Sample Rates
All SpectraVue sample rates are supported with **exact precision**:

- **2.048 MHz** - Wideband monitoring
- **1.2288 MHz** - Digital communications  
- **614.4 kHz** - Medium bandwidth
- **495.483 kHz** - Custom rate
- **370.120 kHz** - Narrow bandwidth
- **245.76 kHz** - Voice communications
- **122.88 kHz** - Standard narrow
- **64 kHz** - Very narrow
- **48 kHz** - Audio bandwidth
- **34.594 kHz** - Ultra narrow
- **30.72 kHz** - Minimal bandwidth
- **16 kHz** - Extremely narrow

## Technical Details

### Adaptive Master Clock
The bridge automatically selects optimal master clock frequencies to achieve exact sample rates:

- **61.44 MHz** - For rates 122.88 kHz and above
- **30.72 MHz** - For 64 kHz (decimation 480)
- **15.36 MHz** - For 48 kHz, 34.594 kHz, 30.72 kHz
- **7.68 MHz** - For 16 kHz (decimation 480)

This ensures all decimation ratios stay within the B210's 512 limit.

### Streaming Architecture
- **Primary Buffer**: 4-second capacity, absorbs USB burst transfers
- **Secondary Buffer**: 1-second capacity, provides smooth UDP delivery
- **Precision Timing**: Adaptive timing correction maintains exact sample rates
- **Professional Quality**: No dropped samples or timing jitter

### Protocol Compatibility
Implements complete CloudSDR control protocol:
- Device identification and capabilities
- Frequency and gain control
- Sample rate configuration
- UDP data streaming (16-bit and 24-bit modes)
- Status and error reporting

## Troubleshooting

### Connection Issues
```
❌ B210: Hardware initialization failed
```
**Solutions:**
- Check USB 3.0 connection
- Verify UHD installation: `uhd_find_devices`
- Check device permissions (Linux users may need udev rules)
- Try different USB port or cable

### Rate Errors
```
⚠️ Rate error: XXXX Hz
```
**Usually indicates:**
- Master clock auto-selection working correctly
- Minor timing differences (< 1000 Hz are normal)
- Check UHD version if errors persist

### SpectraVue Connection
```
SpectraVue shows "connection refused"
```
**Solutions:**
- Ensure bridge is running before starting SpectraVue
- Check firewall settings
- Verify correct host/port in SpectraVue
- Try `--host 0.0.0.0` to bind all interfaces

### Audio Quality Issues
```
Garbled or distorted audio
```
**Solutions:**
- Check sample rate compatibility
- Verify gain settings aren't too high
- Ensure stable USB 3.0 connection
- Try lower sample rates

### Frequency Range
```
Can't tune above 2 GHz in SpectraVue
```
**Solution:**
- Use `--offset` parameter to access higher frequencies
- Example: `--offset 2e9` adds 2 GHz to all frequencies

## Command Line Reference

```bash
python b210_bridge.py [OPTIONS]

Options:
  -v, --verbose LEVEL     Verbosity: 0=quiet, 1=normal, 2=debug, 3=trace
  -p, --port PORT         TCP port to listen on (default: 50000)
  -d, --device ARGS       UHD device arguments (e.g., "serial=12345")
  --offset FREQUENCY      Frequency offset in Hz (access full B210 range)
  --host ADDRESS          Host address to bind to (default: localhost)
  -h, --help             Show help message
```

## Example Configurations

### Standard Operation
```bash
# Basic SpectraVue connection
python b210_bridge.py --verbose 1

# Output:
# 🚀 USRP B210 CloudSDR Bridge
# 📡 USRP B210 → CloudSDR Protocol → SpectraVue  
# ✅ Ready for SpectraVue connection...
```

### High Frequency Operation
```bash
# Monitor 3.5 GHz cellular band
python b210_bridge.py --offset 3.3e9

# SpectraVue 200 MHz → B210 3.5 GHz actual
```

### Multi-User Setup
```bash
# Bind to all interfaces for remote access
python b210_bridge.py --host 0.0.0.0 --port 50001
```

## Performance Tips

### USB 3.0 Optimization
- Use **Intel USB 3.0 controllers** when possible
- Enable **USB 3.0 power management** disabled
- Use high-quality **USB 3.0 cables** (< 3 meters)

### Sample Rate Selection
- **Higher rates** (> 1 MHz) for wideband monitoring
- **Medium rates** (100-500 kHz) for digital signals
- **Lower rates** (< 100 kHz) for voice and narrow signals

### Gain Settings
- Start with **-10dB** (45dB B210 gain) for general use
- Use **0dB** (60dB) for weak signals only
- Use **-20dB** or **-30dB** for strong local signals

## Contributing

Contributions welcome! Areas for improvement:
- Additional SDR hardware support
- Enhanced streaming protocols
- GUI interface
- Additional client software compatibility

## License

MIT License - see LICENSE file for details.

## Credits

- **CloudSDR Protocol**: Based on Pieter Ibelings' CloudSDR Emulator
- **USRP Hardware**: Ettus Research USRP B210
- **SpectraVue**: RF Explorer SpectraVue software
- **UHD**: Ettus Research USRP Hardware Driver

## Support

For issues and questions:
1. Check this documentation
2. Review troubleshooting section  
3. Open GitHub issue with:
   - Bridge version and command line used
   - UHD version (`uhd_find_devices --version`)
   - Error messages and logs
   - Operating system and hardware details

---

**Happy SDR monitoring!** 📡✨