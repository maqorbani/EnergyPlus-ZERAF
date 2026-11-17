# ZERAF actuator interface

These names are taken from the registrations in `src/EnergyPlus/SurfaceGeometry.cc`. They are available in this modified build; a standard EnergyPlus 23.2.0 installation does not provide these custom actuators.

## Component type and key

For all custom actuators below:

- Component type: `SurfaceProperty:ExteriorNaturallyVentedCavity`
- Unique key: the **name of the associated `SurfaceProperty:OtherSideConditionsModel` object**, as stored in `OSCM.Name`.

The key is not the surface name or necessarily the exterior-cavity object's name. Set up the surface, other-side conditions model, and exterior naturally vented cavity consistently in the IDF.

| Control type (exact spelling) | Value and interpretation |
| --- | --- |
| `Kinetic Cladding Control` | `0` selects the retracted/exposed path; `1` selects the deployed/cavity path. Without an override, the custom cavity handling defaults to deployed. The implementation treats any nonzero override as deployed; use only `0` and `1`. |
| `Underlying Surface Temperature` | Prescribed underlying exterior-surface temperature, °C. |
| `Underlying Surface Absorbed Solar Irradiation` | Absorbed solar flux on the underlying surface, W/m². Supply absorbed flux, not incident irradiance. |
| `Underlying Surface Sky Exposure Factor` | Dimensionless sky exposure factor. |
| `Underlying Surface Ground Exposure Factor` | Dimensionless ground exposure factor. |
| `Cladding Surface Absorbed Solar Irradiation` | Absorbed solar flux on the cladding, W/m². |
| `Baffle Surface Temperature` | Baffle temperature override, °C; check its use in the selected baffle-model overload. |
| `Cavity Air Temperature` | Cavity air temperature override, °C. |
| `Cavity Air Velocity` | Cavity air velocity override, m/s. |
| `Cladding Thermal Transmittance` | Cladding material conductance `k/L`, W/(m²·K). Activating this override selects the two-temperature baffle model; it is not the whole-façade U-value. |
| `Rolling Textile Translucency` | Solar transmission fraction, **0–1**. Although the registration labels the units `[%]`, the equations use the value directly as a fraction: use `0.10` for 10%, not `10`. |

Set the sky and ground exposure overrides together. Supply physically consistent values between 0 and 1 with a sum no greater than 1: the implementation calculates the remaining cladding exposure as `1 - (sky + ground)`. Do not rely on the engine to validate these bounds.

The radiation geometry and any time-varying absorbed-flux calculations must come from the model/controller or a separate precomputation. Setting KC alone does not calculate louvre geometry or generate those inputs. The one-node textile path and two-temperature louvre path have different inputs; do not assume all override combinations are meaningful.

## EMS example

This illustrates the actuator declaration only. Replace `My OSCM` with the actual other-side conditions model name and supply an EMS program and calling manager appropriate to the model:

```idf
EnergyManagementSystem:Actuator,
    ZERAF_KC_State,
    My OSCM,
    SurfaceProperty:ExteriorNaturallyVentedCavity,
    Kinetic Cladding Control;
```

## Python API example

In a callback where API data are ready, using the `api` instance and `state` for the current simulation:

```python
if api.exchange.api_data_fully_ready(state):
    handle = api.exchange.get_actuator_handle(
        state,
        "SurfaceProperty:ExteriorNaturallyVentedCavity",
        "Kinetic Cladding Control",
        "My OSCM",
    )
    if handle == -1:
        raise RuntimeError("ZERAF KC actuator not found; check the build and OSCM name")
    api.exchange.set_actuator_value(state, handle, 1.0)
```

This is an interface illustration, not a complete controller or validated callback schedule. Acquire/cache handles separately for each simulation state, set values before the relevant heat balance is evaluated, and check timestep and warmup behaviour against the intended experiment. The paper's supervisory control and AIS fan control require additional implementation.
