# Stormwater-Management-Model

Stormwater Management Model (aka "SWMM") solver only

## Changes in this repository

- added the hydraulic radius to the variables for links in the output file

# Reading the out-file with python using the swmm-api package

Calculating the shear stress in the conduits.

$$\tau = \rho * g * r_{hydr} * S_{o}$$
with:
- $\tau$ ... shear stress in $N/m^2$
- $\rho$ ... density of water = 1000 $kg/m^3$
- $g$ ... gravity = 9.81 $m/s^2$
- $r_{hydr}$ ... hydraulic radius in $m$
- $S_o$ ... conduit bed slope in $m/m$

```python
from numpy import nan

from swmm_api import CONFIG, SwmmInput
from swmm_api.input_file.macros import conduit_slopes
from swmm_api.output_file.definitions import LINK_VARIABLES, OBJECTS, VARIABLES
from swmm_api.run_swmm import swmm5_run_epa
from swmm_api.run_swmm.run_temporary import swmm5_run_temporary

CONFIG['exe_path'] = '/Users/markus/CLionProjects/Stormwater-Management-Model/build/bin/runswmm'

inp = SwmmInput('./epaswmm5_apps_manual/Example6-Final.inp')

LINK_VARIABLES.HYDRAULIC_RADIUS = 'hydraulic radius'  # in meters
LINK_VARIABLES.LIST_ += [LINK_VARIABLES.HYDRAULIC_RADIUS]

with swmm5_run_temporary(inp, run=swmm5_run_epa, label='example_run_swmm') as res:
    hydraulic_radius = res.out.get_part(OBJECTS.LINK, None, VARIABLES.LINK.HYDRAULIC_RADIUS).replace(-999., nan)
    slopes = conduit_slopes(inp)
    shear_stress = 1000*9.81*hydraulic_radius*slopes
    print()
```
