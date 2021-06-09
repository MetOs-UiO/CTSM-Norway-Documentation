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
After you have completed one of the methods in the [Get CTSM](https://metos-uio.github.io/CTSM-Norway-Documentation/get/#get-ctsm) step you need to load the correct external repositories. This you do with the command  

    [~/CTSM_ROOT]$ ./manage_externals/checkout_externals

```{discussion} Updating Fates?  
If you are updating FATES go [here](https://github.com/NordicESMhub/ctsm-dev/blob/master/Updating_FATES.md) first.  
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
To run over the Scandinavia region (latitude 41N to 48N, longitude 4E to 42E) at 0.5 degree resolution you first need to produce the domain and surface data files for the region using the python script `subset_data.py` available under the CTSM tools directory `CTSM_ROOT/tools/contrib/subset_data.py`. The python script file can be found [here](https://github.com/ESCOMP/CTSM/blob/master/tools/contrib/subset_data.py). 

The python scripts requires input arguments corresponding to your region of interest. Command below helps you to understand on how to provide
input argument variables:

    ./subset_data.py reg --help
    
First, you need to change the global input data directory variable inside the script (standard global CTSM dataset) as below:

    dir_inputdata='/cluster/shared/noresm/inputdata/'

Run the script as below with the input arguments for Scandinavia region:

    [~/CTSM_ROOT/tools/contrib] ./subset_data.py --reg scand --lat1 41.0 --lat2 48.0 --lon1 4.0 --lon2 42.0 --create_domain True --create_surface True --outdir $MYREGDATA  #(choose where you want to store the regional data eg. /cluste/projects/nn2806k/)

### Create a case for the compset of your interest

You can see the available compset aliases and their long names by typing the following :

    [~/CTSM_ROOT/cime/scripts]$ ./query_config --compsets clm

The compset chosen here is I2000Clm50BgcCropGs. Initialize the model for the year 2000 conditions with GSWP3 atmospheric forcing data.
    
    [~/CTSM_ROOT/cime/scripts]$ export CESM_ACCOUNT=nn2806k
    [~/CTSM_ROOT/cime/scripts]$ ./create_newcase -case ~/cases/scand --res CLM_USRDAT --compset I2000Clm50BgcCropGs  --machine fram --run-unsupported --project $CESM_ACCOUNT
    
### Inputdata

Add domain file path following below commands in the case directory:

    [~/cases/scand]$./xmlchange ATM_DOMAIN_PATH=$MEREGDATA,LND_DOMAIN_PATH=$MYREGDATA
    [~/cases/scand]$export LNDDOM=domain.0.5x0.5_scand.nc
    [~/cases/scand]$./xmlchange ATM_DOMAIN_FILE=$ATMDOM,LND_DOMAIN_FILE=$ATMDOM
    
Add region name into env_run.xml file (eg.choose the same name as created case name):

    [~/cases/scand]$./xmlchange CLM_USRDAT_NAME=scand
    
Change the number of CPUs and few other variables: 

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

```{discussion} Variable naiming convention  
The variable names (e.g. `TSA`, `TSKIN`,
`EFLX_LH_TOT`, etc.) are following the CESM name convention.
```

### Build and submit the case 
Now you can perform the final steps to build and submit the case to the job queue.

    [~/cases/SCA_SpRun]$ ./case.build
    [~/cases/SCA_SpRun]$ ./case.submit
