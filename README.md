# s2p3 Downloading CMIP model data to drive downscaling model, and creating forcing files

## software requirements to process and produce forcing following tehse instructions:
* git
* Python2.7 with additional libraries (see below)
<p>installing conda will make this easier https://conda.io/docs/user-guide/install/index.html<p>
additional libraries can then be installed with (note some of these may already be installed or be installed by others):
conda install pandas
conda install numpy
conda install -c conda-forge iris
conda install -c conda-forge iris-grib
conda install matplotlib
conda install -c conda-forge gridfill
CDO (https://code.mpimet.mpg.de/projects/cdo/)
instructions: https://code.mpimet.mpg.de/projects/cdo/wiki#Download-Compile-Install note this can be tricky - make sure that it used the same netcdf libraries and hdf libraries as your python bits and pieces
this may work:
sudo apt-get install libnetcdf-dev libhdf5-dev
Download latest stable version of cdo from https://code.mpimet.mpg.de/projects/cdo/files
e.g.
wget https://code.mpimet.mpg.de/attachments/download/17094/cdo-1.9.4rc2.tar.gz
unpack it (make sure the file name is correct if you've downloaded a different version):
tar zxvf cdo-1.9.4rc2.tar.gz
move into the unpacked directory (making sure to specify the correct directory name)
cd cdo-1.9.4rc2
./configure --enable-netcdf4 --enable-zlib --with-netcdf=/usr/ --with-hdf5=/usr/
make
sudo make install
The OSU Tidal Data Inversion software, installation described below (http://volkov.oce.orst.edu/tides/)
