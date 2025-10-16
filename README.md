# gs_can_weact_H7_coreboard

gs_can firmware for the STM32H7 core board from AliExpress, implementing a USB to CAN adapter using the gs_usb protocol.

## Dependencies

### Required Tools

#### ARM GCC Toolchain
```bash
# Download from ARM official site or install via package manager
# Version tested: gcc-arm-none-eabi-10.3-2021.10
sudo apt update
sudo apt install gcc-arm-none-eabi
```

#### ST-Flash Utilities
```bash
# For flashing the STM32 via ST-Link
sudo apt install stlink-tools
```

#### Make
```bash
sudo apt install make
# Version tested: GNU Make 4.3
```

#### CAN Utilities
```bash
# For CAN interface management and testing
sudo apt install can-utils
```

### Optional Tools
- **ST-Link V2 programmer** (for flashing firmware)
- **WeAct Studio MiniDebugger**: A compatible debugger for this board. [Link](https://github.com/WeActStudio/WeActStudio.MiniDebugger)
- **Logic analyzer or oscilloscope** (for debugging)

## Hardware

### STM32H7 Core Board
This project uses the **WeAct STM32H743VIT6 Core Board** available on AliExpress:
- **MCU**: STM32H743VIT6 (Cortex-M7, 480MHz)
- **Flash**: 2MB
- **RAM**: 1MB
- **Package**: LQFP-100
- **Key features**: USB-C connector, onboard LED, user buttons

### Board Pinout (Relevant Pins)
- **LED1**: PE3 (Status indicator)
- **USER_BTN_K1**: Configurable. See troubleshooting for notes on PC13, PE3, and PB12.
- **CAN1/CAN2**: Pins for external CAN transceiver.
- **USB**: USB-C connector for communication.

### CAN Transceiver Requirements
You'll need external CAN transceivers (e.g., TJA1050, MCP2551) to interface with actual CAN buses, as the STM32 only provides the CAN controller, not the physical layer.

## Building and Flashing

### Clone and Build
```bash
git clone https://github.com/yourusername/gs_can_weact_H7_coreboard.git
cd gs_can_weact_H7_coreboard
make clean
make
```

### Flash Firmware
```bash
# Using st-flash utility
make flash

# Or manually:
st-flash --reset write build/DevEBoxH7_fw.bin 0x8000000
```

## SocketCAN Setup

### Set Up CAN Interface
```bash
# Bring up the CAN interface
sudo ip link set can0 up type can bitrate 500000

# Verify interface is up
ip link show can0
```

### Configure Different Bitrates
```bash
# For 125 kbps
sudo ip link set can0 up type can bitrate 125000

# For 250 kbps  
sudo ip link set can0 up type can bitrate 250000

# For 1 Mbps
sudo ip link set can0 up type can bitrate 1000000
```

### Take Down CAN Interface
```bash
# Bring down the interface
sudo ip link set can0 down
```

## Usage

### Testing with Button
1. **Connect** the STM32H7 board via USB-C.
2. **Set up** the CAN interface (see above).
3. **Press the K1 button** on the board.
4. **Observe** the CAN message transmission.

The button sends a test frame:
- **CAN ID**: 0x123
- **Data**: `11 22 34 44 55 66 77 88`
- **Length**: 8 bytes

### Monitor CAN Traffic
```bash
# Monitor all CAN traffic
candump can0

# Example output:
# can0  123   [8]  11 22 34 44 55 66 77 88
```

### Send CAN Messages
```bash
# Send a test message
cansend can0 456#DEADBEEF
```

## Troubleshooting

### Button Not Working
If the K1 button doesn't send messages:

1.  **Check CAN interface status**: Ensure `can0` is UP.
2.  **Verify host channel is active**: The firmware only sends when the CAN channel is enabled from the host software.
3.  **Pin Configuration**: The button's GPIO pin is critical.
    *   **PC13**: This pin is connected to button labeled K1 on the board. It must be set as PULLDOWN to work properly
    *   **PE3**: This pin is connected to the onboard LED. See board.c and board.h for details and configuration.

### CAN Interface Issues
```bash
# Reset CAN interface
sudo ip link set can0 down
sudo ip link set can0 up type can bitrate 500000

# Check kernel logs for errors
dmesg | grep can
```

## Development

### Adding Custom CAN Handling
Modify the `task_queue_from_host()` function in `Core/Src/main.c` to add custom CAN frame processing.

The onboard LCD display can also be used to do funny things

### LED Behavior
- **Solid ON**: CAN channel active.
- **Blinking**: CAN traffic activity.

## License

MIT License.
