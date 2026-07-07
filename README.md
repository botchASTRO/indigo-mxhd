# MX-HD INDIGO Driver

This is an initial INDIGO driver for the MX-HD equatorial mount.

It uses the same mount protocol assumptions as the existing ASCOM and INDI drivers:

- LX200-style commands: `:GR#`, `:GD#`, `:Sr#`, `:Sd#`, `:MS#`, `:CM#`
- MX-HD status command: `@ST#`, returning 3 binary bytes
- MX-HD tracking commands: `@FD1#`, `@FD0#`, `@CE0#`, `@LP1#`, `@LP4#`
- MX-HD home/park commands: `@OG#`, `@Hm#`
- MX-HD pulse guide commands: `@mN#`, `@mS#`, `@mE#`, `@mW#`

## Devices

- `MX-HD Mount`: the main INDIGO mount device
- `MX-HD Mount (guider)`: pulse-guide device using the mount's open serial connection

The mount and guider devices share the same serial connection and can be connected in either order.

## Build

Install the official INDIGO Astronomy package on the target Linux/Raspberry Pi machine, then build:

```bash
echo "deb [trusted=yes] https://indigo-astronomy.github.io/indigo_ppa/ppa indigo main" | sudo tee /etc/apt/sources.list.d/indigo.list
sudo apt update
sudo apt install -y build-essential cmake pkg-config indigo

cd indigo-mxhd
cmake -S . -B build
cmake --build build -j
sudo cmake --install build
```

On Debian/Raspberry Pi OS, the distribution package named `libindigo-dev` can conflict with the official INDIGO Astronomy package. If `indigo` fails to install because `/usr/lib/libindigo.so` already belongs to `libindigo-dev`, remove the distribution package first:

```bash
sudo apt remove -y libindigo-dev
sudo apt install -y indigo
```

The CMake file accepts either an `indigo`/`libindigo` pkg-config package or installed headers plus `libindigo`.

## Run

```bash
indigo_server indigo_mount_mxhd
```

If the driver is installed under `/usr/local/lib/indigo` and `indigo_server` cannot find it by name, run it by full path:

```bash
indigo_server /usr/local/lib/indigo/indigo_mount_mxhd.so
```

Or add a symlink so the short name works:

```bash
sudo ln -sf /usr/local/lib/indigo/indigo_mount_mxhd.so /usr/lib/indigo_mount_mxhd.so
```

The default serial port is `/dev/ttyUSB0` and the default baud rate is `9600`.
USB serial connections typically appear as `/dev/ttyUSB0` or `/dev/ttyACM0`.
Bluetooth serial connections are also supported when the OS exposes the MX-HD link as an RFCOMM serial device such as `/dev/rfcomm0`.
Change the port and baud rate in the INDIGO client using `DEVICE_PORT` and `DEVICE_BAUDRATE`.

The `MX-HD Mount` and `MX-HD Mount (guider)` devices share one serial connection.
Either device can be connected first; the first connected device opens the serial port and the last disconnected device closes it.

For early driver testing, operate `MX-HD Mount` directly. If an INDIGO Mount Agent tries to select `MX-HD Mount` while it is already connected directly, the agent can report that the device is busy or in use.

## Current Scope

Implemented:

- serial connection
- RA/Dec polling
- goto and sync
- site and UTC/time push
- park, unpark via HOME, and HOME
- manual NSEW motion
- sidereal/solar/lunar tracking rates
- tracking on/off
- simple abort
- pulse guide via the separate guider device
- HA-derived side of pier

## Real Hardware Status

Verified on MX-HD hardware:

- driver load via `indigo_server`
- serial connection through `MX-HD Mount`
- RA/Dec polling with changing coordinates
- park with `@Hm#`
- unpark via HOME with `@OG#`
- HOME with `@OG#`
- tracking on/off with `@FD1#` / `@FD0#`
- manual NSEW motion
- goto with `:Sr`, `:Sd`, `:MS#`
- abort during goto
- guider-device connection after mount connection
- pulse guide command/state flow with `@mE#`, `@mW#`, `@mN#`, `@mS#` and timed `:Q#`

Still worth validating further on real hardware:

- exact baud rate used by each MX-HD connection mode
- abort behavior while HOME is active
- whether `:MS#` returns only `0` on all firmware versions
- pulse-guide direction mapping against actual star motion
- long-running guide pulses during simultaneous mount polling

## Version Notes

The project release version is `v0.1.0`.
The INDIGO Control Panel may show the driver version as `2.0.0.1`.
This corresponds to the internal INDIGO driver revision `DRIVER_VERSION 0x0001`.

## Upstream PR Status

This standalone repository contains the initial MX-HD INDIGO driver.
For upstream submission to `indigo-astronomy/indigo`, the driver has also been refactored for the INDIGO 3.0 API on the `mount-mxhd-indigo3` branch of `botchASTRO/indigo`.

The standalone INDIGO 2.0 driver also carries the review-driven cleanup where it can be applied without changing the target API:

- added INDIGO-style copyright, disclaimer and author headers
- replaced platform-specific serial read/write loops with INDIGO `indigo_read()` and `indigo_write()` wrappers
- replaced custom sexagesimal parsing/formatting with `indigo_stod()` and `indigo_dtos()`
- moved device communication out of property handlers into scheduled callbacks
- removed the required connection order between `MX-HD Mount` and `MX-HD Mount (guider)`
- updated single-statement `if`/`else` blocks to use braces consistently

The INDIGO 3.0 refactor addresses upstream review feedback:

- added INDIGO-style copyright, disclaimer and author headers
- replaced platform-specific serial I/O with INDIGO 3.0 `indigo_uni_*` I/O wrappers
- replaced custom sexagesimal parsing/formatting with `indigo_stod()` and `indigo_dtos()`
- moved device communication out of property handlers into scheduled callbacks
- removed the required connection order between `MX-HD Mount` and `MX-HD Mount (guider)`
- updated single-statement `if`/`else` blocks to use braces consistently

## License

This project is licensed under the INDIGO Astronomy open-source license v3.0.
See `LICENSE.md`.
