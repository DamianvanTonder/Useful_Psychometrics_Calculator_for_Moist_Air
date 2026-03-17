# _Requirements_
Document Version: 1.0  
Standard Reference: ASHRAE Handbook of Fundamentals 2009, Chapter 1

## _Introduction_
### _Purpose_
This document specifies the functional and non-functional requirements for the Psychrometrics Python module. The module calculates the thermodynamic properties of moist air given any two independent psychrometric properties and an atmospheric pressure value.

### _Scope_
The module provides a single public function, `state()`, which resolves a complete psychrometric state. It is intended for use in HVAC analysis, building energy simulation, meteorological tools, and any engineering application requiring moist air property calculations.

### _Definitions_
| Term             | Definition                                                                 |
|------------------|----------------------------------------------------------------------------|
| DBT              | Dry bulb temperature — thermodynamic temperature of the air, in Kelvin     |
| WBT              | Wet bulb temperature — adiabatic saturation temperature, in Kelvin         |
| RH               | Relative humidity — ratio of actual to saturation vapor pressure (decimal) |
| W                | Humidity ratio — mass of water vapor per mass of dry air, kg/kg            |
| H                | Specific enthalpy of moist air, kJ/kg                                      |
| V                | Specific volume of moist air, m³/kg                                        |
| P                | Atmospheric pressure, Pascals                                              |
| Pws              | Saturation pressure of water vapor at a given temperature, Pascals         |
| Pw               | Partial pressure of water vapor, Pascals                                   |
| DPT              | Dew point temperature, Kelvin                                              |
| Bisection method | Iterative root-finding algorithm used to solve implicit property equations  |

### _References_
- ASHRAE Handbook of Fundamentals 2009, Chapter 1 — Psychrometrics

## _Overall Description_
### _Product Perspective_

The module is a standalone Python library with no external dependencies beyond the Python standard library. It is intended to be imported into larger applications or used directly in scripts.

### _Operating Environment_
- Python 3.x runtime
- Any operating system supporting Python 3 (Windows, macOS, Linux)
- No external packages, network access, or file system access required

### _Constraints_
- All inputs and outputs must use base SI units as defined in Section 1.3.
- Temperature inputs outside the valid range (273.15 K to 473.15 K) produce undefined results.
- The module does not perform unit conversion; that responsibility lies with the caller.

## _Functional Requirements_
### _Public Interface_
#### _FR-01 — `state()` Function_
The module shall expose a single public function with the following signature:

```python
state(prop1, prop1val, prop2, prop2val, P) → list
```

#### _FR-02 — Valid Property Names_
The function shall accept the following strings as property identifiers: `"DBT"`, `"WBT"`, `"RH"`, `"W"`, `"V"`, `"H"`.

#### _FR-03 — Return Value_
On success, the function shall return a list of six floats in the order:

```
[DBT, H, RH, V, W, WBT]
```

Units shall be: K, kJ/kg, decimal, m³/kg, kg/kg, K respectively.

#### _FR-04 — Property Pair Order Independence_
The function shall produce identical results regardless of which property is passed as `prop1` or `prop2`.

#### _FR-05 — Invalid Property Names_
If either `prop1` or `prop2` is not one of the six valid property strings, the function shall print `"Valid property must be given."` and return `None`.

#### _FR-06 — Dependent Properties_
If `prop1` and `prop2` are the same string, the function shall print `"Properties must be independent."` and return `None`.

#### _FR-07 — Supported Property Pairs_
The function shall correctly resolve all 15 unique combinations of the six supported properties, as enumerated below:

| Pair         | Pair         | Pair         |
|--------------|--------------|--------------|
| DBT + WBT    | DBT + RH     | DBT + W      |
| DBT + V      | DBT + H      | WBT + RH     |
| WBT + W      | WBT + V      | WBT + H      |
| RH + W       | RH + V       | RH + H       |
| W + V        | W + H        | V + H        |

### _Psychrometric Calculations_
#### _FR-08 — Saturation Pressure_
The module shall compute the saturation pressure of water vapor `Pws(DBT)` using ASHRAE 2009 Chapter 1, Equation 6.

#### _FR-09 — Humidity Ratio from DBT and RH_
The module shall compute `W` from `DBT`, `RH`, and `P` using ASHRAE 2009 Chapter 1, Equations 22 and 24.

#### _FR-10 — Humidity Ratio from DBT and WBT_
The module shall compute `W` from `DBT`, `WBT`, and `P` using ASHRAE 2009 Chapter 1, Equation 35.

#### _FR-11 — Humidity Ratio from DBT and V_
The module shall compute `W` from `DBT`, `V`, and `P` using ASHRAE 2009 Chapter 1, Equation 28.

#### _FR-12 — Humidity Ratio from DBT and H_
The module shall compute `W` from `DBT` and `H` using ASHRAE 2009 Chapter 1, Equation 32.

