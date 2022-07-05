# Single Point Quick Start
The instructions below are tested with `ctsm5.1.dev098`, and ran on `fram` machine.
Change the machine name accordingly.
The instructions assume CTSM repo is cloned in `~/ctsm_escomp`.

    ```{keypoints} Note
    Check out [this tutorial](https://ncar.github.io/CTSM-Tutorial-2022/notebooks/Day2a_GenericSinglePoint.html) by NCAR for more details.
    Note that the tutorial is written for NCAR's machines.
    ```
## Create data for a point

1. If you haven't set up CTSM yet, check out steps 1 to 8 on the [quick start guide](/CTSM-Norway-Documentation/quick-start/).

2. Load the required modules:
    If any of the modules in the following list is not available, search for the module name with `module spider <module_name>` and use the latest available version instead.

    ```bash
    module load git/2.33.1-GCCcore-11.2.0-nodocs
    module load Anaconda3/2019.07
    ```

3. Set up and Install dependencies for `subset_data`:
    ```bash    
    eval "$(/cluster/software/Anaconda3/2019.07/bin/conda shell.bash hook)"

    conda create --name subset_data python=3.9 xarray netcdf4 -y
    conda activate subset_data
    ```

4. Update the input data paths for `./subset_data`:
    
    ```bash
    sed -i 's/glade\/p\/cesmdata\/inputdata/cluster\/shared\/noresm\/inputdata/' ~/ctsm_escomp/tools/site_and_regional/default_data.cfg
    sed -i 's/glade\/p\/cgd\/tss\/CTSM_datm_forcing_data/cluster\/shared\/noresm\/inputdata\/atm\/datm7/' ~/ctsm_escomp/tools/site_and_regional/default_data.cfg
    ```

5. Generate input data for a single point:
This should generate all the data for the given latitude and longitude and period in ~/finse.
    
    ```bash
    ~/ctsm_escomp/tools/site_and_regional/subset_data \
        point \
        --site finse \
        --lat 60.59383774 \
        --lon 7.527008533 \
        --create-user-mods \
        --create-domain \
        --create-surface \
        --create-landuse \
        --create-datm \
        --datm-syr 1970 \
        --datm-eyr 1972 \
        --dompft 7 \
        --outdir ~/finse \
        --overwrite
    ```

6. Run a case for the given point:

    ```bash
    ~/ctsm_escomp/cime/scripts/create_newcase \
        --case ~/cases/finse \
        --compset I2000Clm51BgcCrop \
        --res CLM_USRDAT \
        --machine fram \
        --run-unsupported \
        --user-mods-dirs ~/finse/user_mods \    # This contains the generated namelists and a shell command that CTSM uses to configure your case with the given input data.
        --handle-preexisting-dirs r \
        --project <project_name> \ # e.g. nn2806k

    cd ~/cases/finse

    ./xmlchange MPILIB=impi
    ./xmlchange STOP_OPTION=nyears
    ./xmlchange STOP_N=1
    ./xmlchange RUN_STARTDATE='2001-01-01'
    ./xmlchange DATM_YR_ALIGN=2001
    ./xmlchange DATM_YR_START=2001
    ./xmlchange DATM_YR_END=2002

    ./case.setup
    ./case.build
    ./case.submit
    ```
