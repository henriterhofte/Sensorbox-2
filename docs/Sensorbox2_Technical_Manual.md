# Sensorbox 2 Technical and Programming Manual

The Sensorbox 2 uses a PIC microcontroller to perform CT measurements and send data to an ESP32 module. The ESP32 processes P1 port data (from smart meters) and handles RS485 communication with the SmartEVSE. The PIC microcontroller is programmed by the ESP32 during power-up using a `.hex` file stored in the `/data` folder.

The system is built using the Arduino core 2.0.2 (due to critical bug fixes and improved UART drivers) and the eModbus library. The necessary libraries are automatically installed via the `platformio.ini` file.
## Modbus Registers

- **Modbus RTU address:** 0x0A
- **Speed:** 9600 bps

### Input Registers (FC=04) (Read Only)

| Register Address | Length (16 bits) | Description |
| ---------------- | ---------------- | ----------- |
| 0x0000           | 1                | Sensorbox version 2:<br>0x0114 = 2 lsb mirror the 3/4 Wire and Rotation configuration data;<br>0x0115 = 4 wire, CCW rotation;<br>0x0116 = 3 wire, CW rotation;<br>0x0117 = 3 wire, CCW rotation |
| 0x0001           | 1                | DSMR Version (MSB), CT's or P1 (LSB): 0x3283 = DSMR version 50, P1 port connected (0x80), CTs used (0x03) |
| 0x0002           | 2                | Volts L1 (32 bit floating point), Smartmeter P1 data |
| 0x0004           | 2                | Volts L2 (32 bit floating point), Smartmeter P1 data |
| 0x0006           | 2                | Volts L3 (32 bit floating point), Smartmeter P1 data |
| 0x0008           | 2                | Amps L1 (32 bit floating point), Smartmeter P1 data |
| 0x000A           | 2                | Amps L2 (32 bit floating point), Smartmeter P1 data |
| 0x000C           | 2                | Amps L3 (32 bit floating point), Smartmeter P1 data |
| 0x000E           | 2                | Amps L1 (32 bit floating point), CT input 1 |
| 0x0010           | 2                | Amps L2 (32 bit floating point), CT input 2 |
| 0x0012           | 2                | Amps L3 (32 bit floating point), CT input 3 |

> **Additional Registers** available with software version >= 0x01xx:

| Register Address | Length (16 bits) | Description |
| ---------------- | ---------------- | ----------- |
| 0x0014           | 1                | WiFi Connection Status  xxxxxACL xxxxxxWW = WiFi mode (00=Wifi Off, 01=On, 10=portal started); A = AP_STA mode (portal) active; C = Connected to WiFi; L = Local Time Set|
| 0x0015           | 1                | Time Hour (MSB) Minute (LSB) |
| 0x0016           | 1                | Time Month (MSB) Day (LSB) |
| 0x0017           | 1                | Time Year (MSB) Weekday (LSB) |
| 0x0018           | 2                | IP Address of Sensorbox |
| 0x001A           | 2                | MAC Address of Sensorbox (4 LSB bytes) |
| 0x001C           | 4                | Password Portal (8 bytes) |

### Holding Registers (FC=06) (Write)

| Register Address | Length (16 bits) | Description |
| ---------------- | ---------------- | ----------- |
| 0x0800           | 1                | Field rotation setting and 3/4 wire configuration |
| 0x0801           | 1                | Set WiFi mode: 00 = disabled, 01 = enabled, 02 = start portal |

### Example Commands

**Request Sensorbox2 data:**

```
0a 04 00 00 00 14 f1 7e
```

**Sensorbox2 replies with:**

```
0a 04 28 00 14 32 83 43 66 80 00 43 66 4c cd 43 69 19 9a 3e 12 9a 61 3f 9f 83 80 be 41 4a 5a 3b ea 00 3b bb 6f f3 6b 3a 45 8d e3 ca cb
4 wire CW --^^
DSMR ver(50) --^^
P1 + CT connected ^^
Volts L1 ------------| 230.5 V  |
```


**Request 3 wire grid setting:**

```
0a 06 08 00 00 02 0b 10
```

Next sensorbox2 data:
```
0a 04 28 00 16 32 03 43 66 e6 66 43 66 80 00 43 69 19 9a 3d 66 9c 53 3f 97 99 d2 be a2 8a 28 bb 59 73 10 bb 08 29 4d 3b 8f 23 79 ca e6 			 
------------^^ changed to 3 wire, CW setting.
```