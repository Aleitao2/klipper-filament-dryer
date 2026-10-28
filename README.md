# Klipper Filament Dryer

![status](https://img.shields.io/badge/status-working-brightgreen) ![Klipper](https://img.shields.io/badge/Klipper-config-B12F35) ![STM32](https://img.shields.io/badge/MCU-STM32G0B1-03234B)

A 3D-printer mainboard repurposed as a **filament dry-box controller** running [Klipper](https://www.klipper3d.org/).
The whole application — heater control, fan cycling, countdown timer and a custom LCD menu — is written as
Klipper configuration and Jinja2 G-code macros, with no custom firmware.

## Hardware
- STM32G0B1 printer mainboard running Klipper (`kinematics: none` — no motion system)
- Bed-heater output driving the dryer heater + EPCOS 100K thermistor, **PID-tuned**
- Circulation fan (always on while drying) and exhaust fan (venting cycle)
- Sovol 12864 screen with rotary encoder and beeper (`emulated_st7920`)

## Features
- **One-click profiles** from the LCD: PLA 60 °C / 6 h · PETG 70 °C / 9 h · ABS 80 °C / 8 h
- **Manual mode**: pick 20–90 °C and 1–24 h with the encoder, then *Confirm & Start*
- **Countdown timer** on the status line (`Time Left: 3h 42m`), auto-stop when it reaches zero
- **Exhaust venting cycle**: 30 s on / 120 s off to purge moist air
- **Safety**: max temp 95 °C, 24 h idle-timeout failsafe that turns everything off
- Default Klipper menus stripped down to *Drybox → Profiles / Manual Setup*, *STOP DRYING*, *Setup*

## How it works
| Macro | Role |
|---|---|
| `DRYING_STATE` | Holds state variables (manual temp/time, minutes remaining, drying flag) |
| `START_DRYING TEMP= TIME_HOURS=` | Sets heater, starts fans, kicks off the vent loop and timer |
| `STOP_DRYING` | Resets state, turns off heater/fans, cancels all delayed G-code loops |
| `DRYBOX_TICK` (delayed_gcode) | Runs every 60 s: updates the display and decrements the timer |
| `VENT_CYCLE_START/STOP` | Two delayed_gcode blocks that call each other to make the vent cycle |

## Install
Copy `printer.cfg` to your Klipper config folder, replace the `[mcu] serial:` with your board's
`/dev/serial/by-id/...` path, check the pin names against your board, and re-run `PID_CALIBRATE HEATER=heater_bed TARGET=60`.
