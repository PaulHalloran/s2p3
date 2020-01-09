# s2p3 Downloading CMIP model data to drive downscaling model, and creating forcing files

## software requirements to process and produce forcing following these instructions:
* git
* Python2.7 with additional libraries (see below)
  * installing conda will make this easier https://conda.io/docs/user-guide/install/index.html. Additional libraries can then be installed with (note some of these may already be installed or be installed by others):
    * `conda install pandas`
    * `conda install numpy`
    * `conda install -c conda-forge iris`
    * `conda install -c conda-forge iris-grib`
    * `conda install matplotlib`
    * `conda install -c conda-forge gridfill`
* CDO (https://code.mpimet.mpg.de/projects/cdo/). This will want to be installed with netcdf and hdf libraries and with multiprocessing support
instructions: https://code.mpimet.mpg.de/projects/cdo/wiki#Download-Compile-Install note this can be tricky - make sure that it used the same netcdf libraries and hdf libraries as your python bits
* The OSU Tidal Data Inversion software, installation described below (http://volkov.oce.orst.edu/tides/)

## Downloading CMIP model data

Data available from https://esgf-node.llnl.gov/search/cmip6/

Some potentially useful context here: https://www.earthsystemcog.org/site_media/projects/wip/CMIP6_global_attributes_filenames_CVs_v6.2.6.pdf

and here https://pcmdi.llnl.gov/ipcc/standard_output.pdf?id=2

You will need to set up an account to get access.

You can download it manually, or with wget scripts. With the previous generation of models (CMIP5) I would always use the wget scripts, but I've tried this once through this website and did not get what I expected - I need to look into this. The image below shows which items we need to specify.

![Image of CMIP5 data access](https://github.com/PaulHalloran/s2p3/raw/master/readme_files/cmip6_stuff.jpg)

*Note in addition to what I have specified in the image above, we will also need the variables for the 'historical' Experiment ID*

The variables we want are:

vas, uas, clt, huss, tas, psl, sftlf, rsds and rlds

These correspond to U and V surface winds, cloud fraction, specific humidity (from which we calculate relative humidity), surface air temperature, sea-level pressure, land fraction, net downwelling shortwave radiation and downwelling longwave radiation

Once the files are downloaded merge the files for each model and variable into a single file, i.e. one file per variable per model, which includes the historical experiment and the future experiment (i.e. combines historical and SSP585 into one file and historical and SSP119 into another file). with the name format:

 `MODEL-NAME_VARIABLE-NAME_EXPERIMENT-NAME_all.nc` e.g `MIROC-ESM_tas_HistoricalAndSSP119_all.nc`

For this I use cdo e.g. on the linux command line: `cdo mergetime relevant_input_files*.nc output_filename.nc`, or in practice I would have this in a python script which loops through them all and calls CDO using the subprocess module.

The land sea mask file (sftlf), needs to be in the name format `sftlf_fx_'+cmip_model+'_*.nc`

## Scripts for producing the forcing files

clone this repository to your computer

`git clone https://paulhalloran@bitbucket.org/paulhalloran/s2p3_rv2.0_forcing.git`

change into the s2p3_rv2.0_forcing directory

`cd s2p3_rv2.0_forcing`

Download OSU data and software:

download the TPXO9.1 bin file

`wget ftp://ftp.oce.orst.edu/dist/tides/Global/tpxo9.tar.gz`

uncompress this file:

`tar zxvf tpxo9.tar.gz`

get the tidal calculation software

`wget ftp://ftp.oce.orst.edu/dist/tides/OTPS2.tar.Z`

uncompress this

`tar zxvf OTPS2.tar.Z`

move the data you have downloaded into the current working directory (ignoring the waning about 'cannot move...'):

`mv OTPS2/* .`

(note, ignore the warning: mv: cannot move 'OTPS2/DATA' to './DATA': Directory not empty)

then move the data out of a subdirectory to your working directory:

`mv OTPS2/DATA/load_file DATA/`

then move our own versino of the Model_atlas file (specific to our requirements here) to our data directory

`mv Model_atlas DATA/`

if required install gfortran

`sudo apt-get install gfortran`

compile the code (using the three lines below), not worrying about 'warning' messages:

`gfortran -o extract_HC -fconvert=swap -frecord-marker=4 extract_HC.f90 subs.f90`

`gfortran -o predict_tide -fconvert=swap -frecord-marker=4 predict_tide.f90 subs.f90`

`gfortran -o extract_local_model -fconvert=swap -frecord-marker=4 extract_local_model.f90 subs.f90`

## Producing the domain file containing the latitude, longitude, tidal and bathymetry data

Get hold of the ETOP01 bathymety file:

`wget https://www.ngdc.noaa.gov/mgg/global/relief/ETOPO1/thredds/ETOPO1_Bed_g_gmt4.nc`

edit the first 6 lines in tides_bathymetry.py (downloaded above) to specify the domain you want the model to run for and the horizonal resolution (lines copied below for illustration). I wopudl suggest the below values:

`minimum_latitude = -35.0
maximum_latitude = 35.0
latitude_resolution = 0.1 #degrees

minimum_longitude = -180
maximum_longitude = 180
longitude_resolution = 0.1 #degrees`

point the line `location_of_bathymetry_file =  '/data/NAS-ph290/ph290/observations/ETOP/ETOPO1_Bed_g_gmt4.nc'` to your downloaded bathymetry file (see start of this section)

produce the domain file by running tides_bathymetry.py

`python2.7 tides_bathymetry.py`

This produces the file: `s12_m2_s2_n2_h_map.dat`

## Producing the nutrient initialisation file

download the World Ocean Atlas Nutrient data:

`wget https://data.nodc.noaa.gov/thredds/fileServer/ncei/woa/nitrate/all/1.00/woa18_all_n00_01.nc`

edit the first three lines of

`initialisation_nitrate.py`

to:

`min_depth_lim = 10.0
max_depth_lim = 50.0

output_file_name = 'initial_nitrate.dat'
domain_file_name = 's12_m2_s2_n2_h_map.dat' # note this must match the name of your domain file - see above step
location_of_World_Ocean_Atlas13_Nitrate_file =  '/data/NAS-geo01/ph290/observations/woa13_all_n13_01.nc' # This must match the World Ocean Atlas Nutrient file you've downloaded above`

then run with:

`python2.7 initialisation_nitrate.py`

## Producing the meterological forcing files (note this takes quite a while!)

get the processing script:

`wget https://raw.githubusercontent.com/PaulHalloran/s2p3/master/s2p3_forcing/process_cmip6_era5_for_s2p3_rv2.0.py`

Potentially edit the lines in the file process_cmip6_era5_for_s2p3_rv2.0.py which loosely below the lines below if you want to e.g. change the years over which the model is run.

`min_depth_lim = 10.0
max_depth_lim = 50.0

start_year = 1970
end_year = 2100`

Make sure that the line `domain_file = 's12_m2_s2_n2_h_map.dat'` matches the name of yoru domain file (created above)

Specify the directory holding the CMIP6 data you have downloaded and processed, and the directory you want the final forcing files to go to:

`directory_containing_files_to_process = '/data/BatCaveNAS/ph290/ecmwf_era5/day_mean/region_of_interest/'
output_directory = '/data/BatCaveNAS/ph290/ecmwf_era5/day_mean/output_processed/global_tropics/'`

Run the script to produce the forcing files:

`python2.7 process_cmip6_era5_for_s2p3_rv2.0.py`

Things to note:
- I've made some minor edits to this and have not confirmed it runs yet, if it hits a problem just let me know
- This script can take a good few hours to run
