# s2p3 Downloading CMIP model data to drive downscaling model, and creating forcing files

## software requirements to process and produce forcing following these instructions:
* git
* Python2.7 with additional libraries (see below)
  * installing conda will make this easier https://conda.io/docs/user-guide/install/index.html. Additional libraries can then be installed with (note some of these may already be installed or be installed by others):
    * conda install pandas
    * conda install numpy
    * conda install -c conda-forge iris
    * conda install -c conda-forge iris-grib
    * conda install matplotlib
    * conda install -c conda-forge gridfill
* CDO (https://code.mpimet.mpg.de/projects/cdo/). This will want to be installed with netcdf and hdf libraries and with multiprocessing support
instructions: https://code.mpimet.mpg.de/projects/cdo/wiki#Download-Compile-Install note this can be tricky - make sure that it used the same netcdf libraries and hdf libraries as your python bits
* The OSU Tidal Data Inversion software, installation described below (http://volkov.oce.orst.edu/tides/)

## Downloading CMIP model data

Data available from https://esgf-node.llnl.gov/search/cmip6/

You will need to set up an account to get access.

You can download it manually, or with wget scripts. With the previous generation of models (CMIP5) I would always use the wget scripts, but I've tried this once through this website and did not get what I expected - I need to look into this. The image below shows which items we need to specify.

![Image of CMIP5 data access](https://github.com/PaulHalloran/s2p3/raw/master/readme_files/cmip6_stuff.jpg)

The variables we want are:

vas, uas, clt, huss, tas, psl, sftlf, rsds and rlds

These correspond to U and V surface winds, cloud fraction, specific humidity (from which we calculate relative humidity), surface air temperature, sea-level pressure, land fraction, net downwelling shortwave radiation and downwelling longwave radiation

Once the files are downloaded, I suggest merging the files for each model and variable into a single file, i.e. one file per variable per model. For this I use `CDO mergetime relevant_input_files*.nc output_filename.nc`
