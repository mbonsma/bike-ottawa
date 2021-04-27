# Ottawa Bicycle Counts

A python analysis of the [City of Ottawa bicycle trip counter data](https://open.ottawa.ca/datasets/bicycle-trip-counters).

The metadata for the bicycle trip counters is [here](https://www.arcgis.com/home/item.html?id=f218592c7fe74788906cc6a0eb190af9).

## Installation
### How to run this code 

#### Fork and clone bike-ottawa
1. Create a GitHub account if you don't already have one.
2. Go to https://github.com/mbonsma/bike-ottawa and click 'Fork' in the top right corner.
3. In the terminal, type `git clone https://github.com/YourUserName/bike-ottawa.git` (with your GitHub username)
4. In the terminal type `cd bike-ottawa` to enter the bike-ottawa directory
5. Add reference to upstream repository: `git remote add upstream https://github.com/mbonsma/bike-ottawa.git`

Now you have a fork and a local clone of this repository that you can use and modify however you like!
You can stay synced with the upstream repo by periodically pulling any changes like this:

```
cd bike-ottawa
git checkout main
git pull upstream main
```

#### Install Python

1. Install [Python 3 via Anaconda](https://www.anaconda.com/products/individual) (my recommended Python distribution).

<!-- #### Create conda environment
1. Install Python 3 via Anaconda.
2. `cd bike-ottawa`
3. `conda env create -f environment.yml`. If any packages don't install, try installing them in the environment with `conda install -c conda-forge package-name` after running `source activate bikeottawa`. 
4. **Important:** in order for the conda environments to show up automatically in Jupyter, run this command *without activating the bikeottawa environment* (i.e. in your base environment): `conda install nb_conda_kernels`
4. To start up the bikeottawa environment: `conda activate bikeottawa`. Now your terminal session is running the bikeottawa conda environment.-->

#### Run Jupyter Notebooks

To run the [Jupyter notebook code](https://github.com/mbonsma/bike-ottawa/blob/main/bike_counts.ipynb), follow these steps.

1. `cd bike-ottawa`
<!--2. `source activate bikeottawa`-->
2. `jupyter lab` - this launches Jupyter Lab in your browser.
You should see the `bike_counts.ipynb` file; double-click to open it.

#### Debugging the notebooks

1. `ModuleNotFoundError`

If you see `ModuleNotFoundError: No module named 'seaborn'` or some other package name in place of `seaborn`,
this means that a python package is missing.
Run `conda install package-name` to install it. 

<!--If you see `ModuleNotFoundError: No module named 'seaborn'` or some other package name in place of `seaborn`,
this probably means that the Jupyter notebook is not running the `bikeottawa` environment or that a 
python package is missing. The environment might not be running for 
two common reasons: either you forgot to `source activate bikeottawa`
before running `jupyter notebook`, or Jupyter can't find the right kernel because
`conda install nb_conda_kernels` hasn't been run. You can also try changing the kernel by going to
the Kernels menu, clicking 'Change kernel', and looking for `bikeottawa`.
If a python package is missing, run `conda install package-name` in the `bikeottawa`
environment.-->
