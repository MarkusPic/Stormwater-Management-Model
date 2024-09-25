# Stormwater-Management-Model

Stormwater Management Model (aka "SWMM") solver only

## Changes in this repository

- added the hydraulic radius to the variables for links in the output file
- added the shear stress stats (currently only maximum) as a summary (`Conduit Shear Stress Summary`) table to the report file

# Reading the out-file with python using the swmm-api package

Calculating the shear stress in the conduits.

$$\tau = \rho * g * r_{hydr} * S_{o}$$
with:
- $\tau$ ... shear stress in $N/m^2$
- $\rho$ ... density of water = 1000 $kg/m^3$
- $g$ ... gravity = 9.81 $m/s^2$
- $r_{hydr}$ ... hydraulic radius in $m$
- $S_o$ ... conduit bed slope in $m/m$

```python3
from numpy import nan

from swmm_api import CONFIG, SwmmInput, SwmmReport
from swmm_api.input_file.macros import conduit_slopes
from swmm_api.output_file.definitions import LINK_VARIABLES, OBJECTS, VARIABLES
from swmm_api.run_swmm import swmm5_run_epa
from swmm_api.run_swmm.run_temporary import swmm5_run_temporary
from swmm_api.report_file.helpers import _part_to_frame

CONFIG['exe_path'] = '/Users/markus/CLionProjects/Stormwater-Management-Model/build/bin/runswmm'

inp = SwmmInput('./epaswmm5_apps_manual/Example6-Final.inp')

LINK_VARIABLES.HYDRAULIC_RADIUS = 'hydraulic radius'  # in meters
LINK_VARIABLES.LIST_ += [LINK_VARIABLES.HYDRAULIC_RADIUS]

# ---
def _conduit_shear_stress_summary(self: SwmmReport):
    if self._conduit_surcharge_summary is None:
        p = self._get_converted_part('Conduit Shear Stress Summary')
        self._conduit_surcharge_summary = _part_to_frame(p)
    return self._conduit_surcharge_summary

SwmmReport.conduit_shear_stress_summary = _conduit_shear_stress_summary

# ---
with swmm5_run_temporary(inp, run=swmm5_run_epa, label='example_run_swmm') as res:
    hydraulic_radius = res.out.get_part(OBJECTS.LINK, None, VARIABLES.LINK.HYDRAULIC_RADIUS).replace(-999., nan)
    slopes = conduit_slopes(inp)
    shear_stress = 1000*9.81*hydraulic_radius*slopes
    print()

    shear_stress_max = res.rpt.conduit_shear_stress_summary
```

Example for new Report-file section

```swwmm.rpt
  
  ****************************
  Conduit Shear Stress Summary
  ****************************
  
  ---------------------------------
                            Maximum
                       shear stress
  Conduit                      N/m2
  ---------------------------------
  C1                           1.42
```