# _Psychometrics Calculator for Moist Air_

A Python module for calculating thermodynamic properties of moist air, based on ASHRAE Handbook of Fundamentals 2009, Chapter 1.

## _Overview_

This module provides a single public function — `state()` — that accepts any two independent psychrometric properties and an atmospheric pressure, then returns all six standard properties of the moist air state.

All inputs and outputs use base SI units.

## _Units_

| Symbol | Property                  | Unit                  |
|--------|---------------------------|-----------------------|
| DBT    | Dry bulb temperature      | Kelvin (K)            |
| WBT    | Wet bulb temperature      | Kelvin (K)            |
| DPT    | Dew point temperature     | Kelvin (K)            |
| RH     | Relative humidity         | Decimal (e.g. `0.5`)  |
| W      | Humidity ratio            | kg/kg                 |
| H      | Specific enthalpy         | kJ/kg                 |
| V      | Specific volume           | m³/kg                 |
| P      | Atmospheric pressure      | Pascals (Pa)          |
| Pw     | Water vapor partial pressure | Pascals (Pa)       |

> Note: Relative humidity is a decimal fraction, not a percentage. `0.5` means 50% RH.

## _Public API_

### `state(prop1, prop1val, prop2, prop2val, P)`

Calculates a complete moist air state from two known independent properties and the atmospheric pressure.

#### _Parameters_

| Parameter  | Type    | Description                                      |
|------------|---------|--------------------------------------------------|
| `prop1`    | `str`   | Name of the first known property (see below)     |
| `prop1val` | `float` | Value of the first property in SI units          |
| `prop2`    | `str`   | Name of the second known property (see below)    |
| `prop2val` | `float` | Value of the second property in SI units         |
| `P`        | `float` | Atmospheric pressure in Pascals                  |

Valid property names:** `"DBT"`, `"WBT"`, `"RH"`, `"W"`, `"V"`, `"H"`

#### _Returns_

A list of six values in the following order:

```python
[DBT, H, RH, V, W, WBT]
```

| Index | Property          | Unit   |
|-------|-------------------|--------|
| 0     | DBT               | K      |
| 1     | H                 | kJ/kg  |
| 2     | RH                | decimal|
| 3     | V                 | m³/kg  |
| 4     | W                 | kg/kg  |
| 5     | WBT               | K      |

## _Example Usage_

```python
import psychrometrics as psy

# Standard atmospheric pressure at sea level
P = 101325  # Pa

# Calculate state from dry bulb temperature (25°C) and relative humidity (60%)
DBT = 25 + 273.15   # Convert °C to K
RH  = 0.60          # 60%

result = psy.state("DBT", DBT, "RH", RH, P)

DBT, H, RH, V, W, WBT = result

print(f"Dry Bulb Temp:    {DBT - 273.15:.2f} °C")
print(f"Wet Bulb Temp:    {WBT - 273.15:.2f} °C")
print(f"Enthalpy:         {H:.2f} kJ/kg")
print(f"Humidity Ratio:   {W*1000:.2f} g/kg")
print(f"Relative Humidity:{RH*100:.1f} %")
print(f"Specific Volume:  {V:.4f} m³/kg")
```

## _Supported Property Pairs_

Any combination of two independent properties from the list below is valid:

|       | WBT | RH | W | V | H |
|-------|-----|----|---|---|---|
| DBT | ✓ | ✓ | ✓ | ✓ | ✓ |
| WBT | —   | ✓ | ✓ | ✓ | ✓ |
| RH  |     | — | ✓ | ✓ | ✓ |
| W   |     |   | — | ✓ | ✓ |
| V   |     |   |   | — | ✓ |

## _Temperature Limits_

The module is valid over the following dry bulb temperature range:

| Bound   | Value         |
|---------|---------------|
| Minimum | 273.15 K (0°C)   |
| Maximum | 473.15 K (200°C) |

Inputs outside this range will cause the calculation to return `None`.

## _ASHRAE References_

| Function             | Reference                            |
|----------------------|--------------------------------------|
| Saturation pressure  | ASHRAE 2009 Ch.1, Eq. 6              |
| Humidity ratio       | ASHRAE 2009 Ch.1, Eq. 22 & 24        |
| Specific volume      | ASHRAE 2009 Ch.1, Eq. 28             |
| Wet bulb temperature | ASHRAE 2009 Ch.1, Eq. 35             |
| Specific enthalpy    | ASHRAE 2009 Ch.1, Eq. 32             |
| Dew point            | ASHRAE 2009 Ch.1, Eq. 39             |

## _Notes_

- Internal helper functions are prefixed with `__` and are not part of the public API.
- Unknown states involving two non-DBT properties are solved iteratively using the bisection method with a convergence tolerance of `0.0005 K`.
