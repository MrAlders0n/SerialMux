# SerialMux

A serial port multiplexer that creates multiple virtual TTY ports from a single physical serial device. Multiple programs can simultaneously read from and write to the same serial port.

## How It Works

SerialMux opens a physical serial port and creates virtual PTY devices (e.g. `/dev/ttyV0`, `/dev/ttyV1`, `/dev/ttyV2`) via symlinks. Any program that opens a virtual port gets a bidirectional connection to the real serial device:

- **Device to clients**: Data from the serial device is broadcast to all connected virtual ports.
- **Client to device**: Data written to any virtual port is forwarded to the serial device.

## Requirements

- Python 3
- [pyserial](https://pypi.org/project/pyserial/)
- Linux (uses PTY and symlinks)

## Installation

```bash
pip install pyserial
```

## Configuration

Edit the constants at the top of `SerialMux.py`:

```python
REAL_PORT = '/dev/serial/by-id/usb-...'  # Path to the physical serial device
BAUD = 115200                             # Baud rate
VPORTS = ['/dev/ttyV0', '/dev/ttyV1', '/dev/ttyV2']  # Virtual port paths
```

## Usage

```bash
python SerialMux.py
```

Enable verbose logging with `-v`:

```bash
python SerialMux.py -v
```

Then connect to any virtual port from another program:

```bash
screen /dev/ttyV0 115200
picocom /dev/ttyV1 -b 115200
```

## Recovery

SerialMux handles failures automatically:

- **Client disconnect**: The virtual port is marked idle and reactivated when a new client connects. The symlink path stays the same.
- **USB serial disconnect**: The muxer retries the connection every 2 seconds until the device reappears.
- **Dead PTY**: If a virtual port enters an unrecoverable state, it is torn down and recreated at the same symlink path.

## Stopping

Send `SIGINT` (Ctrl+C) or `SIGTERM`. The muxer closes all file descriptors and removes the symlinks on shutdown.
