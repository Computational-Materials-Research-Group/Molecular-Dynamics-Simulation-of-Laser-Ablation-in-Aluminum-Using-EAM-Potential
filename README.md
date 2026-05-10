# Molecular Dynamics Simulation of Laser Ablation in Aluminum Using EAM Potential

<p align="center">
  <img src="https://img.shields.io/badge/LAMMPS-MD%20Simulation-blue?style=for-the-badge&logo=gnu&logoColor=white"/>
  <img src="https://img.shields.io/badge/Aluminum-EAM%20Potential-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Laser-Ablation-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Thermal%20Spike-Model-purple?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/OVITO-Visualization-green?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/License-CC%20BY--NC%204.0-lightgrey?style=for-the-badge"/>
</p>

<p align="center">
  A fully atomistic <b>molecular dynamics simulation of laser ablation cutting</b>
  of an aluminum thin sheet using LAMMPS. Models a thin FCC Al square sheet subjected
  to a focused laser beam that deposits energy into a cylindrical zone, driving rapid
  heating, melting, and material removal via the <i>EAM/alloy potential (Al99.eam.alloy)</i>.
  Temperature field evolution, Heat Affected Zone (HAZ), and post-cut relaxation are
  captured across all simulation phases.
</p>

<img width="1600" height="1200" alt="ablation2" src="https://github.com/user-attachments/assets/4aa91cad-73a9-434d-9469-8c6ec19534fe" />


---

## Physics

The simulation captures the full sequence of laser ablation cutting at the atomistic scale:

- FCC aluminum thin sheet modeled with the EAM/alloy interatomic potential
- Laser energy deposition modeled via the thermal spike approach: a cylindrical zone is rapidly heated to extreme temperature
- NVT (Nose-Hoover) thermostat applied separately to the laser zone and bulk sheet to simulate differential heating
- Material removal via `delete_atoms` on the laser-irradiated cylinder after ablation phase
- Post-cut quenching and relaxation of the remaining sheet back to ambient temperature
- Temperature gradient field visualized in OVITO: blue (cool bulk) → red (hot HAZ) color map

---

## Geometry

```
  Simulation box (units: Angstrom):

  Top view (XY plane):

  +--------------------------------------------------+   y = 150
  |                                                  |
  |              Bulk Al Sheet                       |
  |                                                  |
  |           +~~~~~~~~~~~~~~~~~~~+                  |
  |           ~  Laser Cut Zone   ~  <- cylinder z   |
  |           ~  center (75,75)   ~     radius 40    |
  |           ~  r = 40 Ang       ~                  |
  |           +~~~~~~~~~~~~~~~~~~~+                  |
  |                                                  |
  +--------------------------------------------------+   y = 0
  x = 0                                         x = 150

  Side view (XZ plane):

  z = 20  +------------------------------------------+  <- sheet top
          |         [  Laser Cylinder Zone  ]        |
          |          center x=75, y=75, r=40         |
          |          full Z thickness cut            |
  z =  0  +------------------------------------------+  <- sheet bottom

  Box XY: 0 to 150 Angstrom (periodic in x and y)
  Box Z:  0 to 20 Angstrom
```

- **Sheet**: FCC Al slab, 150 × 150 × 20 Å; fully periodic in x and y
- **Laser Cut Zone**: Cylinder along z-axis, center (75, 75), radius = 40 Å, full sheet thickness
- **Bulk Sheet**: All atoms outside the cylinder region
- **Lattice**: FCC Al, lattice constant = 4.05 Å

---

## Simulation Phases

| Phase | Run Steps | Simulated Time | Description |
|-------|-----------|----------------|-------------|
| Equilibration | 5,000 | 5 ps | Full sheet thermalization at 300 K |
| Laser Heating Ramp | 3,000 | 3 ps | Laser zone heated from 300 K → 10,000 K |
| Ablation Hold | 5,000 | 5 ps | Laser zone held at 10,000 K (melting/ablation) |
| Atom Deletion | — | — | Material removal — laser cut complete |
| Quench | 5,000 | 5 ps | Remaining sheet cooled from 10,000 K → 300 K |
| Final Relaxation | 5,000 | 5 ps | Sheet equilibration at 300 K |
| **Total** | **23,000** | **23 ps** | — |

---

## Material Parameters

### Aluminum Properties

| Parameter | Symbol | Value | Unit |
|-----------|--------|-------|------|
| Crystal structure | — | FCC | — |
| Lattice constant | a | 4.05 | Angstrom |
| Atomic mass | m | 26.982 | g/mol |
| Interatomic potential | — | EAM/alloy | Al99.eam.alloy |

