# Installation

PowerDash is designed for Linux systems using systemd.

It currently expects an Intel CPU supported by `turbostat` and an NVIDIA GPU supported by `nvidia-smi`.

## 1. Install the requirements

PowerDash needs:

- Bash
- awk
- systemd
- turbostat
- nvidia-smi

On Fedora, `turbostat` is provided by `kernel-tools`:

```bash
sudo dnf install kernel-tools
```

Make sure the NVIDIA driver is installed and working:

```bash
nvidia-smi
```

Test CPU power reporting:

```bash
sudo turbostat --quiet --interval 1 --num_iterations 1 --show PkgWatt
```

Both commands should return valid power readings before continuing.

## 2. Install the commands

From the PowerDash repository:

```bash
sudo install -m 755 bin/powerdash /usr/local/bin/powerdash
sudo install -m 755 bin/power-estimator /usr/local/bin/power-estimator
sudo install -m 755 bin/power-trend /usr/local/bin/power-trend
sudo install -m 755 bin/monthly-cost /usr/local/bin/monthly-cost
```

Confirm that the commands are installed:

```bash
command -v powerdash
command -v power-estimator
command -v power-trend
command -v monthly-cost
```

They should resolve to paths under `/usr/local/bin`.

## 3. Install the configuration

Copy the example configuration:

```bash
sudo cp config/powerdash.conf.example /etc/powerdash.conf
```

Edit it:

```bash
sudo nano /etc/powerdash.conf
```

At minimum, review:

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

The default persistent data directory is:

```text
/var/lib/powerdash
```

The default runtime directory is:

```text
/run/powerdash
```

## 4. Install the systemd units

Copy the unit files:

```bash
sudo cp systemd/power-estimator.service /etc/systemd/system/
sudo cp systemd/power-trend.service /etc/systemd/system/
sudo cp systemd/power-trend.timer /etc/systemd/system/
```

Reload systemd:

```bash
sudo systemctl daemon-reload
```

Enable and start the collector:

```bash
sudo systemctl enable --now power-estimator.service
```

Enable and start the trend timer:

```bash
sudo systemctl enable --now power-trend.timer
```

## 5. Check the services

Check the collector:

```bash
systemctl status power-estimator.service
```

Check the timer:

```bash
systemctl status power-trend.timer
```

See when the next trend snapshot will run:

```bash
systemctl list-timers power-trend.timer
```

Follow collector logs:

```bash
sudo journalctl -u power-estimator.service -f
```

## 6. Open the dashboard

Run:

```bash
powerdash
```

PowerDash automatically chooses a compact or full layout based on terminal height.

You can select a layout manually:

```bash
powerdash --compact
```

or:

```bash
powerdash --full
```

For a continuously refreshed display:

```bash
watch -n 5 powerdash
```

## 7. View the cost report

Run:

```bash
monthly-cost
```

This calculates energy and cost estimates from the collected historical power data.

## Data files

PowerDash stores persistent data in:

```text
/var/lib/powerdash/
```

The main files are:

```text
power.log
trend.log
```

Temporary runtime data is stored in:

```text
/run/powerdash/
```

The runtime files include:

```text
current.env
rolling.csv
```

Runtime files are recreated after reboot.

## Changing the configuration

Edit:

```bash
sudo nano /etc/powerdash.conf
```

Then restart the collector:

```bash
sudo systemctl restart power-estimator.service
```

The dashboard and reporting commands read the same configuration file.

## Uninstalling

Stop and disable the services:

```bash
sudo systemctl disable --now power-estimator.service
sudo systemctl disable --now power-trend.timer
```

Remove the systemd units:

```bash
sudo rm /etc/systemd/system/power-estimator.service
sudo rm /etc/systemd/system/power-trend.service
sudo rm /etc/systemd/system/power-trend.timer
sudo systemctl daemon-reload
```

Remove the commands:

```bash
sudo rm /usr/local/bin/powerdash
sudo rm /usr/local/bin/power-estimator
sudo rm /usr/local/bin/power-trend
sudo rm /usr/local/bin/monthly-cost
```

Remove the configuration:

```bash
sudo rm /etc/powerdash.conf
```

PowerDash data is not removed automatically.

If you also want to remove the collected history:

```bash
sudo rm -rf /var/lib/powerdash
```