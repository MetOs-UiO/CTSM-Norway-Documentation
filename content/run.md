# Run CTSM
The [original documentation](https://escomp.github.io/ctsm-docs/versions/master/html/users_guide/setting-up-and-running-a-case/index.html) gives a detailed description of how to create and run a case. If this is the first time you use CTSM you should always start there. 

The examples you can find on this page are tailored to the Norwegian clusters.  

## Use your project 
For running CTSM on the Norwegian cluster you will be affiliated with a project account. 
To check which project you are affiliated with can run by typing

    [~/HOME]$ projects

after logging into a remote HPC-cluster. This will return something that looks like this

    nn2806k
    nn1000k

In the example above, two projects can be used (nn2806k and nn1000k). Most of our examples are connected to project `nn2806k`. The project you are affiliated with will depend on the account you used when you applied for resources. Make sure you choose the right project when running CTSM. 

```{discussion} Change account  
To change the variable `CESM_ACCOUNT` in a script  

    export CESM_ACCOUNT=nn2806k
```

### Environmental variables
Along with the proper definition of your project, you should make sure that some other environmental variables are properly defined. 

An example of environmental variables your might want to change are

    [~/HOME]$ export CESM_ACCOUNT=nn2806k
    [~/HOME]$ export PROJECT=nn2806k
    [~/HOME]$ export CTSM_ROOT=/cluster/home/$USER/ctsm

    [~/HOME]$ export INC_NETCDF=/cluster/software/VERSIONS/netcdf.intel-4.3.3.1/
    [~/HOME]$ export LIB_NETCDF=/cluster/software/VERSIONS/netcdf.intel-4.3.3.1/
    [~/HOME]$ export NETCDF_ROOT=/cluster/software/VERSIONS/netcdf.intel-4.3.3.1/


## Your first CTSM case

```{discussion} Updating Fates?  
If you are updating FATES go [here](https://github.com/NordicESMhub/ctsm-dev/blob/master/Updating_FATES.md) first.  
```

```{discussion} For reference: manage_externals  
Whenever you want to update to the most recent version of the external components, you need to rerun the command below.

If you are doing changes to the code, remember to create your own branches for the external repositories before you run the command (see [here](https://metos-uio.github.io/CTSM-Norway-Documentation/get/#from-the-nordicesm-hub)).

    [~/CTSM_ROOT]$ ./manage_externals/checkout_externals

Read more about how manage_externals/checkout_externals work [here](https://github.com/ESMCI/manage_externals).

```


### The input data directory
The first time, or whenever your `$WORKDIR` is cleaned, you need to link your working directory with your input data

    [~/CTSM_ROOT]$ cd cime/scripts
    [~/CTSM_ROOT/cime/scripts]$ ./link_dirtree $CESM_DATA /work/users/$USER/inputdata

### Create a case
Before you can run the model you must create a new case. 

    [~/CTSM_ROOT/cime/scripts]$ ./create_newcase --case ~/cases/I2000Clm50BgcCruGs  --compset I2000Clm50BgcCruGs   --res f19_g17 --machine saga --run-unsupported --project $CESM_ACCOUNT

Make sure that you set the arguments for `--machine` and `--project` which are relevant for yourself. 

### Configure your case 
To properly configure your case there are several steps you might need to do. 

1. Navigate to your newly created case folder

        [~/CTSM_ROOT/cime/scripts]$ cd ~/cases/I2000Clm50BgcCruGs

1. Check the current configuration

        [~/cases/I2000Clm50BgcCruGs]$ ./xmlquery --l #(--l list --f file) 

    e.g.

        [~/cases/I2000Clm50BgcCruGs]$ ./xmlquery STOP_OPTION

2. Change the current configuration

    For instance, you might want to change the duration of a simulation to 5 days:


        [~/cases/I2000Clm50BgcCruGs]$ ./xmlchange STOP_OPTION=ndays #(nyears, nmonths)
        [~/cases/I2000Clm50BgcCruGs]$ ./xmlchange STOP_N=5 #(then 5 days)

    ```{discussion} xlm-files  
    To manually edit the `*.xlm`-files is **not** recommended! 
    ```
    
3. Setup your case

        [~/cases/I2000Clm50BgcCruGs]$ ./case.setup 

4. Change history filelds 

    The history filelds controls which variables that are written to file during a run. `hist_mfilt` allows you to specify the number of output
    files and `hist_nhtfrq` the frequency.

    To change the history files you can edit the `user_nl_clm.xlm` add the following lines below

        hist_mfilt=5 #(number of output files)
        hist_nhtfrq=-24 #(means daily outputs)
 
5. Build your case

        [~/cases/I2000Clm50BgcCruGs]$ ./case.build

    ```{discussion} Build failed?  
    Make sure you clean the previous build

        [~/cases/I2000Clm50BgcCruGs] ./case.build --clean
    ```
6. Run your case

        [~/cases/I2000Clm50BgcCruGs]$ ./case.submit 

## Running a FATES case

Follow the tips given at [ESCOMP guide](https://github.com/ESCOMP/ctsm/wiki/Protocols-on-updating-FATES-within-CTSM).

    [~/CTSM_ROOT/cime/scripts]$ ./create_newcase --case ../../../ctsm_cases/fates_f19_g17 --compset 2000_DATM%GSWP3v1_CLM50%FATES_SICE_SOCN_MOSART_SGLC_SWAV --res f19_g17 --machine saga --run-unsupported --project $CESM_ACCOUNT

## Run a single cell case
CLM supports running single-point simulations. Available options are summarized at [ESCOMP guide](https://escomp.github.io/ctsm-docs/versions/master/html/users_guide/running-single-points/single-point-and-regional-grid-configurations.html#single-and-regional-grid-configurations). 

For testing purpose, we recommend running a single point using global data, i.e., [PTS_MODE](https://escomp.github.io/ctsm-docs/versions/master/html/users_guide/running-single-points/running-pts_mode-configurations.html#running-a-single-point-using-global-data-pts-mode). Below we show you how to create a single-point case using global data.

    [~/CTSM_ROOT/cime/scripts]$ export CESM_ACCOUNT=nn2806k
    [~/CTSM_ROOT/cime/scripts]$ ./create_newcase -case ~/cases/testPTS_MODE --res f19_g17_gl4 --compset I2000Clm50SpGs -pts_lat 40.0 -pts_lon -105 --machine saga --run-unsupported --project $CESM_ACCOUNT

For dedicated single-cell simulation in order to compare with site-level observation data, we recommend running single-cell simulation using your own datasets following the examples (1.6.3.4 - 1.6.3.6) from [ESCOMP guide] (https://escomp.github.io/ctsm-docs/versions/master/html/users_guide/running-single-points/running-single-point-configurations.html#example-using-clm-usrdat-name-to-run-a-simulation-using-user-datasets-for-a-specific-region-over-alaska) 

There are dedicated tools and workflow for running single-cell simulations over Norwegian ecological observation sites using CLM and CLM-FATES, please check [NorESM_LandSites_Platform](https://github.com/NorESMhub/NorESM_LandSites_Platform) and follow the instructions there.

## Run a regional case
Follow the below procedure to run CTSM over a specific region of interest for a specific resolution. 

### Domain and surface data
To run over the Scandinavia region (latitude 48N to 81N, longitude 4E to 42E) at 0.5 degree resolution you first need to produce the domain and surface data files for the region using the python script `subset_data.py` available under the CTSM tools directory `CTSM_ROOT/tools/contrib/subset_data.py`. The python script file can be found [here](https://github.com/ESCOMP/CTSM/blob/master/tools/contrib/subset_data.py). 

The python scripts requires input arguments corresponding to your region of interest. Command below helps you to understand on how to provide
input argument variables:

    ./subset_data.py reg --help
    
First, you need to change the global input data directory variable inside the script (standard global CTSM dataset) as below:

    dir_inputdata='/cluster/shared/noresm/inputdata/'
    
Choose a location for storing the domain and surface data files for Scandinavia (eg. /cluster/projects/nn2806k/$USER/regiondata). 
Set the environment variable for the directory as below:

    MYREGDATA='/cluster/projects/nn2806k/$USER/regiondata/'

```{discussion} Choose the global surface data file corresponding to the resolution! 
For example: for 0.5x0.5 resolution, choose `surfdata_360x720cru_16pfts_Irrig_CMIP6_simyr2000_c170824.nc` (under shared noresm folder)
and add this file location in the python script `subset_data.py`
```

Run the script as below with the input arguments for Scandinavia region:

    [~/CTSM_ROOT/tools/contrib] ./subset_data.py --reg scand --lat1 48.0 --lat2 81.0 --lon1 4.0 --lon2 42.0 --create_domain True --create_surface True --outdir $MYREGDATA

### Create a case for the compset of your interest

You can see the available compset aliases and their long names by typing the following :

    [~/CTSM_ROOT/cime/scripts]$ ./query_config --compsets clm

The compset chosen here is I2000Clm50BgcCropGs. Initializes the model for the year 2000 conditions with GSWP3 atmospheric forcing data.
    
    [~/CTSM_ROOT/cime/scripts]$ export CESM_ACCOUNT=nn2806k
    [~/CTSM_ROOT/cime/scripts]$ ./create_newcase -case ~/cases/scand --res CLM_USRDAT --compset I2000Clm50BgcCropGs  --machine fram --run-unsupported --project $CESM_ACCOUNT
    
### Inputdata

Change data atmosphere domain (ATM_DOMAIN_PATH) and land domain (LND_DOMAIN_PATH) paths following below commands in the case directory:

    [~/cases/scand]$./xmlchange ATM_DOMAIN_PATH=$MYREGDATA,LND_DOMAIN_PATH=$MYREGDATA
    
```{discussion} xml-files
Manually editing the `*.xml`-files is **not** advised!
```

To add the domain file names (same domain file for both ATM_DOMAIN_FILE and LND_DOMAIN_FILE) first set the environment variable for file name:

    [~/cases/scand]$export DOMFILE=domain.0.5x0.5_scand.nc

and use the xmlchange command to add the file names into the xml files.

    [~/cases/scand]$./xmlchange ATM_DOMAIN_FILE=$DOMFILE,LND_DOMAIN_FILE=$DOMFILE
    
Add region name into env_run.xml file (eg.choose the same name as created case name):

    [~/cases/scand]$./xmlchange CLM_USRDAT_NAME=scand
    
Change the number of CPUs and project number: 

    [~/cases/scand]$./xmlchange NTASKS=32
    [~/cases/scand]$./xmlchange PROJECT=$CESM_ACCOUNT
    
Setup the case: 

    [~/cases/scand]$ ./case.setup
    [~/cases/scand]$ ./preview_run

#### Customizing the CLM namelist 

Specify the location of surface data created for the region in the file `user_nl_ctsm`: 

    fsurdat="$MYREGDATA/surfdata_0.5_scand_hist_16pfts_Irrig_CMIP6_simyr2000_c190814.nc"

    

```{discussion} Single-point and regional cases 
You can only use the compsets with `SGLC` (Stub glacier (land ice) component) for single-point and regional cases. Otherwise, the simulation will fail. WHY??? SHOULD HAVE AN EXPLANATION HERE I (Sunniva) THINK? 
```
If you want to print out only selected variables with hourly resolution in
the model output files for each year you would need to update the file `~/cases/scand/user_nl_ctsm` to contain the following variables 

    hist_empty_htapes = .true.
    hist_fincl1 = 'TSA', 'TSKIN', 'EFLX_LH_TOT', 'FSH', 'WIND', 'TWS', 'SNOWLIQ', 'SNOWICE', 'SNOW_DEPTH', 'TSOI', 'H2OSOI'
    hist_nhtfrq = -1
    hist_mfilt = 365

```{discussion} Variable naming convention
The variable names (e.g. `TSA`, `TSKIN`,
`EFLX_LH_TOT`, etc.) are following the CESM name convention.
```

### Build and submit the case 
Now you can perform the final steps to build and submit the case to the job queue.

    [~/cases/SCA_SpRun]$ ./case.build
    [~/cases/SCA_SpRun]$ ./case.submit

## Example scripts

### Regional simulations

`````{tabs}

````{tab} Scandinavia (0.5&deg;)

This example Bash script shows the steps needed to run a regional simulation
on Fram, using the setup explained in the
["Run a regional case" section](https://metos-uio.github.io/CTSM-Norway-Documentation/run/#run-a-regional-case).

```bash
# Project accounting
export CESM_ACCOUNT=nn2806k
export PROJECT=nn2806k

# Paths: CLM and NetCDF
export CTSM_ROOT=$HOME/ctsm_fates_emerald
export INC_NETCDF=/cluster/software/VERSIONS/netcdf.intel-4.3.3.1/
export LIB_NETCDF=/cluster/software/VERSIONS/netcdf.intel-4.3.3.1/
export NETCDF_ROOT=/cluster/software/VERSIONS/netcdf.intel-4.3.3.1/

# Input
export CESM_DATA=/cluster/shared/noresm/inputdata
export DIR_DOM=${CESM_DATA}/share/domains
export DOMFILE=domain.0.5x0.5_scand.nc
export SURF_DATA=${CESM_DATA}/lnd/clm2/surfdata_map/surfdata_0.5_scand_hist_16pfts_Irrig_CMIP6_simyr2000_c190814.nc
export ROOT_FRC=/cluster/shared/noresm/inputdata/atm/datm7

# Model components (COMPSET)
export COMPSET=I2000Clm50SpGs
# (listed by $CTSM_ROOT/cime/scripts/query_config --COMPSETs clm)

# Case folder
export GRID_NAME=0.5_scand
export CASE_NAME=${COMPSET}_${GRID_NAME}
export DIR_CASE=~/cases/$CASE_NAME

# Get CLM from NordicESMhub and externals
if ! [ -d $CTSM_ROOT ]; then
    module load git/2.23.0-GCCcore-8.3.0
    git clone -b release-clm5.0 https://github.com/NordicESMhub/ctsm.git $CTSM_ROOT
    cd $CTSM_ROOT
    ./manage_externals/checkout_externals
fi;

# Create case
if ! [ -d $DIR_CASE ]; then
    cd $CTSM_ROOT/cime/scripts
    ./create_newcase --case $DIR_CASE --compset $COMPSET --res CLM_USRDAT \
        --machine fram --run-unsupported --project $CESM_ACCOUNT
fi;

# Modify XML configuration files generated by case creation
cd $DIR_CASE
./xmlchange DIN_LOC_ROOT=$CESM_DATA
./xmlchange DIN_LOC_ROOT_CLMFORC=$ROOT_FRC
./xmlchange ATM_DOMAIN_PATH=$DIR_DOM,LND_DOMAIN_PATH=$DIR_DOM
./xmlchange ATM_DOMAIN_FILE=$DOMFILE,LND_DOMAIN_FILE=$DOMFILE
./xmlchange CLM_USRDAT_NAME=$GRID_NAME
./xmlchange STOP_OPTION="ndays"
./xmlchange STOP_N=5
./xmlchange NTASKS=16
./xmlchange JOB_WALLCLOCK_TIME="00:05:00"
./xmlchange PROJECT=$CESM_ACCOUNT
./xmlchange RUN_STARTDATE="2000-01-01"
./xmlchange RESUBMIT="0"
./xmlchange DATM_CLMNCEP_YR_ALIGN="2000"
./xmlchange DATM_CLMNCEP_YR_START="2000"
./xmlchange DATM_CLMNCEP_YR_END="2000"
./xmlchange DOUT_S="FALSE"
./xmlchange GMAKE_J="8"

# Case setup
./case.setup

# Add surface data to user namelist
cat << EOF > user_nl_clm
fsurdat="$SURF_DATA"
hist_nhtfrq=-24
EOF
# (if restart file is available from spin-up ($RESTART), add: finidat="$RESTART")

# Preview
./preview_run

# Compile
./case.build
#./case.build --clean

# Run
./case.submit

# Check output files
ll $USERWORK/noresm/$CASE_NAME/run/${CASE_NAME}.clm2.h0.*.nc

# View slurm log
cat $DIR_CASE/${CASE_NAME}.run
```
````

````{tab} N. Europe (0.25&deg;)

This example Bash script shows the steps needed to run a regional simulation
over Northern Europe at 0.25&deg; resolution on Fram, using
[CLM from NordicESM hub](https://github.com/NordicESMhub/ctsm).
It differs from the
[regional case workflow](https://metos-uio.github.io/CTSM-Norway-Documentation/run/#run-a-regional-case)
in the following ways:
- `$CTSM_ROOT/tools/contrib/subset_data.py` is not used.
- it relies on domain and surface files located in
  `/cluster/projects/nn9373k/data_regional_SBS/`.

```bash
# Project accounting
export CESM_ACCOUNT=nn9373k
export PROJECT=nn9373k

# Paths: CLM, CIME and NetCDF
export CTSMROOT=$HOME/ctsm_fates_emerald
export CIMEROOT=$CTSMROOT/cime
export INC_NETCDF=/cluster/software/VERSIONS/netcdf.intel-4.3.3.1/
export LIB_NETCDF=/cluster/software/VERSIONS/netcdf.intel-4.3.3.1/
export NETCDF_ROOT=/cluster/software/VERSIONS/netcdf.intel-4.3.3.1/

# Input
export CESM_DATA=/cluster/shared/noresm/inputdata
export MYCTSMDATA=$USERWORK/inputdata
export REG_DATA=/cluster/projects/nn9373k/data_regional_SBS
export root_frc=/cluster/shared/noresm/inputdata/atm/datm7

# Model components (compset)
export compset=I2000Clm50SpGs
# (listed by $CTSMROOT/cime/scripts/query_config --compsets clm)

# Domain info
export GRIDNAME=0.25_SBS
export ATMDOM=domain.0.25x0.25_SBS.nc

# Case folder
export case_name=${compset}_${GRIDNAME}
export casedir=~/cases/$case_name

# Get CLM from NordicESMhub and externals
if ! [ -d $CTSMROOT ]; then
    module load git/2.23.0-GCCcore-8.3.0
    git clone -b release-clm5.0 https://github.com/NordicESMhub/ctsm.git $CTSMROOT
    cd $CTSMROOT
    ./manage_externals/checkout_externals
fi;

# Link shared input folder to user's work folder
if ! [ -d $MYCTSMDATA ]; then
    cd $CIMEROOT/scripts
    ./link_dirtree $CESM_DATA $MYCTSMDATA
fi;

# Create case
if ! [ -d $casedir ]; then
    cd $CIMEROOT/scripts
    ./create_newcase --case $casedir --compset $compset --res CLM_USRDAT \
        --machine fram --run-unsupported --project $CESM_ACCOUNT
fi;

# Modify XML configuration files generated by case creation
cd $casedir
./xmlchange DIN_LOC_ROOT=$MYCTSMDATA
./xmlchange DIN_LOC_ROOT_CLMFORC=$root_frc
./xmlchange ATM_DOMAIN_PATH=$REG_DATA,LND_DOMAIN_PATH=$REG_DATA
./xmlchange ATM_DOMAIN_FILE=$ATMDOM,LND_DOMAIN_FILE=$ATMDOM
./xmlchange CLM_USRDAT_NAME=$GRIDNAME
./xmlchange STOP_OPTION="nyears"
./xmlchange STOP_N=1
./xmlchange NTASKS=16
./xmlchange JOB_WALLCLOCK_TIME="01:40:00"
./xmlchange PROJECT=$CESM_ACCOUNT
./xmlchange RUN_STARTDATE="2000-01-01"
./xmlchange RESUBMIT="0"
./xmlchange DATM_CLMNCEP_YR_ALIGN="2000"
./xmlchange DATM_CLMNCEP_YR_START="2000"
./xmlchange DATM_CLMNCEP_YR_END="2000"
./xmlchange DOUT_S="FALSE"
./xmlchange GMAKE_J="8"

# Case setup
./case.setup

# Add surface data to user namelist
cat << EOF > user_nl_clm
fsurdat="$REG_DATA/surfdata_0.25_SBS_hist_16pfts_Irrig_CMIP6_simyr2000_c191002.nc"
hist_nhtfrq=-24
EOF
# (if restart file is available from spin-up ($RESTART), add: finidat="$RESTART")

# Preview
./preview_run

# Compile
#./case.build --clean
./case.build

# Run
./case.submit

# Check output files
ll $USERWORK/noresm/$case_name/run/${case_name}.clm2.h0.*.nc

# View slurm log
cat $casedir/${case_name}.run
```
````

`````
