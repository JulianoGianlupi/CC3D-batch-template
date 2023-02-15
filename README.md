# CC3D-batch-template
This repo implements the batch execution method for CompuCell3D created by Sego et al. in [1].

## About

This package was built to make the batch execution of CompuCell3D packages easier. It has been set up for the 
slurm queueing system, but can easily be modified to other queueing systems.

## Set up

NOTE: I'll assume you are using conda to manage your packages and install CC3D

1. You need to set two environment variables in `batch_exec.sh`, namely: `PREFIX_CC3D` and `PYTHON_INSTALL_PATH`
   1. `PREFIX_CC3D` is the root directory of your CC3D installation, _e.g._, `/.../<conda>/<CC3D_env>/bin`
   2. `PYTHON_INSTALL_PATH` is the path to the Python root directory, _e.g._, `/.../<conda>/<CC3D_env>/bin`
2. The file `batch_exec.py` controls where the output will be (`sweep_output_folder`)
3. Set he input files (`input_modules`)
4. It will set the parameter variation using multipliers (`mult_dict`)
5. Set the number of replicates (`num_rep`)
6. Set slurm configurations (`config_template`)

## How to use

Your model should be place in the `Models` folder, _e.g._, `Models/MyModel/`. `Models/MyModel/`
is the folder containing the `.cc3d` file and `Simulation` folder. You should put an `__init__.py` file in all
folders Your model inputs Have to be placed in a parameter-only file in  `Models/MyModel/Simulation`, _e.g._, 
`Models/MyModel/Simulation/MyInputs.py`. The structure of the inputs is very specific, it should be:

```python
# Info for documentation of parameter values goes here, in *__param_desc__*
# The parameter name and assigned description can be quickly exported to csv along with specified values using
# *export_parameters*, in export_parameters.py
# To use, assign the name of the code variable as a key, and a string description as its value
# e.g.,     __param_desc__['my_var'] = 'My variable'
#           my_var = 1
# The generated csv will read, "my_var, 1, My variable'
__param_desc__ = {}
__param_desc__["my_parameter0"] = "(Optional) Description for my_parameter0"
my_parameter0 = 123
```
The keys of `__param_desc__` **must absolutely match** the variable names, and there **must** be a key for every
parameter in `__param_desc__`. Your simulation will then have to import
the inputs file. The `batch_exec.py` file will modify the values of the inputs using multipliers 
defined in `mult_dict`, therefore the 
input file should be imported in `batch_exec.py`, and registered there with `BatchRunLib.register_auto_inputs`. 

The dictionary `config_template` sets the slurm configuration via keys, they are:
* `jn`: job name
* `wh`: Number of hours of wall time requested (total time wh:wm)
* `wm`: Number of minutes of wall time requested (total time wh:wm)
* `ppn`: Number of tasks per node
* `vmem`: Virtual memory request in GB

To actually apply the parameter multiplication the `BatchRun` package must be loaded in the steppable 
file, and `BatchRun.BatchRunLib.apply_external_multipliers` applied to the input file _in the 1st steppable_ 
that is registered in the main python file. _E.g._:

```python
# ...
# Import project libraries and classes
sys.path.append(os.path.dirname(os.path.dirname(os.path.dirname(os.path.dirname(__file__)))))
# Import project libraries and classes
sys.path.append(os.path.dirname(__file__))
from Simulation.ViralInfectionVTMSteppableBasePy import *
import ViralInfectionVTMLib
# from ViralInfectionVTMModelInputs import *
from BatchRun import BatchRunLib

# Import toolkit
sys.path.append(os.path.dirname(os.path.dirname(__file__)))
from nCoVToolkit import nCoVUtils
from ModelInputs import *

# ...

class Steppable0(ViralInfectionVTMSteppableBasePy):

    def __init__(self, frequency=1):

        ViralInfectionVTMSteppableBasePy.__init__(self, frequency)
        import ModelInputs
        BatchRunLib.apply_external_multipliers(__name__, ModelInputs)
# ...
```

Note that we are using `ViralInfectionVTMSteppableBasePy` as the parent class of the steppable, instead of the usual
`SteppableBasePy`.



## References
1. Sego, T. J., Josua O. Aponte-Serrano, Juliano Ferrari Gianlupi, Samuel R. Heaps, Kira Breithaupt, Lutz Brusch, 
Jessica Crawshaw et al. "A modular framework for multiscale, multicellular, spatiotemporal modeling of acute 
primary viral infection and immune response in epithelial tissues and its application to drug therapy timing and 
effectiveness." PLoS computational biology 16, no. 12 (2020): e1008451.
