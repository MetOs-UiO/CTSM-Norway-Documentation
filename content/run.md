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

```{discussion} Remark 
Fates is not automatically checked out with the latest version (as
it is still under development), and this has to be done manually.
```

Follow the tips given at [NordicESMhub](https://github.com/NordicESMhub/ctsm-dev/blob/master/Updating_FATES), which is based on [this](https://github.uio.no/huit/clm5.0_notes/issues/26) insctruction, and the [ESCOMP guide](https://github.com/ESCOMP/ctsm/wiki/Protocols-on-updating-FATES-within-CTSM).

    [~/CTSM_ROOT/cime/scripts]$ ./create_newcase --case ../../../ctsm_cases/fates_f19_g17 --compset 2000_DATM%GSWP3v1_CLM50%FATES_SICE_SOCN_MOSART_SGLC_SWAV --res f19_g17 --machine saga --run-unsupported --project $CESM_ACCOUNT

## Run a single cell case
CLM supports running using single-point or [regional](ttps://metos-uio.github.io/CTSM-Norway-Documentation/run/#run-a-regional-case) datasets that are
customized to a particular region.

In the section below we show you how to run ready-to-use single-point
configurations (out of the box) and then show you how to create your own
dataset for any location of your choice.

### Out of the box
To run for the Brazil test site do the following:

    [~/CTSM_ROOT/cime/scripts]$ export CESM_ACCOUNT=nn2806k
    [~/CTSM_ROOT/cime/scripts]$ ./create_newcase -case ~/cases/testSPDATASET --res 1x1_brazil --compset I2000Clm50SpGs  --machine saga --run-unsupported --project $CESM_ACCOUNT

## Run a regional case
Follow the below procedure to run CTSM over a specific region of interest for a specific resolution. 

### Domain and surface data
To run over the Scandinavia region (latitude 41N to 48N, longitude 4E to 42E) at 0.5 degree resolution you first need to produce the domain and surface data files for the region using the python script `subset_surfdata.py` available under the CTSM tools directory `CTSM_ROOT/tools/contrib/subset_surfdata.py`. The python script file can be found [here](https://github.com/ESCOMP/CTSM/blob/master/tools/contrib/subset_data.py). 



Change the variables that specifies your region, in our case:

    ln1 = 4.0
    ln2 = 42.0 
    lt1 = 41.0 
    lt2 = 48.0 

### Input data 
Use the already available input data stored under the shared directory `/cluster/shared/noresm
` on both `fram` and `saga`. 

To use the atmospheric forcing data you must export the environmental variable `ATMDATA` to point to the right place

    [~/CTSM_ROOT/cime/scripts]$ export ATMDATA=/cluster/shared/noresm/inputdata/atm/datm7/

### Make a case
```{discussion} Remember to check your environmental variables 
Before making your regional case be sure that all your environmental variables are properly defined.
```
To make a case with the Satellite Phenology compset 

    [~/CTSM_ROOT/cime/scripts]$ ./create_newcase --case ~/cases/SCA_SpRun --res CLM_USRDAT --compset I2000Clm50SpGs --machine saga --run-unsupported --project $CESM_ACCOUNT
    
Here, the compset `I2000Clm50SpGs` initialize the model from 2000 conditions with GSWP3 atmospheric forcing data. If you want to run CTSM in the BGC mode and force with CRU NCEP v7 data set, use the `I2000Clm50BgcCruGs` compset. 

You can see the available compset aliases and their long names by typing the following :

    [~/CTSM_ROOT/cime/scripts]$ ./query_config --compsets clm

```{discussion} Single-point and regional cases 
You can only use the compsets with `SGLC` (Stub glacier (land ice) component) for single-point and regional cases. Otherwise, the simulation will fail. WHY??? SHOULD HAVE AN EXPLANATION HERE I (Sunniva) THINK? 
```


### Set up the case 

To do a test simulation for three years between 2000-2002 we would do the following changes in the case directory

    [~/cases/SCA_SpRun]$ export GRIDNAME=0.25_SCA
    [~/cases/SCA_SpRun]$ export SCA_DATA=$MYCTSMDATA/share/domains/
    [~/cases/SCA_SpRun]$ export ATMDOM=domain.0.25x0.25_SCA.nc
    [~/cases/SCA_SpRun]$ ./xmlchange DIN_LOC_ROOT=$MYCTSMDATA
    [~/cases/SCA_SpRun]$ ./xmlchange ATM_DOMAIN_PATH=$SCA_DATA,LND_DOMAIN_PATH=$SCA_DATA
    [~/cases/SCA_SpRun]$ ./xmlchange ATM_DOMAIN_FILE=$ATMDOM,LND_DOMAIN_FILE=$ATMDOM
    [~/cases/SCA_SpRun]$ ./xmlchange CLM_USRDAT_NAME=$GRIDNAME
    [~/cases/SCA_SpRun]$ ./xmlchange STOP_OPTION=nyears
    [~/cases/SCA_SpRun]$ ./xmlchange STOP_N=3
    [~/cases/SCA_SpRun]$ ./xmlchange NTASKS=16
    [~/cases/SCA_SpRun]$ ./xmlchange JOB_WALLCLOCK_TIME="03:59:00"
    [~/cases/SCA_SpRun]$ ./xmlchange PROJECT=$CESM_ACCOUNT
    [~/cases/SCA_SpRun]$ ./xmlchange RUN_STARTDATE="2000-01-01"
    [~/cases/SCA_SpRun]$ ./xmlchange RESUBMIT="0"
    [~/cases/SCA_SpRun]$ ./xmlchange DATM_CLMNCEP_YR_ALIGN="2000"
    [~/cases/SCA_SpRun]$ ./xmlchange DATM_CLMNCEP_YR_START="2000"
    [~/cases/SCA_SpRun]$ ./xmlchange DATM_CLMNCEP_YR_END="2002"
    [~/cases/SCA_SpRun]$ ./xmlchange DOUT_S="FALSE"
    [~/cases/SCA_SpRun]$ ./xmlchange GMAKE_J="8"
    [~/cases/SCA_SpRun]$ ./case.setup
    [~/cases/SCA_SpRun]$ ./preview_run

#### Customizing the CLM namelist 
In order to define the location of the surface data file for our Scandinavia region, we add the variable
`fsurdat` to the file `~/cases/SCA_SpRun/user_nl_ctsm`.  

    fsurdat="$MYCTSMDATA/lnd/clm2/surfdata_map/surfdata_0.25_SCA_hist_16pfts_Irrig_CMIP6_simyr2000_c190814.nc"

If you want to print out only selected variables with hourly resolution in
the model output files for each year you would need to update the file `~/cases/SCA_SpRun/user_nl_ctsm` to contain the following variables 

    hist_empty_htapes = .true.
    hist_fincl1 = 'TSA', 'TSKIN', 'EFLX_LH_TOT', 'FSH', 'WIND', 'TWS', 'SNOWLIQ', 'SNOWICE', 'SNOW_DEPTH', 'TSOI', 'H2OSOI'
    hist_nhtfrq = -1
    hist_mfilt = 365

```{discussion} Variable naiming convention  
The variable names (e.g. `TSA`, `TSKIN`,
`EFLX_LH_TOT`, etc.) are following the CESM name convention.
```

If you run your simulation with the `BGC`-compset, the correct surface data map would be 
    
    fsurdat="$MYCTSMDATA/lnd/clm2/surfdata_map/surfdata_0.25_SCA_hist_78pfts_CMIP6_simyr2000_c190819.nc"


### Build and submit the case 
Now you can perform the final steps to build and submit the case to the job queue.

    [~/cases/SCA_SpRun]$ ./case.build
    [~/cases/SCA_SpRun]$ ./case.submit
