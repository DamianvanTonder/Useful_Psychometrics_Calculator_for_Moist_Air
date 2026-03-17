# _User Guide - Psychrometrics Calculator_
## _Introduction_
This module calculates thermodynamic properties of moist air for use in HVAC, building science, and environmental engineering applications. Given any two known independent properties of a moist air sample and the atmospheric pressure, the module computes the full psychrometric state.

All calculations follow the ASHRAE Handbook of Fundamentals 2009, Chapter 1.

## _Getting Started_
### _Prerequisites_
- Python 3.x
- Standard library only — no external packages required

### _Installation_
Place the module file in your project directory and import it:

```python
import psychrometrics as psy
```

## _Unit Conventions_
> Important: All inputs and outputs use base SI units. Temperatures must be in Kelvin — not Celsius or Fahrenheit. Relative humidity is a decimal fraction, not a percentage.

| Property             | Symbol | Unit         | Example          |
|----------------------|--------|--------------|------------------|
| Dry bulb temperature | DBT    | Kelvin (K)   | 298.15 K = 25°C  |
| Wet bulb temperature | WBT    | Kelvin (K)   | 291.15 K = 18°C  |
| Relative humidity    | RH     | Decimal      | 0.60 = 60%       |
| Humidity ratio       | W      | kg/kg        | 0.012            |
| Specific enthalpy    | H      | kJ/kg        | 55.4             |
| Specific volume      | V      | m³/kg        | 0.855            |
| Atmospheric pressure | P      | Pascals (Pa) | 101325 Pa        |

### _Temperature Conversion_
```python
# Celsius to Kelvin
T_kelvin = T_celsius + 273.15

# Kelvin to Celsius
T_celsius = T_kelvin - 273.15
```

## _The `state()` Function_
This is the only public function in the module.

### _Signature_
```python
state(prop1, prop1val, prop2, prop2val, P)
```

### _Parameters_
| Parameter   | Type    | Description                                   |
|-------------|---------|-----------------------------------------------|
| `prop1`     | `str`   | Name of the first known property              |
| `prop1val`  | `float` | Value of the first property in SI units       |
| `prop2`     | `str`   | Name of the second known property             |
| `prop2val`  | `float` | Value of the second property in SI units      |
| `P`         | `float` | Atmospheric pressure in Pascals               |

### _Valid Property Names_
`"DBT"`, `"WBT"`, `"RH"`, `"W"`, `"V"`, `"H"`

### _Return Value_
A list of six values in the following fixed order:

```python
[DBT, H, RH, V, W, WBT]
```

| Index | Property             | Unit    |
|-------|----------------------|---------|
| `[0]` | Dry bulb temperature | K       |
| `[1]` | Specific enthalpy    | kJ/kg   |
| `[2]` | Relative humidity    | decimal |
| `[3]` | Specific volume      | m³/kg   |
| `[4]` | Humidity ratio       | kg/kg   |
| `[5]` | Wet bulb temperature | K       |

## _Usage Examples_
### _Example 1 — DBT and RH (most common case)_
```python
import psychrometrics as psy

P   = 101325        # Standard atmospheric pressure, Pa
DBT = 25 + 273.15   # 25°C in Kelvin
RH  = 0.60          # 60% relative humidity

result = psy.state("DBT", DBT, "RH", RH, P)
DBT, H, RH, V, W, WBT = result

print(f"Dry Bulb Temp:     {DBT - 273.15:.2f} °C")
print(f"Wet Bulb Temp:     {WBT - 273.15:.2f} °C")
print(f"Enthalpy:          {H:.2f} kJ/kg")
print(f"Humidity Ratio:    {W * 1000:.2f} g/kg")
print(f"Relative Humidity: {RH * 100:.1f} %")
print(f"Specific Volume:   {V:.4f} m³/kg")
```

Output:
```
Dry Bulb Temp:     25.00 °C
Wet Bulb Temp:     18.60 °C
Enthalpy:          55.38 kJ/kg
Humidity Ratio:    11.89 g/kg
Relative Humidity: 60.0 %
Specific Volume:   0.8566 m³/kg
```

### _Example 2 — DBT and WBT_
```python
DBT = 30 + 273.15   # 30°C
WBT = 22 + 273.15   # 22°C
P   = 101325

result = psy.state("DBT", DBT, "WBT", WBT, P)
DBT, H, RH, V, W, WBT = result
```

### _Example 3 — Enthalpy and Relative Humidity_
```python
H  = 60.0    # kJ/kg
RH = 0.50    # 50%
P  = 101325

result = psy.state("H", H, "RH", RH, P)
DBT, H, RH, V, W, WBT = result
```

### _Example 4 — High-Altitude Site (reduced pressure)_
```python
# Johannesburg, South Africa (~1750 m elevation)
P   = 82500         # Approximate pressure at 1750 m, Pa
DBT = 28 + 273.15
RH  = 0.40

result = psy.state("DBT", DBT, "RH", RH, P)
```

## _Supported Property Combinations_
Any two independent properties may be provided. The table below shows all valid pairs:

|         | WBT | RH | W | V | H |
|---------|:-------:|:------:|:-----:|:-----:|:-----:|
| DBT | ✓       | ✓      | ✓     | ✓     | ✓     |
| WBT | —       | ✓      | ✓     | ✓     | ✓     |
| RH  |         | —      | ✓     | ✓     | ✓     |
| W   |         |        | —     | ✓     | ✓     |
| V   |         |        |       | —     | ✓     |

> Providing the same property twice (e.g., `"DBT"` and `"DBT"`) will print an error and return nothing.

## _Valid Temperature Range_
The module is valid for dry bulb temperatures between:

| Bound   | Kelvin    | Celsius |
|---------|-----------|---------|
| Minimum | 273.15 K  | 0°C     |
| Maximum | 473.15 K  | 200°C   |

Inputs outside this range will cause internal functions to return `None`, and the result will be undefined.

## _Error Handling_
The function will print a message and return `None` in two cases:

| Condition                          | Message printed                        |
|------------------------------------|----------------------------------------|
| `prop1` and `prop2` are the same   | `"Properties must be independent."`   |
| An invalid property name is given  | `"Valid property must be given."`      |

Always validate inputs before passing them to `state()` if your application may receive unexpected values.

## _Notes_
- The property pair order does not matter — `state("DBT", ..., "RH", ...)` and `state("RH", ..., "DBT", ...)` produce identical results.
- Cases where DBT is not one of the two known properties are solved iteratively using the bisection method with a convergence tolerance of 0.0005 K.
- All internal helper functions are private (prefixed with `__`) and should not be called directly.
