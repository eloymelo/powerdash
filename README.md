# PowerDash

PowerDash is a terminal power monitor for Linux servers.

It reads CPU package power from `turbostat` and GPU power from `nvidia-smi`, then adds a configurable estimate for the rest of the system and applies a PSU efficiency model.

The result is an estimated wall power figure together with daily, monthly, and yearly energy cost projections.

PowerDash was originally built for monitoring a home server, but the configuration is kept separate so it can be adapted to other systems.

## What it shows

PowerDash tracks:

- CPU package power
- NVIDIA GPU power
- Estimated platform power
- Estimated wall power
- Rolling average power
- Long term average, minimum, and maximum power
- Daily energy estimate
- Monthly energy estimate
- Yearly energy estimate
- Estimated electricity cost
- Periodic trend snapshots

The main dashboard can automatically switch between compact and full layouts depending on terminal size.

## Requirements

PowerDash currently expects:

- Linux
- Bash
- systemd
- `awk`
- `turbostat`
- `nvidia-smi`
- An Intel CPU supported by `turbostat`
- An NVIDIA GPU supported by `nvidia-smi`

Support for other hardware is not currently implemented.

## How it works

The collector reads CPU and GPU power values at regular intervals.

The basic model is:

```text
Internal power =
    CPU package power
  + GPU power
  + fixed platform estimate

Estimated wall power =
    internal power / estimated PSU efficiency
```

The fixed platform estimate represents components that are not measured directly, such as the motherboard, memory, storage, fans, and controllers.

PowerDash stores historical power readings under:

```text
/var/lib/powerdash
```

Temporary runtime state is stored under:

```text
/run/powerdash
```

The default configuration file is:

```text
/etc/powerdash.conf
```

## Commands

Show the dashboard:

```bash
powerdash
```

Force the compact view:

```bash
powerdash --compact
```

Force the full view:

```bash
powerdash --full
```

Watch the dashboard continuously:

```bash
watch -n 5 powerdash
```

Show the monthly estimate directly:

```bash
monthly-cost
```

## Configuration

An example configuration is included at:

```text
config/powerdash.conf.example
```

The main settings are:

```text
RATE
CURRENCY_SYMBOL
FIXED_LOAD_W
PSU_EFF_LOW
PSU_EFF_HIGH
PSU_SWITCH_W
CALIBRATION_FACTOR
SAMPLE_INTERVAL
AVG_WINDOW
DATA_DIR
RUNTIME_DIR
```

Copy the example configuration to:

```text
/etc/powerdash.conf
```

and change the values for your system.

For installation instructions, see [`docs/installation.md`](docs/installation.md).

## Accuracy

PowerDash is a software estimate.

It does not measure total power directly from the electrical outlet.

CPU and GPU power come from software accessible sensors, while the rest of the system is represented by a configurable fixed estimate. PSU efficiency is also modeled rather than measured.

For better accuracy, compare PowerDash against a physical wall power meter and adjust `FIXED_LOAD_W` and `CALIBRATION_FACTOR`.

PowerDash skips samples when CPU or GPU power cannot be read instead of inserting assumed fallback values into the historical average.

## Project structure

```text
powerdash/
├── bin/
│   ├── monthly-cost
│   ├── power-estimator
│   ├── power-trend
│   └── powerdash
├── config/
│   └── powerdash.conf.example
├── docs/
│   └── installation.md
├── systemd/
│   ├── power-estimator.service
│   ├── power-trend.service
│   └── power-trend.timer
├── LICENSE
└── README.md
```

## !!! TESTING STATUS !!! PLEASE READ !!!

PowerDash is currently maintained without access to a dedicated Linux server for hardware testing.

The shell scripts are checked for syntax errors and the project structure is kept consistent, but the current release has not yet been validated end to end on a clean Linux installation.

If you test PowerDash on compatible hardware, bug reports and installation feedback are welcome.

## License

PowerDash is available under the MIT License.