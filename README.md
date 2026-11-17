# ZERAF research extensions

**Modelling adaptive opaque façades with a modified version of EnergyPlus 23.2.0.**

This repository contains the modified simulation source code accompanying our paper on the thermal behaviour of the ZERAF adaptive opaque façade. The extensions address heat-transfer processes that are difficult to represent with standard exterior boundary conditions, including deployable cladding, partially open louvres, and semi-transparent rolling textiles.

> **Related publication**  
> Mohammad Ali Qorbani, Francesco Isaia, Miren Juaristi & Roel Loonen (2026). *Capturing dynamic thermal behaviour of adaptive opaque façades in building performance simulation: a case study of Zeraf*. Journal of Building Performance Simulation.  
> [Read the open-access paper](https://doi.org/10.1080/19401493.2026.2758619)

This is an independent research modification, not an official EnergyPlus release. The base version is **23.2.0**, as recorded in [`cmake/Version.cmake`](cmake/Version.cmake). We describe the experimental validation, model assumptions, and scope of application in the paper.

## What is modelled?

ZERAF combines two complementary façade systems:

- **Active Insulation System (AIS):** a fan-driven, closed air loop transfers heat between channels on either side of the insulation, increasing heat exchange when beneficial. Following the modelling principle introduced by [Koenders et al. (2018)](https://doi.org/10.1016/j.enbuild.2018.05.051), AIS is represented in the paper using internal heat source constructions and a coupling zone.
- **Kinetic Cladding (KC):** vertical louvres, horizontal louvres, or a rolling textile change the solar exposure and thermal boundary conditions of the underlying façade.

### Main modelling features

| Capability | Implementation |
| --- | --- |
| Switchable exterior boundary conditions | Switch between a deployed exterior cavity and an exposed-surface treatment at runtime through an EMS/Python-API actuator. |
| Dynamic radiative exchange | When the louvres partly expose the underlying surface, update its view factors to the sky and ground at runtime through EMS/Python-API actuators. The remaining view factor accounts for exchange with the cladding. |
| Finite cladding thermal resistance | Set the louvre thermal transmittance $U_{\mathrm{baffle}} = k/L$ through the `Cladding Thermal Transmittance` EMS actuator. This activates the model with separate temperatures for the inner and outer cladding faces. |
| Rolling-textile transmission | Use the `Rolling Textile Translucency` EMS actuator to set the fraction of solar radiation transmitted through the textile (0 to 1). |

### Experimental overrides

We added these overrides to prescribe boundary conditions for some experiments. In routine simulations, leave them unset so EnergyPlus calculates these quantities.

| Override | Purpose in the study |
| --- | --- |
| Solar gains behind and on the cladding | Prescribe the absorbed solar flux on the underlying surface and on the cladding. |
| Boundary condition temperatures | Prescribe selected temperatures, such as the underlying surface, baffle, and cavity air temperatures, for comparison with measurements. |

AIS geometry, fan operation, cladding geometry, and supervisory control still need to be defined in simulation inputs and any accompanying control routines. Compiling this source alone does not create a complete ZERAF model.

## Start here

1. Read the paper for the model formulation, assumptions, calibration, and validation cases.
2. Build the complete source tree using the instructions below.
3. Use the EMS actuators to supply the rolling textile's solar transmission fraction or the louvre thermal transmittance $U_{\mathrm{baffle}}$ at the start of the simulation. Use EMS or the Python API to switch the exterior boundary conditions and update view factors at runtime. See the [actuator interface notes](docs/ZERAF_INTERFACE.md).

The two startup parameters are currently available only through EMS actuators, which can also be set through the Python API. We plan to add them to the IDD so they can be set directly in the IDF.

## Companion data and simulation inputs

We provide the experimental data and EnergyPlus input files (IDFs) through the companion [Zenodo archive cited in the paper](https://doi.org/10.5281/zenodo.22306700). **The archive files are currently under embargo and will become publicly accessible once the embargo ends.** Please use this persistent link to access them when released.

This repository provides the modified simulation source; the companion archive provides the study data and IDFs. The inherited EnergyPlus examples and tests included here are separate from the paper's validation cases.

## Source guide

The principal implementation locations are listed below.

| File | Role |
| --- | --- |
| [`HeatBalanceSurfaceManager.cc`](src/EnergyPlus/HeatBalanceSurfaceManager.cc) | Boundary-condition switching, modified exterior heat balances, and selection of the baffle model. |
| [`HeatBalanceSurfaceManager.hh`](src/EnergyPlus/HeatBalanceSurfaceManager.hh) | Corresponding function declarations, including the KC argument. |
| [`TranspiredCollector.cc`](src/EnergyPlus/TranspiredCollector.cc) | Extended passive baffle/cavity calculations, including the two-temperature baffle model. |
| [`TranspiredCollector.hh`](src/EnergyPlus/TranspiredCollector.hh) | Declarations of the original and extended baffle routines. |
| [`SurfaceGeometry.cc`](src/EnergyPlus/SurfaceGeometry.cc) | Registration of the custom EMS actuators. |
| [`DataSurfaces.hh`](src/EnergyPlus/DataSurfaces.hh) | Actuator values and override flags associated with the other-side conditions model. |
| [`DataHeatBalance.hh`](src/EnergyPlus/DataHeatBalance.hh) | Additional exterior-baffle temperature state. |

## Building on Windows

The following commands build the source with **Visual Studio 2022, x64**. Install the C++ desktop development tools before configuring the project.

Requirements:

- Git and CMake. The project declares CMake 3.17 as its minimum; use a compatible CMake 3.x installation for this historical source tree.
- A C++17 compiler; the commands below select Visual Studio 2022.
- A matching 64-bit Python installation with headers and import library when `LINK_WITH_PYTHON=ON`.

From the repository root, in PowerShell:

```powershell
cmake -S . -B build -G "Visual Studio 17 2022" -A x64 -DLINK_WITH_PYTHON=ON -DBUILD_TESTING=OFF -DDOCUMENTATION_BUILD=DoNotBuild
cmake --build build --config Release --target energyplus --parallel 4
```

If Python discovery selects the wrong installation, add `-DPython_ROOT_DIR="C:/path/to/Python"` to the configure command, replacing the example path with the installation you intend to use. Record the actual Python, compiler, and CMake versions with your release.

The project places runtime products under `build/Products`; a multi-configuration generator may use a `Release` subdirectory. To locate the executable:

```powershell
Get-ChildItem build/Products -Recurse -Filter energyplus.exe
```

Use this build's executable, matching IDD, runtime libraries, and generated `pyenergyplus` package together. Python/API users should ensure that imports resolve to this build rather than a separate standard installation. Compiling a modified executable does not update an installed copy automatically.

For broader build guidance, see the [upstream build documentation](https://github.com/NREL/EnergyPlus/wiki/Building-EnergyPlus). Instructions for current upstream releases may differ from the requirements of version 23.2.0.

## Citation

If you use these models or extensions in research, please cite the associated paper and identify the exact source commit or release used. Citation is requested for scientific attribution; it is not an additional software-license condition.

```bibtex
@article{Qorbani2026ZERAF,
  author  = {Qorbani, Mohammad Ali and Isaia, Francesco and Juaristi, Miren and Loonen, Roel},
  title   = {Capturing dynamic thermal behaviour of adaptive opaque fa\c{c}ades in building performance simulation: a case study of {Zeraf}},
  journal = {Journal of Building Performance Simulation},
  year    = {2026},
  doi     = {10.1080/19401493.2026.2758619},
  url     = {https://doi.org/10.1080/19401493.2026.2758619}
}
```

Machine-readable citation information is provided in [`CITATION.cff`](CITATION.cff).

## Funding acknowledgment

This work was supported by the European Union's Horizon Europe research and innovation programme under grant agreement **No. 101098490 (ZERAF)**.

## License

The EnergyPlus source code and the ZERAF modifications in this repository are distributed under the terms in [`LICENSE.txt`](LICENSE.txt). This is an independent research modification; no endorsement by the EnergyPlus developers or the U.S. Department of Energy is implied.

## Contact

Research contact: **Mohammad Ali Qorbani**, Department of the Built Environment, Eindhoven University of Technology — [m.a.qorbani@tue.nl](mailto:m.a.qorbani@tue.nl).

For questions about this implementation, open a repository issue and include the commit, build environment, a minimal input case, and relevant error output. For the experimental work and collaborating institutions, see the linked paper.
