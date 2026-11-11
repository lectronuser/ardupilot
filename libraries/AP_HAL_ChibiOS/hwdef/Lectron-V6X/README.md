# Lectron V6X Flight Controller

TBD: board image

The Lectron V6X is a flight controller designed by [Lectron](https://lectrontech.com/). It follows the Pixhawk Autopilot FMUv6X standard: an STM32H753 FMU module plugs into a carrier (base) board through the Pixhawk Autopilot Bus (PAB) connector. The FMU module carries triple redundant IMUs, dual barometers, a magnetometer, IMU heating, FRAM and the Ethernet PHY. An STM32F103 IO co-processor on the carrier board handles RC input and the MAIN PWM outputs.

## Where to Buy

For pricing and purchasing information, please [contact Lectron](https://lectrontech.com/contact-us/).

## Specifications

- **Processor**
  - STM32H753 32-bit processor (FMU), Arm Cortex-M7, 480 MHz, 2 MB flash, 1 MB RAM
  - STM32F103 IO co-processor (IOMCU), Arm Cortex-M3, 72 MHz
- **Sensors**
  - InvenSense ICM-42688-P IMU (accel, gyro)
  - InvenSense ICM-45686 IMU (accel, gyro)
  - Bosch BMI088 IMU (accel, gyro)
  - Bosch BMM350 magnetometer
  - Bosch BMP581 barometer
  - TE MS5611 barometer
- **Onboard storage**
  - FM25V02A FRAM (SPI) for parameter storage
  - 24LC64 EEPROM (I2C)
  - microSD card for logging
- **Power**
  - TBD
- **Interfaces**
  - 8x UARTs, 7 available for customer use
  - 16x PWM outputs (8x MAIN from the IOMCU, 8x AUX from the FMU)
  - RC input (on the IOMCU)
  - 3x external I2C buses
  - 2x CAN buses
  - External SPI port (SPI6) with 2 chip selects, 2 data-ready inputs, reset and sync lines
  - USB port
  - 10/100 Mbps Ethernet (LAN8742A PHY)
  - Buzzer and safety switch
  - RGB LED
- **Size and Dimensions**
  - Weight: TBD
  - Dimensions: TBD

## Pinout

TBD: pinout image

## UART Mapping

Default mapping and protocols are shown below.

| Serial # | UART   | Protocol | Port      | Function            | Notes                    |
| -------- | ------ | -------- | --------- | ------------------- | ------------------------ |
| SERIAL0  | OTG1   | MAVLink2 | USB       | USB                 |                          |
| SERIAL1  | UART7  | MAVLink2 | TELEM1    | Telemetry port      | RTS/CTS pins             |
| SERIAL2  | UART5  | MAVLink2 | TELEM2    | Telemetry port      | RTS/CTS pins             |
| SERIAL3  | USART1 | GPS      | GPS1      | GPS                 |                          |
| SERIAL4  | UART8  | GPS      | GPS2      | GPS                 |                          |
| SERIAL5  | USART2 | None     | TELEM3    | Telemetry port      | RTS/CTS pins             |
| SERIAL6  | UART4  | None     | TBD       | User                |                          |
| SERIAL7  | USART3 | None     | FMU DEBUG | Debug               | No DMA                   |

- USART6 provides the internal link between the FMU and the IO co-processor (IOMCU) and is not available as a serial port.
- The USART3 connector is labelled debug, but is available as a general purpose UART with ArduPilot.
- All UARTs except USART3 are DMA capable.

## RC Input

RC input is connected to the IO co-processor through the RC IN connector on the carrier board. The IOMCU decodes PPM, S.BUS, DSM, SUMD, SRXL, IBUS, ST24 and FPort (RC only) on this connector. TBD: confirm whether a separate DSM/Spektrum satellite connector is fitted on the carrier board.

FPort connected to RC IN provides RC only, without telemetry. CRSF/ELRS and SRXL2 need a true bi-directional UART and cannot be used on RC IN.

To use CRSF, SRXL2, or FPort with telemetry, connect the receiver to a full UART (see [UART Mapping](#uart-mapping)), such as TELEM1, and set the SERIALn parameters of the UART used:

`SERIALn_PROTOCOL` should be set to `23` (RCIN).

- FPort requires `SERIALn_OPTIONS` be set to `15`.
- CRSF requires `SERIALn_OPTIONS` be set to `0`.
- SRXL2 requires `SERIALn_OPTIONS` be set to `4` and connects only the TX pin.

See [Radio Control Systems](https://ardupilot.org/copter/docs/common-rc-systems.html) for details.

## PWM Output

The Lectron V6X supports up to 16 PWM outputs. All outputs support all normal PWM output formats. Outputs within the same timer group need to use the same output rate and protocol. If any output in a group uses DShot then all outputs in the group need to use DShot.

### MAIN outputs (IOMCU)

The 8 MAIN outputs are driven by the IO co-processor. DShot is not supported on the MAIN outputs.

| Output | Timer group | DShot | Bi-Directional DShot |
| ------ | ----------- | ----- | -------------------- |
| MAIN 1 | 1 (TIM2)    | No    | No                   |
| MAIN 2 | 1 (TIM2)    | No    | No                   |
| MAIN 3 | 2 (TIM4)    | No    | No                   |
| MAIN 4 | 2 (TIM4)    | No    | No                   |
| MAIN 5 | 3 (TIM3)    | No    | No                   |
| MAIN 6 | 3 (TIM3)    | No    | No                   |
| MAIN 7 | 3 (TIM3)    | No    | No                   |
| MAIN 8 | 3 (TIM3)    | No    | No                   |

### AUX outputs (FMU)

The 8 AUX outputs are driven directly by the FMU.

| Output | Timer group | DShot | Bi-Directional DShot |
| ------ | ----------- | ----- | -------------------- |
| AUX 1  | 1 (TIM5)    | Yes   | Yes                  |
| AUX 2  | 1 (TIM5)    | Yes   | Yes                  |
| AUX 3  | 1 (TIM5)    | Yes   | Yes                  |
| AUX 4  | 1 (TIM5)    | Yes   | Yes                  |
| AUX 5  | 2 (TIM4)    | Yes   | No                   |
| AUX 6  | 2 (TIM4)    | Yes   | No                   |
| AUX 7  | 3 (TIM12)   | No    | No                   |
| AUX 8  | 3 (TIM12)   | No    | No                   |

To use Bi-Directional DShot on AUX 1-4, set `SERVO_BLH_BDMASK` for those outputs and use an ESC firmware that supports it.

## Battery Monitoring

The board is configured by default for an INA226 I2C digital power monitor on the power port. The correct battery monitor parameters depend on the power module connected.

The default parameters are:

- `BATT_MONITOR` = 21
- `BATT_I2C_BUS` = 1
- `BATT_I2C_ADDR` = 0 (probe the supported addresses)

TBD: power port details

## Compass

The Lectron V6X has a built-in BMM350 compass. Due to potential interference, the autopilot is usually used with an external I2C compass as part of a GPS/Compass combination.

## IMU Heater

The IMU heater target temperature is set with the `BRD_HEAT_TARG` parameter, in degrees C. The default is 45 C.

## Ethernet

The board has a 10/100 Mbps Ethernet interface using a LAN8742A PHY. Networking is enabled by default with the following settings:

- `NET_ENABLE` = 1
- `NET_P1_TYPE` = 1 (UDP client)
- `NET_P1_PROTOCOL` = 2 (MAVLink2)
- `NET_P1_IP0` to `NET_P1_IP3` = 192.168.144.14 (destination address)
- `NET_P1_PORT` = 14550

The flight controller uses DHCP by default (`NET_DHCP` = 1). Without a DHCP server it falls back to the static address set by `NET_IPADDR0` to `NET_IPADDR3`. See [Ethernet/Networking](https://ardupilot.org/copter/docs/common-network.html) for details.

## CAN

The Lectron V6X has two independent CAN buses (CAN1 and CAN2).

## External SPI

SPI6 is available on an external connector with two chip selects, two data-ready inputs, a reset line and a sync line. No devices are configured on this bus by default.

## GPIOs

The AUX PWM outputs and some other pins can be used as GPIOs (relays, buttons, RPM etc). To use an output as a GPIO, set its `SERVOx_FUNCTION` to -1. See [GPIOs](https://ardupilot.org/copter/docs/common-gpios.html) for details.

The numbering of the GPIOs for PIN variables in ArduPilot is:

- AUX1 50
- AUX2 51
- AUX3 52
- AUX4 53
- AUX5 54
- AUX6 55
- AUX7 56
- AUX8 57
- FMU_CAP1 58
- NFC_GPIO 60
- MAIN1 101
- MAIN2 102
- MAIN3 103
- MAIN4 104
- MAIN5 105
- MAIN6 106
- MAIN7 107
- MAIN8 108

## Analog Inputs

The Lectron V6X has the following analog inputs:

- ADC Pin12 -> ADC 6.6V Sense
- ADC Pin13 -> ADC 3.3V Sense
- ADC Pin103 -> RSSI input on the RC IN connector, read on the IO co-processor (STM32F103), not the FMU. To use analog RSSI, set `RSSI_TYPE` = 1 and `RSSI_ANA_PIN` = 103.

## Connectors

TBD

## Loading Firmware

Firmware for this board can be found at the [ArduPilot firmware server](https://firmware.ardupilot.org) in sub-folders labeled "Lectron-V6X".

The board comes pre-installed with an ArduPilot compatible bootloader, allowing the loading of \*.apj firmware files with any ArduPilot compatible ground station.
