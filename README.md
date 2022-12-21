# Fwiffo's Voron Klipper Config

This repository contains the configuration options for Klipper,
Mainsail/Fluidd, Moonraker, etc. for my Voron printers (currently two V0.1 with
a V2.4 coming in the future.)

Along with printer-specific configs, there are a number of macros, etc. that
others might find useful.

## Fancy `PRINT_START` and `PRINT_END` macros

The `PRINT_START` macro simplifies the slicer configuration in addition to
adding a lot of automation and functionality.

### Heat soak

When I previously wanted to do a heat-soak to warm up the chamber before
printing ABS, I would set the temperature, but would then forget to go back and
start the actual print. I added some macros to manage heat soaks automatically.

You can run `SET_HEAT_SOAK TIME=[time in seconds]` to request an heat soak at
the start of the next print. The soak time will be shown on the printer display.
I recommend putting options for this in the temperature pre-set menu in
Mainsail/Fluidd (e.g. 5, 10, 20 minute soaks).

The next time you start a print, a countdown will start after the bed has warmed
up, allowing the chamber to come up to temperature before resuming the print.

TODO: Add an automatic heat soak based on chamber temperature if a sensor is
present.

### Improved heater behavior

Generally, the hotend warms up much faster than the heated bed, so if you start
heating them at the same time, the filament sits there cooking needlessly. This
can actually cause problems with some filaments, and also wastes energy.

The `PRINT_START` macro waits until the bed is nearly up to temperature (and
until after any heat soak has been completed) before turning on the hotend
heater. Both the bed and hotend will get up to temperature at about the same
time.

### Improved homing behavior

To avoid wasting time, the start macro only homes the printhead at the start of
print if it's not already homed. It then centers the printhead over the bed to
making heating as even as possible. Immediately before starting the print, it
then homes again to account for any movement/expansion during heating.

The second homing can be disabled with the `SKIP_HOME` parameter to
`PRINT_START` in case you're printing with multiple filaments in succession.
This reduces the chance of misalignment. There is also a `STAY_HOT` option for
`PRINT_END` to keep the heaters on.

### Per-print or per-filament settings management

If you set custom speed limits or pressure advance for a specific filament or
print using `SET_VELOCITY_LIMIT` or `SET_PRESSURE_ADVANCE` the defaults from
your config will be restored at the end of print, or if the print is cancelled.

### Automatic Nevermore management

If you pass the filament type to the `PRINT_START` macro, it will automatically
set the fan speed for the Nevermore air filter (if present). You can also set
the speed in your filament custom gcode (e.g. `NEVERMORE SPEED=0.5`). The
Nevermore turns off automatically 5 minutes after printing has completed.

TODO: Add slicer and filament `PRINT_START` and `PRINT_END` config here.

### Other functionality

  * Lots of status/LED updates during startup/printing/pause/resume/end/cancel
  * Jerks printhead at end of print to remove string
  * Parks the head in a useful place automatically based on printer geometry

## Fancy functionality for [V0 display](https://github.com/VoronDesign/Voron-Hardware/tree/master/V0_Display)

 * Nevermore fan speed, if present - alternates with part cooling fan speed
 * Chamber temperature if a chamber sensor is present - alternates with bed
   temperature
 * Filename during printing (with scrolling) - alternates with `M117` status
 * Time remaining during printing, if available from `M73`, otherwise estimates
   based on print progress
 * Elapsed time at end of print
 * Time requested for heat soak (or remaining time) if a heat soak is requested
   or in progress

## Acceleration limits

The `max_accel` configuration option and `SET_VELOCITY_LIMITS ACCEL=...`
commands in Klipper don't actually set a maximum or limit to acceleration; they
just set a default. Acceleration values produced by your slicer (`M204`) simply
override them.

This is inconsistent with other limits set in Klipper firmware (velocity, build
volume, temperature, etc.) and contrary to their naming and the Klipper
documentation. Generally, firmware limits are provided to protect the printer in
the event of an inadvertant slicer misconfiguration, but since the acceleration
settings are ignored, they serve no purpose.

[This is apparently intended behavior](https://github.com/Klipper3d/klipper/blob/master/docs/Config_Changes.md) (20210430).

To make things work as expected, I've overriden `M204` to respect these limits.

## Misc. quality-of-life macros

  * Macro to conditionally home the printer only if it's not already homed
  * Useful automatic parking macro and improved homing behavior
  * Smart printhead-centering macro
  * Macro to set fan speed only if it exists to make it easier to share macros
    across multiple printers
  * Mini-afterburner appropriate filament load/unload macros

```
PARK_USEFULLY [SPEED=6000]
HOME_IF_NOT_HOMED [SPEED=6000]
CENTER_PRINTHEAD
SET_FAN_SPEED_IF_EXISTS FAN=fan_name SPEED=0.5
LOAD_FILAMENT
UNLOAD_FILAMENT
```

## Other things

  * Reasonable starting points for lots of V0 settings
  * A corrected thermistor configuration for Keenovo bed heaters
  * Configuration for adxl345 accelerometer for input shaper
  * Configuration for Klipper expander, Nevermore air filter, and BME280-family
    environmental sensors
