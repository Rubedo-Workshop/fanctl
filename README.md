# fanctl

Fan control daemon for the Mac Mini 2018 (Macmini8,1) running Linux with the `applesmc_t2` kernel module.

The Mac Mini's T2 chip manages the fan by default, but under Linux it tends to run the fan too conservatively — temps can climb silently during heavy workloads. fanctl takes over and runs a configurable temperature-to-RPM curve instead.

## Requirements

- Mac Mini 2018 (Macmini8,1)
- Linux with `applesmc_t2` kernel module loaded
- Tested on CachyOS, should work on any Arch-based distro

## Install

```bash
sudo cp fanctl /usr/local/bin/fanctl
sudo chmod +x /usr/local/bin/fanctl
sudo cp fanctl.conf /etc/fanctl.conf
sudo cp fanctl.service /etc/systemd/system/fanctl.service
sudo systemctl daemon-reload
sudo systemctl enable --now fanctl
```

## Commands

```
fanctl status         Show current mode, fan RPM, and max temp
fanctl dynamic        Temperature-based curve (default on start)
fanctl auto           Hand control back to T2 firmware
fanctl set <RPM>      Lock fan to a specific RPM, e.g: fanctl set 3500
fanctl turbo          Set fan to max overclock RPM (defined in fanctl.conf)
```

## Configuration

Edit `/etc/fanctl.conf` to tune the curve — changes are picked up within one poll interval, no restart needed.

```bash
# Temperature curve: "TEMP_C:RPM ..." in ascending order
CURVE="45:1700 55:2500 65:3200 72:3800 78:4400"
```

The daemon uses the **maximum** temp across all monitored sensors, so it reacts to any single core spiking rather than waiting for overall heat to rise.

| Temp | RPM |
|------|-----|
| ≤ 45°C | 1700 (minimum) |
| 55°C | 2500 |
| 65°C | 3200 |
| 72°C | 3800 |
| ≥ 78°C | 4400 |
| ≥ 85°C | 5000 (emergency turbo) |

When temp hits `EMERGENCY_TEMP` (default 85°C), the daemon bypasses the curve and jumps straight to `OVERCLOCK_RPM`. It drops back to the normal curve once temps fall below the threshold.

## Hardware limits

- Minimum RPM: 1700
- Maximum RPM: 4400
- `OVERCLOCK_RPM` in config defaults to 5000 — the hardware will cap it at its physical limit
- `EMERGENCY_TEMP` in config defaults to 85°C — auto-triggers turbo in dynamic mode
