# Poetry + Conda
Use Conda for managing the environment and conda-only packages, while use Poetry for managing packages

Inspired by [https://stackoverflow.com/a/71110028](https://stackoverflow.com/a/71110028)

More readings:
* [Creating an environment from an environment.yml file](https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#creating-an-environment-from-an-environment-yml-file)
* [Why `conda-lock`](https://github.com/conda-incubator/conda-lock#why)
* [`conda update --prune` feature is broken since conda 4.4](https://github.com/conda/conda/issues/7279#issuecomment-516438069)

## Prerequisites
Make sure `mamba`, `conda-lock`, and `poetry` are available. 
* `conda-lock` is to create conda env lock file
* `mamba` which works well with conda and is faster and does better at resolving dependency conflicts.
   Most importantly, `mamba update` works as expected unlike `conda update --prune` since conda 4.4 (TODO: verify)

You can create a conda environment with these packages and remove it later
``` cmd
# create 
conda create -n bootstrap -c conda-forge mamba conda-lock poetry>=1
# remove 
conda env remove -n bootstrap
```

## Create or update .lock files for reproducing an environment
1. Create a new `environment.yml` or update the content of an existing one. 
 
2. Create conda .lock file(s) for the conda env using `conda-lock`
   ``` cmd 
   # multi-platform lockfile (conda-lock.yml)
   conda-lock -f environment.yml  
   
   # or platform(s) specific
   conda-lock -f environment.yml -p win-64 -p osx-64 -p linux-64
   ```
   
3. Set up poetry and generate `poetry.lock` file
   ``` cmd
   # Only needed in the first-time setup of the project to generate the pyproject.toml file
   poetry init --python=~3.10  # must use the same python version as in environment.yml
   
   # Then use "poetry add --lock <package=version>" to add/update packages installed via conda (aka the environment.yml)
   # This will only update the poetry lockfile (won't install package). For example.
   # poetry add --lock tensorflow=2.8.0 torch=1.11.0 torchaudio=0.11.0 torchvision=0.12.0
   
   # Install packages using poetry
   # poetry install ...
   
   # Generate `poetry.lock` file
   poetry lock
   ```
   
## Reproduce a conda environment using the .lock files
### Create a conda environment from scratch
Firstly copy the latest `conda-lock.yml` and `poetry.lock` files, then
``` cmd
conda-lock create --name my_proj_env --file conda-lock.yml
conda activate my_proj_env
poetry install  # this will read the poetry.lock file
```

### Update a conda environment
Firstly copy the latest `conda-lock.yml` and `poetry.lock` files, then
``` cmd
conda activate my_proj_env
mamba env update --file conda-lock.yml
poetry install  # this will read the poetry.lock file
```
