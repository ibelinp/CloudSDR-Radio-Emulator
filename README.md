# CloudSDR Radio Emulator

A software emulator for CloudSDR hardware that enables SpectraVue to operate with full frequency range support and UDP data streaming.

## Overview

The CloudSDR Emulator provides a complete software implementation of the CloudSDR protocol, allowing SpectraVue software to connect and operate as if connected to real CloudSDR hardware. This opens up many possibilities for SDR experimentation and integration.

## Features

- ✅ **Full Protocol Support** - Implements CloudSDR TCP control protocol with exact hardware responses
- ✅ **Extended Frequency Range** - Configurable frequency range (default: 0-2 GHz)*
- ✅ **UDP Data Streaming** - Generates realistic I/Q data with carrier signals and noise
- ✅ **SpectraVue Compatible** - Works with SpectraVue software set to CloudSDR radio mode
- ✅ **Bridge Capability** - Can be modified to interface with other SDR hardware
- ✅ **Flexible Configuration** - Multiple verbosity levels and easy parameter adjustment

*Higher frequencies might need the use of external radio commands. SpectraVue has limits of 32 bits for direct frequency control.

## Use Cases

### 1. **SDR Hardware Bridge**
Modify the emulator to bridge SpectraVue with other SDR hardware:
- RTL-SDR dongles
- HackRF devices  
- USRP radios
- Custom SDR solutions

### 2. **Data Source Integration**
Connect SpectraVue to alternative data sources:
- Pre-recorded I/Q files
- Network streaming sources
- Signal generators
- Test equipment

### 3. **Development and Testing**
- Protocol development and reverse engineering
- SpectraVue feature testing without hardware
- Educational demonstrations
- Software validation

### 4. **Remote Operation**
- Network-based SDR access
- Cloud SDR implementations
- Remote laboratory setups

## Quick Start

### Installation

```bash
git clone https://github.com/ibelinp/CloudSDR-Radio-Emulator.git
cd CloudSDR-Radio-Emulator
```

No additional dependencies required - uses Python standard library only.

### Basic Usage

```bash
# Start with default settings (recommended)
python cloudsdr_emulator.py -v 1

# Quiet mode (errors only)
python cloudsdr_emulator.py -v 0

# Debug mode (detailed protocol logging)
python cloudsdr_emulator.py -v 2

# Custom port
python cloudsdr_emulator.py -p 50001
```

### Connect SpectraVue

1. Start the emulator
2. In SpectraVue, configure for **CloudSDR radio mode**
3. Connect to `localhost:50000` (or your custom port)
4. SpectraVue will detect the emulator as CloudSDR hardware
5. Full 0-2 GHz frequency range will be available*

*Note: Higher frequencies may require external radio commands due to SpectraVue's 32-bit frequency limitations.

## Configuration

### Frequency Range

To change the maximum frequency, edit the frequency range response in `handle_request_range_message()`:

```python
# Current: 2 GHz maximum
0x00, 0x94, 0x35, 0x77, 0x00, 0x00, 0x00, 0x00, # Max freq: 2000000000 Hz = 2 GHz

# Example: 6 GHz maximum  
0x00, 0x1A, 0x07, 0x24, 0x01, 0x00, 0x00, 0x00, # Max freq: 6000000000 Hz = 6 GHz
```

### UDP Data Generation

The emulator generates realistic I/Q data in the `udp_data_sender()` method. Modify this function to:
- Connect to real SDR hardware
- Stream from files
- Generate custom test signals
- Bridge network sources

## Advanced Usage

### Creating an SDR Bridge

To bridge with real SDR hardware:

1. **Replace UDP data generation** with SDR hardware interface
2. **Implement frequency control** to tune the SDR hardware  
3. **Handle sample rate changes** to configure SDR accordingly
4. **Process real I/Q data** and forward to SpectraVue

Example modification points:
```python
def udp_data_sender(self):
    # Replace this method with SDR hardware interface
    # - Initialize SDR device
    # - Configure frequency/sample rate based on self.receiver_settings
    # - Read real I/Q data from SDR
    # - Forward to SpectraVue via UDP

def handle_set_message(self, client_socket, ci_code, params):
    # Add SDR hardware control here
    if ci_code == 0x0020:  # Frequency change
        # Configure real SDR hardware frequency
        pass
```

### File Streaming

To stream from I/Q files:
```python
# In udp_data_sender(), replace data generation with:
with open('iq_file.bin', 'rb') as f:
    # Read and stream I/Q data from file
    # Handle loop/repeat as needed
```

**Important:** File playback must match the configured sample rate for proper timing. SpectraVue expects real-time data flow, so the emulator must stream file data at exactly the sample rate specified by SpectraVue (e.g., 2.048 MHz). 

**Better approach:** Use I/Q data from other clocked sources (real SDRs, signal generators) that naturally produce data at the correct sample rates, rather than trying to time file playback precisely.

## Protocol Details

The emulator implements the CloudSDR Interface Specification with these key features:

- **Control Items**: Full support for receiver configuration, status, and capability queries
- **Message Types**: SET, REQUEST, and REQUEST_RANGE operations  
- **Frequency Encoding**: 8-byte frequency fields supporting up to exahertz range
- **UDP Streaming**: Configurable 16-bit/24-bit I/Q data with proper packet headers

**Note:** Not all messages from SpectraVue are currently processed by the emulator. Additional control items and message types can be implemented by referring to the `CloudSdrIQInterfaceSpec009.pdf` specification document included in this repository.

## Troubleshooting

### SpectraVue Won't Connect
- Verify emulator is running and listening on correct port
- Check Windows firewall settings
- Ensure no other software is using port 50000

### Limited Frequency Range
- Check frequency range response in logs
- Verify frequency encoding in `handle_request_range_message()`
- Some SpectraVue versions may have built-in limits

### No UDP Data
- Check UDP socket creation and addressing
- Verify SpectraVue sends UDP configuration commands
- Monitor logs for UDP transmission errors

## Contributing

Contributions welcome! Areas of interest:
- Additional SDR hardware bridges
- **Protocol enhancements** - Implement missing control items from CloudSdrIQInterfaceSpec009.pdf
- Performance optimizations
- Documentation improvements

## License

MIT License - see LICENSE file for details.

## Author

**Pieter Ibelings**

## Acknowledgments

- CloudSDR protocol reverse engineering
- SpectraVue compatibility testing
- SDR community feedback and suggestions

---

*This software is not affiliated with or endorsed by the original CloudSDR hardware manufacturers.*