### Laser Parameters

| Parameter | Symbol | Value | Unit |
|-----------|--------|-------|------|
| Laser zone geometry | — | Cylinder (z-axis) | — |
| Cylinder radius | r | 40 | Angstrom |
| Cylinder center | (cx, cy) | (75, 75) | Angstrom |
| Peak laser temperature | T_laser | 10,000 | K |
| Initial sheet temperature | T_init | 300 | K |

### Thermostat Settings

| Group | Fix | Temperature | Damping |
|-------|-----|-------------|---------|
| Full sheet (Stage 1) | NVT | 300 K | 0.1 ps |
| Bulk sheet (Stage 2–3) | NVT | 300 K | 0.1 ps |
| Laser zone (Stage 2) | NVT ramp | 300 → 10,000 K | 0.1 ps |
| Laser zone (Stage 3) | NVT hold | 10,000 K | 0.1 ps |
| Remaining sheet (Stage 5) | NVT quench | 10,000 → 300 K | 0.1 ps |
| Remaining sheet (Stage 6) | NVT | 300 K | 0.1 ps |

---

## Governing Equations

### Thermal Spike Model

```
T_laser(t) = T_init + (T_peak - T_init) * (t / t_ramp)

where:
  T_init   = initial temperature (300 K)
  T_peak   = peak laser temperature (10,000 K)
  t_ramp   = ramp duration (3 ps)
```

### EAM/Alloy Potential Energy

```
E_i = F_alpha(sum_{j != i} rho_alpha_beta(r_ij)) + 0.5 * sum_{j != i} phi_alpha_beta(r_ij)

where:
  F_alpha    = embedding energy function
  rho        = electron density contribution
  phi        = pair interaction potential
```

### Heat Affected Zone (HAZ) Radius

```
r_HAZ > r_laser

The HAZ extends beyond the cylinder boundary due to thermal conduction.
Atoms in the HAZ are not deleted but show elevated temperature and
structural disorder in OVITO visualization.
```

---

## Group Definitions

```
sheet          <- Full FCC Al slab (all atoms)
laser_zone     <- Cylinder z, center (75,75), radius 40, z: 0 to 20
bulk_sheet     <- All atoms NOT in laser_zone (subtract all laser_zone)
remaining_sheet <- All atoms after delete_atoms (type 1, post-deletion)
```

---

## Output Files

| File | Dump Frequency | Content |
|------|---------------|---------|
| `dump_laser_cut.lammpstrj` | every 500 steps | id, type, x, y, z, vx, vy, vz (all phases) |
| `final_sheet_after_laser_cut.data` | — | Final atomic positions and velocities (LAMMPS data format) |
| `final_sheet.restart` | — | Binary restart file for continuation runs |

---

## Repository Structure

```
laser_ablation_md/
|
|-- laser_cut_circle_Al.lammps          # Main LAMMPS simulation script
|-- Al99.eam.alloy                      # EAM potential file (required)
|-- README.md                           # This file
|
|-- outputs/                            # Generated on run
    |-- dump_laser_cut.lammpstrj        # Full trajectory (all phases)
    |-- final_sheet_after_laser_cut.data  # Final atomic configuration
    |-- final_sheet.restart             # Restart file
```

---

## How to Run

### Requirements

- LAMMPS (any recent version with EAM package): https://www.lammps.org
- EAM potential file `Al99.eam.alloy` (available from NIST IPRP or LAMMPS potentials directory)
- OVITO or VMD for trajectory visualization: https://www.ovito.org

### Step 1 — Obtain the EAM potential file

```bash
# Place Al99.eam.alloy in the same directory as the script
ls Al99.eam.alloy
```

### Step 2 — Run the simulation

```bash
lmp -in laser_cut_circle_Al.lammps
```

Or in parallel with MPI:

```bash
mpirun -np 4 lmp -in laser_cut_circle_Al.lammps
```

The script will:
1. Build the FCC Al thin sheet (150 × 150 × 20 Å)
2. Assign initial velocities at 300 K
3. Equilibrate the full sheet
4. Ramp and hold the laser cylinder zone to 10,000 K
5. Delete atoms inside the cylinder (material removal)
6. Quench and relax the remaining sheet back to 300 K
7. Write final data and restart files

### Step 3 — Visualize in OVITO

