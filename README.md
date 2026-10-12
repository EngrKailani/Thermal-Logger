# Thermal Controller & Logger

**Original source code recovered.** This repository contains the Python desktop application for the thermal validation workflow developed at LUX Dynamics. The source has been reviewed for architecture and basic configuration, but the present recovery has **not** been verified with a Pico TC-08 or Arduino on an instrumented test bench.

## What is implemented

- Tkinter desktop interface with test metadata and channel configuration.
- Pico TC-08 thermocouple acquisition through the Pico `usbtc08.dll` driver.
- Channels 1 through 8, with optional cold-junction (channel 0).
- Approximately one UI-scheduled read per second by default (`SAMPLE_INTERVAL = 1.0`). This is not a guaranteed measured sampling rate.
- Separate live plotting window with channel selection and pan/zoom.
- Trend display based on a rolling window, range, mean, and end-to-end delta.
- Optional Arduino ambient controller over serial, with fixed or stepped temperature setpoints.
- CSV logging and optional color-coded Excel export using `openpyxl`.
- A preconfigured Windows/network-drive output folder, with a fallback to a local `logs/` folder.

## Source file map

| File | Responsibility |
|---|---|
| `main_ui.pyw` | Entry point, graphical interface, test setup, logging, and CSV/Excel export |
| `logger_core.py` | Shared configuration and Arduino serial protocol |
| `tc08_interface.py` | Windows driver wrapper and Pico TC-08 acquisition |
| `graph_window.py` | Live Tkinter temperature plot |
| `LUX_Thermal_Logger.spec` | PyInstaller packaging recipe |
| `usbtc08.dll` | TC-08 vendor driver supplied with the archived source |

## Running the original application on Windows

You need Windows, a compatible Python installation, a working Pico TC-08 with connected thermocouples, and a vendor driver compatible with the Python process.

```powershell
git clone https://github.com/EngrKailani/Thermal-Logger.git
cd Thermal-Logger
py -m pip install openpyxl pyserial pywin32
py main_ui.pyw
```

You can also use `python main_ui.pyw` when Python is on your PATH. **There is no `main_logger.py` or `run_logger.bat` in this repository.** A prior version of this README incorrectly referenced another repository (`PID_UI`) and those unavailable files.

The `openpyxl` package supports Excel export; `pyserial` supports the optional Arduino connection; `pywin32` supports desktop shortcut integration. Tkinter is normally included with standard Python Windows distributions. The GUI may prompt to install optional missing packages.

This original code loads the vendor `usbtc08.dll` when importing `tc08_interface.py`, so **it is not a hardware-free demo** and is not expected to run unchanged on Linux or macOS. Use an appropriate driver, connected TC-08, and matching Python/driver architecture. Vendor-licensed materials must be used in accordance with their licenses.

## Optional Arduino control

The protocol implemented in `logger_core.py` sends newline-terminated ASCII:

```text
SET:40.00
AMB:38.42
```

The Arduino is expected to return data resembling:

```text
AMB:38.42,HOLD:40.00,PWM:155,STATUS:OK
```

Older `TEMP:` readings are accepted by the parser. The serial link defaults to **9600 baud**. The companion Arduino firmware and electrical wiring diagram have **not been recovered here**; do not assume arbitrary hardware is compatible or connect an unverified heater/control circuit.

## Export format

The CSV starts with a test metadata row, a blank row, then a column header beginning with `Data_Sample`. Selected thermocouple temperatures use `<channel name>_C`; Arduino runs also add `Ambient_Feedback_C`, `Ambient_Setpoint_C`, and `Arduino_PWM`.

The sample number is **not an actual timestamp**. Do not infer exact time intervals or demonstrate instrument timing accuracy from this field alone.

## Recovery and engineering records

See [Thermal Controller & Logger in the engineering portfolio](https://github.com/EngrKailani/engrkailani.github.io/tree/main/projects/thermal) for an independently written architecture description, source review and verification plan.

The surviving source is not evidence that every feature was revalidated during recovery. Raw company thermal data, internal reports, and undocumented test results are intentionally not reproduced.

## Attribution

Developed by Kailani Puava Alarcon for thermal-test automation at LUX Dynamics. This public repository preserves the current available source snapshot. Hardware operation and numerical measurement accuracy require fresh validation.
