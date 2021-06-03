# Run CTSM
The [original documentation](https://escomp.github.io/ctsm-docs/versions/master/html/users_guide/setting-up-and-running-a-case/index.html) gives a detailed description of how to create and run a case. If this is the first time you use CTSM you should always start there. 

The examples you can find on this page are tailored to the Norwegian clusters.  

## Use your project 
For running CTSM on the Norwegian cluster you will be affilicated with a project account. 
To check which project you are affiliated with can run by typing

    [~/HOME]$ projects

after logging to a remote HPC-computer. This will return something that looks like this

    nn2806k
    nn1000k

In the example above, two projects can be used (nn2806k and nn1000k). Most of our examples are connected to project `nn2806k`. The project you are affiliated with will depend on the accout you used when you applied for resources. Make sure you choose the right project when running CTSM. 

```{discussion} Change account  
To change the variable `CESM_ACCOUNT` in a script  

    export CESM_ACCOUNT=nn2806k
```

### Environmental variables
Along with the proper definition of your project you should make sure that some other environmental variables are propely defined. 

An example of environmental variables your might want to change are

    [~/HOME]$ export CESM_ACCOUNT=nn2806k
    [~/HOME]$ export PROJECT=nn2806k
    [~/HOME]$ export CTSMROOT=/cluster/home/$USER/ctsm

    [~/HOME]$ export INC_NETCDF=/cluster/software/VERSIONS/netcdf.intel-4.3.3.1/
    [~/HOME]$ export LIB_NETCDF=/cluster/software/VERSIONS/netcdf.intel-4.3.3.1/
    [~/HOME]$ export NETCDF_ROOT=/cluster/software/VERSIONS/netcdf.intel-4.3.3.1/


## Your first CTSM case
After you have completing one of the methods in the [Get CTSM](https://metos-uio.github.io/CTSM-Norway-Documentation/get/#get-ctsm) step you need to load the correct external repositories. This you do with the command  

    [~/CTSM_ROOT]$ ./manage_externals/checkout_externals

```{discussion} Updating Fates?  
If you are updating FATES go [here](https://github.com/NordicESMhub/ctsm-dev/blob/master/Updating_FATES.md) first.  
```

### Inputdata
The first time, or whenever your `$WORKDIR` is cleaned, you need to link your working directory with your input data

    [~/CTSM_ROOT]$ cd cime/scripts
    [~/CTSM_ROOT/cime/scripts]$ ./link_dirtree $CESM_DATA /work/users/$USER/inputdata

### Create a case
Before you can run the model you must create a new case. 

    [~/CTSM_ROOT/cime/scripts]$ ./create_newcase --case ~/cases/I2000Clm50BgcCruGs  --compset I2000Clm50BgcCruGs   --res f19_g17 --machine saga --run-unsupported --project $CESM_ACCOUNT

Make sure that you set the arguments for `--machine` and `--project` which are relevant for yourself. 

### Configure your case 
To properly confugure your case there are several steps you might need to do. 

1. Navigate to your newly created case folder

        [~/CTSM_ROOT/cime/scripts]$ cd ~/cases/I2000Clm50BgcCruGs

1. Check the current configuration

        [~/cases/I2000Clm50BgcCruGs]$ ./xmlquery --l #(--l list --f file) 

    eg

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

    The history filelds controls which varaibles that are written to file during a run. `hist_mfilt` allows you to specify the number of output
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

In the section below we show you how to run ready to use single-point
configurations (out of the box) and then show you how to create your own
dataset for any location of your choice.

### Out of the box
To run for the Brazil test site do the following:

    [~/CTSM_ROOT/cime/scripts]$ export CESM_ACCOUNT=nn2806k
    [~/CTSM_ROOT/cime/scripts]$ ./create_newcase -case ~/cases/testSPDATASET --res 1x1_brazil --compset I2000Clm50SpGs  --machine saga --run-unsupported --project $CESM_ACCOUNT

## Run a regional case
Follow the below procedure to run CTSM over a specific region of interest for a specific resolution. 

### Domain and surface data
To run over Scandinavia region (latitude 41N to 48N, longitude 4E to 42E) at 0.5 degree resolution you first need to produce the domain and surface data files for the region using the python script `subset_surfdata.py` available under the CTSM tools directory (`CTSM_ROOT/tools/contrib/subset_surfdata.py`).

Change the variables that specifies your region, in our case:

    ln1 = 4.0
    ln2 = 42.0 
    lt1 = 41.0 
    lt2 = 48.0 

The python script file can be found [here](https://github.com/ESCOMP/CTSM/blob/master/tools/contrib/subset_data.py). 


### Inputdata
Use the already available inputdata stored under the shared directory `/cluster/shared/noresm
` on both `fram` and `saga`. 

To use the atmospheric forcing data you must export the environmental variable `ATMDATA` to point to the right place

    [~/CTSM_ROOT/cime/scripts]$ export ATMDATA=/cluster/shared/noresm/inputdata/atm/datm7/

### Make a case
```{discussion} Remember to check your environmantal variables 
Before making your regional case be sure that all your environmental variables are properly defined.
```
To make a case with Satellite Phenology mode 

    [~/CTSM_ROOT/cime/scripts]$ ./create_newcase --case ~/cases/SCA_SpRun --res CLM_USRDAT --compset I2000Clm50SpGs --machine saga --run-unsupported --project $CESM_ACCOUNT
    
Here, the compset `I2000Clm50SpGs` initialize the model from 2000 conditions with GSWP3 atmospheric forcing data. If you want to run CTSM with BGC mode and force with CRU NCEP v7 data set, use the `I2000Clm50BgcCruGs` compset. 

You can see the available compset aliases and their long names by typing the following :

    [~/CTSM_ROOT/cime/scripts]$ ./query_config --compsets clm

```{discussion} Single point and regional cases 
You can only use the compsets with `SGLC` (Stub glacier (land ice) component) for single point and regional cases. Otherwise, the simulation will fail. WHY??? SHOULD HAVE AN EXPLANATION HERE I (Sunniva) THINK? 
```


### Set up the case 

Can be used for a reduced size of the carbon pools.
The following code would be saved in the script `~/MY_CASE/tailor_my_case_acc_spin_up.sh`,
to tailor the type of case you want to run

````        
./xmlchange CLM_FORCE_COLDSTART="on"
./xmlchange CLM_NML_USE_CASE="2000_control"
./xmlchange DATM_CLMNCEP_YR_START="1991"
./xmlchange DATM_CLMNCEP_YR_END="2010"
./xmlchange DATM_PRESAERO="clim_2000"
./xmlchange CCSM_CO2_PPMV="369."
./xmlchange STOP_OPTION="nyears"
./xmlchange RUN_REFDATE="0001-01-01"
./xmlchange RUN_STARTDATE="0001-01-01"
#Additional xml changes for AD mode
./xmlchange CLM_ACCELERATED_SPINUP="on"
#Number of years for spin-up 
#For Tropics only STOP_N=100 is needed!
./xmlchange STOP_N=100
#Frequency of restart files (1/4 of STOP_N)
./xmlchange REST_N=25
#Setting wall clock time (on saga 5 years / 30 min)
./xmlchange JOB_WALLCLOCK_TIME=36:00:00
./case.setup
./preview_namelists
````

Now, we are setting up our test
simulation for three years between 2000-2002.

    cd ~/cases/SCA_SpRun
    export GRIDNAME=0.25_SCA
    export SCA_DATA=$MYCTSMDATA/share/domains/
    export ATMDOM=domain.0.25x0.25_SCA.nc

    ./xmlchange DIN_LOC_ROOT=$MYCTSMDATA
    ./xmlchange ATM_DOMAIN_PATH=$SCA_DATA,LND_DOMAIN_PATH=$SCA_DATA
    ./xmlchange ATM_DOMAIN_FILE=$ATMDOM,LND_DOMAIN_FILE=$ATMDOM
    ./xmlchange CLM_USRDAT_NAME=$GRIDNAME
    ./xmlchange STOP_OPTION=nyears
    ./xmlchange STOP_N=3
    ./xmlchange NTASKS=16
    ./xmlchange JOB_WALLCLOCK_TIME="03:59:00"
    ./xmlchange PROJECT=$CESM_ACCOUNT
    ./xmlchange RUN_STARTDATE="2000-01-01"
    ./xmlchange RESUBMIT="0"
    ./xmlchange DATM_CLMNCEP_YR_ALIGN="2000"
    ./xmlchange DATM_CLMNCEP_YR_START="2000"
    ./xmlchange DATM_CLMNCEP_YR_END="2002"
    ./xmlchange DOUT_S="FALSE"
    ./xmlchange GMAKE_J="8"
    ./case.setup
    ./preview_run

### Customizing the CLM namelist 
In order to define the location of surface data file for our Scandinavia region, we add the variable
`fsurdat` setting to the land model's namelist.

    cat << EOF > user_nl_clm
    fsurdat="$MYCTSMDATA/lnd/clm2/surfdata_map/surfdata_0.25_SCA_hist_16pfts_Irrig_CMIP6_simyr2000_c190814.nc"
    EOF

If you run your simulation with the `BGC`-mode, then you need to set
another surface data map as follows:

    cat << EOF > user_nl_clm
    fsurdat="$MYCTSMDATA/lnd/clm2/surfdata_map/surfdata_0.25_SCA_hist_78pfts_CMIP6_simyr2000_c190819.nc"
    EOF

If you want to print out only selected variables with hourly resolution in
the model output files for each year:

    cat << EOF > user_nl_clm
    fsurdat="$MYCTSMDATA/lnd/clm2/surfdata_map/surfdata_0.25_SCA_hist_16pfts_Irrig_CMIP6_simyr2000_c190814.nc"
    hist_empty_htapes = .true.
    hist_fincl1 = 'TSA', 'TSKIN', 'EFLX_LH_TOT', 'FSH', 'WIND', 'TWS', 'SNOWLIQ', 'SNOWICE', 'SNOW_DEPTH', 'TSOI', 'H2OSOI'
    hist_nhtfrq = -1
    hist_mfilt = 365
    EOF

**Note that** the variable names (e.g. `TSA`, `TSKIN`,
`EFLX_LH_TOT`, etc.) are following the CESM name convention.

### Build and submit the case 
Final step is to build the case and submit the job to the queue.

    ./case.build
    ./case.submit
