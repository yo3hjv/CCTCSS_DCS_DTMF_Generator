# ESP32 based  GPS-Disciplined Signal Generator

A high-precision signal generator based on ESP32, capable of generating CTCSS, DCS, DTMF, and sine wave signals with GPS-disciplined accuracy.

## Overview

This project transforms an ESP32 development board into a laboratory-grade signal generator by leveraging GPS timing for precise frequency calibration. The system generates industry-standard signaling tones used in radio communications with accuracy that rivals commercial equipment.

### Key Features

- **GPS-Disciplined Oscillator** for exceptional frequency accuracy
- **Dual DAC Architecture** for simultaneous signal generation
- **Web Interface** for easy control from any device
- **Multiple Signal Types**:
  - CTCSS (Continuous Tone Coded Squelch System): 67.0-254.1 Hz
  - DCS (Digital Coded Squelch): Standard codes 023-754
  - DTMF (Dual-Tone Multi-Frequency): Full keypad with sequence support
  - SINE: Variable frequency 50-1200 Hz
- **Industry-Grade Precision**: <0.1 Hz accuracy for CTCSS/DCS tones

## System Architecture

```
+-----------------------------------------------------------------------+
|                      ESP32 SIGNAL GENERATOR                            |
+-----------------------------------------------------------------------+

+-------------+     +----------------+     +----------------+
| GPS Module  |---->| PPS Input      |---->| PPS Processing |
| (NEO-6M/7M) |     | (GPIO 34)      |     | ISR            |
+-------------+     +----------------+     +----------------+
                                                  |
                                                  v
+----------------+     +----------------+     +----------------+
| Tick           |<----| Hardware       |<----| Calibration    |
| Counter        |     | Timer          |     | System         |
+----------------+     +----------------+     +----------------+
        |                                            |
        v                                            v
+----------------+     +----------------+     +----------------+
| Drift          |---->| Correction     |---->| Dynamic        |
| Calculator     |     | Factor         |     | Correction     |
+----------------+     +----------------+     +----------------+
                                                  |
                                                  v
+-----------------------------------------------------------------------+
|                      SOFTWARE DDS GENERATORS                           |
+-----------------------------------------------------------------------+
        |                      |                       |
        v                      v                       v
+----------------+     +----------------+     +----------------+
| CTCSS DDS      |     | DCS DDS        |     | DTMF DDS       |
| (67-254.1 Hz)  |     | (Digital Code) |     | (Dual Tone)    |
+----------------+     +----------------+     +----------------+
        |                      |                       |
        v                      v                       v
+----------------+     +----------------+     +----------------+
| Mixer/Selector |     | Mixer/Selector |     | Mixer/Selector |
| Channel 1      |     | Channel 1      |     | Channel 2      |
+----------------+     +----------------+     +----------------+
        |                      |                       |
        +----------------------+                       |
                  |                                    |
                  v                                    v
        +------------------+              +------------------+
        | DAC 1 (GPIO 25)  |              | DAC 2 (GPIO 26)  |
        | CTCSS/DCS/SINE   |              | DTMF             |
        +------------------+              +------------------+
                  |                                    |
                  v                                    v
        +------------------+              +------------------+
        | Analog Output    |              | Analog Output    |
        | Channel 1        |              | Channel 2        |
        +------------------+              +------------------+

+-----------------------------------------------------------------------+
|                      AUXILIARY DIGITAL OUTPUTS                         |
+-----------------------------------------------------------------------+
        |                      |                       |
        v                      v                       v
+----------------+     +----------------+     +----------------+
| PPS Output     |     | DCS Sync       |     | 5 KHz 50%      |
| (GPIO 17)      |     | (GPIO 10)      |     | (GPIO 32)      |
+----------------+     +----------------+     +----------------+
```

## GPS Disciplining Process

The heart of this project is the GPS disciplining system that provides exceptional frequency accuracy:

```
+----------------------------------------------------------------------------------------------+
|                       GPS DISCIPLINED OSCILLATOR                                             |
+----------------------------------------------------------------------------------------------+

+-------------+     +-------------+     +----------------+     +----------------+
| GPS PPS     |---->| GPIO        |---->| Count cycles   |---->| Calculate      |
| Signal      |     | Interrupt   |     | since last     |     | actual period  |
| (GPIO 34)   |     |             |     | PPS pulse      |     | (ticks/sec)    |
+-------------+     +-------------+     +----------------+     +----------------+
                                                                      |
                                                                      v
+----------------+     +----------------+     +----------------+     +----------------+
| Apply          |<----| Calculate      |<----| Compare with   |<----| Theoretical   |
| correction     |     | correction     |     | theoretical    |     | period        |
| factor to DDS  |     | factor         |     | period         |     | (constant)    |
+----------------+     +----------------+     +----------------+     +----------------+
      |
      v
+----------------+     +----------------+     +----------------+
| Generate       |---->| Verify         |---->| Output         |
| corrected      |     | synchronization|     | signal         |
| signal         |     | on oscilloscope|     | (GPIO 25/26)   |
+----------------+     +----------------+     +----------------+
                            |
                            v
                      +----------------+
                      | Reference      |
                      | signal 5kHz    |
                      | (GPIO 32)      |
                      +----------------+
```