1. Open OVITO > `File > Load File` > select `dump_laser_cut.lammpstrj`
2. Add modifier: **Color Coding** → set property to `Temperature` (or `v` for velocity magnitude)
3. Step through frames to observe laser heating and hole formation
4. Add modifier: **Slice** to view cross-section of the cut zone
5. Use **Common Neighbor Analysis (CNA)** to identify disordered atoms in the HAZ

---

## What to Look for in Results

### Temperature Field

During the heating phase, the laser cylinder zone rapidly reaches 10,000 K (red in OVITO). A radial temperature gradient is visible: red/orange at the cut edge, yellow/green in the HAZ, and blue in the unaffected bulk sheet.

### Heat Affected Zone (HAZ)

Atoms immediately surrounding the cut cylinder show elevated temperature and structural disorder. The HAZ radius extends beyond the cylinder boundary due to thermal conduction through the FCC lattice.

### Material Removal

After `delete_atoms`, a clean circular hole through the full sheet thickness is created. The cylinder geometry ensures a geometrically perfect circular cross-section when viewed along the z-axis in OVITO.

### Post-Cut Relaxation

During quenching, atoms at the hole edge rearrange due to surface tension and dangling bond effects. The HAZ gradually returns to near-ambient temperature, and the FCC structure partially recovers in the bulk.

---

## Common Errors and Fixes

| Error | Cause | Fix |
|-------|-------|-----|
| `Cannot open potential file Al99.eam.alloy` | Potential file not found | Place `Al99.eam.alloy` in the run directory |
| `Lost atoms` during heating | Atoms escape box at high temperature | Add `thermo_modify lost warn` or extend box |
| Hole appears oval not circular | Used `sphere` region on thin sheet | Use `cylinder z` region (already in this script) |
| Hole closes after relaxation | Periodic BC + surface tension pulls atoms inward | Use `boundary s s p` or freeze edge atoms |
| High initial energy | Atoms placed on lattice sites with overlap | Run `minimize` before dynamics if needed |
| Group empty after delete_atoms | Group defined before deletion no longer valid | Redefine group with `group remaining_sheet type 1` after deletion |
| Temperature spike at start | Velocities not equilibrated | Ensure Stage 1 equilibration runs fully before laser heating |

---

## Extending the Model

| Extension | What to Change |
|-----------|----------------|
| Different hole shape | Replace `cylinder z` with `ellipsoid` or `prism` region |
| Multiple holes | Define `cut2`, `cut3` cylinder regions; repeat heating and delete steps |
| Larger sheet | Increase `lx`, `ly` variables; adjust `cx`, `cy` accordingly |
| Thicker sheet | Increase `lz`; update cylinder z bounds to match |
| Different material (Cu, Ni) | Change `mass`, `pair_coeff`, lattice constant, and potential file |
| Prevent hole closure | Add `fix freeze boundary setforce 0.0 0.0 0.0` on edge atoms |
| Higher laser energy | Increase `T_laser` variable beyond 10,000 K |
| Radial distribution function | Add `compute rdf all rdf 100` and `fix ave/time` after relaxation |
| Two-temperature model (TTM) | Replace NVT ramp with LAMMPS `fix ttm` for electron-phonon coupling |

---

## Citation

If you use this code in your research, please cite:

```bibtex
@software{mishra_2026_laserablation,
  author    = {Mishra, A.},
  title     = {Molecular Dynamics Simulation of Laser Ablation in Aluminum Using EAM Potential},
  year      = {2026},
  publisher = {Zenodo},
  doi       = {10.5281/zenodo.20111185},
  url       = {https://doi.org/10.5281/zenodo.20111185}
}
```

Plain text citation:

> Mishra, A. (2026). *Molecular Dynamics Simulation of Laser Ablation in Aluminum Using EAM Potential*. Zenodo. https://doi.org/10.5281/zenodo.20111185

---

## Author

**akshansh11**
GitHub: https://github.com/akshansh11

---

## License

<p>
<a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/">
<img alt="Creative Commons Licence" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/88x31.png"/>
</a>
<br/>
This work is licensed under a
<a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/">Creative Commons Attribution-NonCommercial 4.0 International License</a>.
</p>

You are free to:

- **Share** — copy and redistribute the material in any medium or format
- **Adapt** — remix, transform, and build upon the material

Under the following terms:

- **Attribution** — You must give appropriate credit to akshansh11 and provide a link to this repository
- **NonCommercial** — You may not use the material for commercial purposes

Copyright 2026 akshansh11. All rights reserved for commercial use.
