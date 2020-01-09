# s2p3 Downloading CMIP model data to drive downscaling model, and creating forcing files

## software requirements to process and produce forcing following tehse instructions:
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
