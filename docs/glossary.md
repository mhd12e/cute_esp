# Glossary

> Terms used across the project, in alphabetical order.

| Term | Meaning |
|------|---------|
| **A7670E** | The 4G LTE Cat1 modem chip. Has built-in GPS. Communicates via UART AT commands. |
| **AT command** | Text-based commands to the modem (e.g., `AT+CGPS=1` to enable GPS) |
| **APN** | Access Point Name. Identifies which 4G data gateway to use (e.g., "internet" for some operators). Operator-specific. |
| **BCLK** | Bit Clock for I2S audio. One pulse per bit. |
| **BL** | Backlight (display LED illumination) |
| **CS** | Chip Select for SPI. LOW = device active, HIGH = device ignored. |
| **DC** | Data/Command select for display SPI. LOW = command, HIGH = data. |
| **Deep sleep** | ESP32 power state where most of the chip is off. Wakes from timer or RTC GPIO. |
| **DIN** | Data In for I2S audio (the actual audio samples) |
| **DTR** | Data Terminal Ready. A modem control pin. HIGH = sleep, LOW = active. |
| **ext0 wake** | ESP32 wake source. Triggers on a single GPIO pin going to a specified level. Requires RTC GPIO. |
| **FT6236** | Capacitive touch controller (we don't use it) |
| **gpio_hold_en()** | ESP-IDF function to maintain a GPIO output level through sleep. Only persists in deep sleep for RTC GPIOs. |
| **HSPI** | One of ESP32's two hardware SPI peripherals. Used by SD card on this board. |
| **I2S** | Digital audio bus standard. Three wires: BCLK, LRC, DIN. |
| **ILI9488** | The display controller chip. 480×320 RGB, SPI interface. |
| **L76K** | An external GPS chip. Used on A7670G boards (NOT our A7670E). |
| **LiPo** | Lithium polymer battery. Pouch cell form factor. 3.7V nominal. |
| **LRC** | Left-Right Clock for I2S. Also called WS (Word Select). HIGH = right channel, LOW = left. |
| **MAX98357A** | I2S Class D audio amplifier chip. Built-in DAC. Drives speakers up to 3W. |
| **MISO** | Master In Slave Out. SPI data line from peripheral to MCU. We don't use it (write-only display). |
| **MJPEG** | Motion JPEG. Video format = sequence of JPEG frames. ESP32 can decode in real time. |
| **MOSI** | Master Out Slave In. SPI data line from MCU to peripheral. |
| **MUX/PSRAM** | Pseudo-Static RAM. The ESP32 has 8MB external PSRAM for buffers. |
| **PWRKEY** | Power Key. Modem control pin. Pulse to power on/off. |
| **RI** | Ring Indicator. Modem pin that pulses on incoming call/SMS. |
| **RTC GPIO** | A subset of ESP32 GPIOs that remain powered during deep sleep. Required for ext0 wake-up and persistent gpio_hold. |
| **SCK** | Serial Clock for SPI |
| **SCL** | Serial Clock for I2C |
| **SD pin (MAX98357A)** | Shutdown / channel select pin. 3.3V = mono mix, GND = shutdown. |
| **SDA** | Serial Data for I2C |
| **SMS over SGs** | A network feature allowing SMS delivery while phone is on 4G/LTE |
| **TinyGSM** | Arduino library for cellular modem control via AT commands |
| **TP4056** | Common single-cell lithium charging IC. Likely on the LILYGO board. |
| **TTS** | Text-to-Speech. Server converts Karim's text to spoken audio file. |
| **VSPI** | The other ESP32 hardware SPI bus. We use it for the display. |
| **WS** | Word Select for I2S. Same as LRC. |

## Related pages

- [System architecture](system-architecture.md) — many of these terms in context
- [Pin assignment](pin-assignment.md) — every signal name explained
- [Firmware patterns](firmware-patterns.md) — code that uses these primitives