#### _FR-13 — Specific Enthalpy_
The module shall compute `H` from `DBT` and `W` using ASHRAE 2009 Chapter 1, Equation 32.

#### _FR-14 — Specific Volume_
The module shall compute `V` from `DBT`, `W`, and `P` using ASHRAE 2009 Chapter 1, Equation 28.

#### _FR-15 — Relative Humidity_
The module shall compute `RH` from `DBT`, `W`, and `P` using ASHRAE 2009 Chapter 1, Equations 22 and 24.

#### _FR-16 — Dew Point Temperature_
The module shall compute the dew point temperature from the water vapor partial pressure `Pw` using ASHRAE 2009 Chapter 1, Equation 39. This is used internally as the lower bound when solving for WBT.

#### _FR-17 — Wet Bulb Temperature_
The module shall compute `WBT` from `DBT`, `W`, and `P` by iterative inversion of Equation 35 using the bisection method.

#### _FR-18 — Iterative Solving for DBT_
When `DBT` is not one of the two provided properties, the module shall determine `DBT` iteratively using the bisection method over the valid temperature range.

### _Internal Solver_
#### _FR-19 — Bisection Convergence Tolerance_
The bisection solver shall iterate until the interval width is less than or equal to 0.0005 K.

#### _FR-20 — Bisection Search Range_
The bisection solver shall search over the range `[273.15 K, 473.15 K]` for `DBT`, and `[DPT, DBT]` for `WBT`.

#### _FR-21 — Partial Vapor Pressure_
The module shall compute the partial pressure of water vapor `Pw` from `W` and `P` using ASHRAE 2009 Chapter 1, Equation 22.

## _Non-Functional Requirements_
### _Accuracy_
#### _NFR-01_
All property calculations shall conform to the ASHRAE 2009 formulations. Results shall be consistent with ASHRAE psychrometric chart values within the limits imposed by the bisection convergence tolerance of 0.0005 K.

### _Performance_
#### _NFR-02_
A single call to `state()` shall complete in under one second on any modern general-purpose processor, for all valid input combinations.

### _Portability_
#### _NFR-03_
The module shall depend only on the Python standard library (`math` module). No third-party packages shall be required.

#### _NFR-04_
The module shall be compatible with Python 3.0 and later.

### _Maintainability_
#### _NFR-05_
All private helper functions shall be prefixed with double underscores (`__`) and shall not be part of the public API.

#### _NFR-06_
Each private function shall include an inline comment referencing the corresponding ASHRAE equation number.

### _Reliability_
#### _NFR-07_
The module shall return `None` (and not raise an unhandled exception) when invalid property names or dependent property pairs are provided.

#### _NFR-08_
For inputs within the valid temperature range, no unhandled exceptions shall be raised.

## _Constraints and Assumptions_
| ID  | Description |
|-----|-------------|
| C-01 | Inputs must be in base SI units. No unit conversion is performed by the module. |
| C-02 | Dry bulb temperature must be between 273.15 K (0°C) and 473.15 K (200°C). Behaviour outside this range is undefined. |
| C-03 | Relative humidity must be provided as a decimal fraction in the range [0.0, 1.0]. |
| C-04 | Atmospheric pressure must be a positive value in Pascals. |
| C-05 | The module assumes the ASHRAE 2009 formulations are applicable (i.e., the air is a mixture of dry air and water vapor behaving as ideal gases). |
| C-06 | The WBT bisection lower bound is set to the dew point temperature, which assumes WBT ≥ DPT for physically valid states. |

## _Traceability Matrix_
| Requirement | ASHRAE Reference        | Function(s)                          |
|-------------|-------------------------|--------------------------------------|
| FR-08       | Ch.1 Eq. 6              | `__Pws`                              |
| FR-09       | Ch.1 Eq. 22, 24         | `__W_DBT_RH_P`, `__RH_DBT_W_P`      |
| FR-10       | Ch.1 Eq. 35             | `__W_DBT_WBT_P`                      |
| FR-11       | Ch.1 Eq. 28             | `__W_DBT_V_P`, `__V_DBT_W_P`        |
| FR-12       | Ch.1 Eq. 32             | `__W_DBT_H`                          |
| FR-13       | Ch.1 Eq. 32             | `__H_DBT_W`                          |
| FR-14       | Ch.1 Eq. 28             | `__V_DBT_W_P`                        |
| FR-15       | Ch.1 Eq. 22, 24         | `__RH_DBT_W_P`                       |
| FR-16       | Ch.1 Eq. 39             | `__DPT_Pw`                           |
| FR-17       | Ch.1 Eq. 35             | `__WBT_DBT_W_P`                      |
| FR-18       | Bisection (iterative)   | `__DBT_*` family of functions        |
| FR-21       | Ch.1 Eq. 22             | `__Pw_W_P`                           |