## Hardware Requirements

- ESP32 development board (recommended: ESP32-WROOM-32)
- GPS module with PPS output (NEO-6M/7M)
- Basic passive components (resistors, capacitors)
- Optional: oscilloscope for calibration verification

## GPIO Assignments

- **GPS PPS Input**: GPIO 34
- **DDS 1 (Tones) DAC 1**: GPIO 25
- **DDS 2 (DTMF) DAC 2**: GPIO 26
- **GPS PPS Output**: GPIO 17
- **DCS Sync**: GPIO 10
- **GPS derived 5 KHz 50%**: GPIO 32

## Web Interface

The project includes a responsive web interface accessible via WiFi:
- Control panel for all signal generation parameters
- Real-time frequency and level adjustment
- DTMF keypad and sequence programming
- System status monitoring
- File management for configuration

## Technical Innovations

### Cycle Counting vs. Time Measurement
Instead of using traditional time measurement functions (millis, micros) which introduce random errors, this project uses direct cycle counting between GPS PPS pulses for superior stability.

### Dual DAC Architecture
The ESP32's two DACs are utilized to enable simultaneous generation of different signal types:
- DAC1 (GPIO 25): CTCSS/DCS/SINE generation
- DAC2 (GPIO 26): DTMF generation

### DDS Implementation
Direct Digital Synthesis (DDS) algorithms with GPS correction provide exceptional frequency accuracy despite the limited resolution of the ESP32's 8-bit DACs.

## Building the Project

### Prerequisites
- ESP32 Flash Download Tool from Espressif
- Binary files (.bin) for the project
- USB connection to ESP32
- Web interface files archive (ESP32_SigGen_WebUI.zip)

### Firmware Installation
1. Download the ESP32 Flash Download Tool from [Espressif's website](https://www.espressif.com/en/support/download/other-tools)
2. Launch the ESP32 Flash Download Tool
3. Configure the tool with the following settings:
   - Select appropriate COM port for your ESP32
   - Set baud rate to 921600
   - Add the binary files (.bin) with their respective addresses:
     - ESP32_SigGen_... bootloader.bin: 0x1000
     - ESP32_SigGen_...partitions.bin: 0x8000     
     - ESP32_SigGen_....bin: 0x10000
     
4. Check "DoNotChgBin" for all files
5. Click "START" to flash the ESP32
6. Wait for the flashing process to complete (indicated by "FINISH")

### Web Interface Files Installation
1. Extract the `ESP32_SigGen_WebUI.zip` archive to a local folder
2. Power on the ESP32 and connect to its WiFi network "ESP32_SigGen"
3. Navigate to http://192.168.4.1/filemanager in your browser
4. #define DEFAULT_AP_SSID "ESP32_SigGen"
   #define DEFAULT_AP_PASSWORD "siggen123"
   #define DEFAULT_HOSTNAME "siggen"
5. In the File Manager interface:
   - Click "Choose File" and select all web related files from the extracted archive
   - Click "Upload" to send the files to the ESP32
   - All files can be uploaded at once (batch)

6. The web interface files will be stored in the ESP32's SPIFFS file system

### About the Web Interface Archive
The `WEB_GUI....zip` archive contains all necessary files for the responsive web interface:
- HTML files for the main pages (index.html, control.html, wifi.html, filemanager.html)
- CSS files for styling and responsive layout
- JavaScript files for client-side functionality
- Image files for icons and visual elements

The interface is designed to work on multiple devices (desktop, tablet, mobile) with both portrait and landscape orientations. It provides full control over all signal generation parameters and real-time feedback of the generator's status.

## Usage

1. Power on the ESP32
2. Connect to the WiFi network "ESP32_SigGen" (default password: "password")
3. Navigate to http://192.168.4.1 in your browser
4. Use the control panel to generate desired signals

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Copyright

© 2025 Adrian YO3HJV.

## Acknowledgments

- GPS disciplining technique inspired by precision frequency standards
- Web interface built with modern responsive design principles made with intensive AI contribution
- Special thanks to the ESP32 and Arduino communities for their excellent libraries and documentation
