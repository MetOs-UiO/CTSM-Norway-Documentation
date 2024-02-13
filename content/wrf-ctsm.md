# WRF-CTSM 

```{keypoints} Info
A detailed documentation](https://escomp.github.io/ctsm-docs/versions/master/html/lilac/specific-atm-models/index.html)] about how to set up and run CTSM with WRF can be found in the original. 
```

What follows here is a modified version of that workflow that works on the SIGMA2 HPC systems (As of right now Fram with some parts having to be run on Saga). 

## 3.4.1.1. Clone WRF and CTSM Repositories

    [~/HOME]$ git clone https://github.com/wrf-model/WRF.git WRF-CTSM
    [~/HOME]$ cd WRF-CSTM
    [~/WRF-CTSM]$ git checkout develop
    

    [~/WRF-CTSM]$ git clone https://github.com/NorESMhub/CTSM.git
    [~/WRF-CTSM]$ cd CTSM
    [~/WRF-CTSM/CTSM]$ git checkout ctsm5.1.dev151-noresm_v1
    [~/WRF-CTSM/CTSM]$ ./manage_externals/checkout_externals


change the content of WRF-CTS/CTSM/src/cpl/utils/lnd_import_export_utils.F90 and WRF-CTSM/phys/module_sf_mynn.F 

WRF-CTSM/phys/module_sf_mynn.F from line 1147 should look like this (that is insert the if-block in the middle):

        Q2(I)=QSFCMR(I)+(QV1D(I)-QSFCMR(I))*PSIQ2/PSIQ
        Q2(I)= MAX(Q2(I), MIN(QSFCMR(I), QV1D(I)))
        Q2(I)= MIN(Q2(I), 1.05*QV1D(I))

        IF (Q2(I) .LT. 0.0) THEN
            print*,"DEBUG: NEGATIVE Q2 VALUE IN MYNN SFCLAYER",&
            I,J, "Q2: ",Q2(I)
            print*,"WARNING: NEGATIVE Q2 SET TO ZERO"
            Q2(I)=0.0
        ENDIF
        IF (QSFC(I) .LT. 0.0) THEN
            print*,"DEBUG: NEGATIVE QSFC VALUE IN MYNN SFCLAYER",&
            I,J, "QSFC: ",QSFC(I)
        ENDIF

        IF ( debug_code ) THEN
            yesno = 0

WRF-CTS/CTSM/src/cpl/utils/lnd_import_export_utils.F90 from line 128 should look like this: 

       end if
       if ( wateratm2lndbulk_inst%forc_q_not_downscaled_grc(g) < 0.0_r8 )then
         write(iulog,*) 'Value of wateratm2 = ', wateratm2lndbulk_inst%forc_q_not_downscaled_grc(g)  
         write(iulog,*) 'Value of g = ', g
         !call shr_sys_abort( subname//&                                                                                                                                                                          
         !     ' ERROR: Bottom layer specific humidty sent from the atmosphere model is less than zero' )  
       end if
    end do

## 3.4.1.2 Building CTSM

    [~/WRF-CTSM/CTSM]$ ./lilac/build_ctsm ctsm_build_dir --compiler intel --machine fram

## 3.4.1.3. Building WRF with CTSM

    [~/WRF-CTSM/CTSM]$ source ctsm_build_dir/ctsm_build_environment.sh
    [~/WRF-CTSM/CTSM]$ export WRF_CTSM_MKFILE=/cluster/home/$USER/WRF-CTSM/CTSM/ctsm_build_dir/bld/ctsm.mk

```{discussion} NETCDF
In addition to the decribed steps, it is necessary to set the path to the Fortran NetCDF on FRAM before running `./configure`.
```
Set the path variables

    [~/WRF-CTSM/CTSM]$ export NETCDF=${EBROOTNETCDFMINFORTRAN}
    [~/WRF-CTSM/CTSM]$ export NETCDF_classic=1
    [~/WRF-CTSM/CTSM]$ cd ..
    [~/WRF-CTSM]$ ./clean -a
    [~/WRF-CTSM/CTSM]$ ./configure

When running `./configure` you will be asked for a set of compiler options [1-75]. A good choice is 16. followed by 1 as the nesting option.

```{discussion} Compiling WRF
Compiling WRF takes a long time. Therefore, you should put the compiling job into the background. If your conection to FRAM breaks the job will still run!
```
    [~/WRF-CTSM]$ nohup ./compile em_real 2>&1 > compile.log & 
    
## 3.4.1.4 Building WPS

    [~/WRF-CTSM]$ git clone https://github.com/wrf-model/WPS
    [~/WRF-CTSM]$ cd WPS
    [~/WRF-CTSM/WPS]$ git checkout v4.3
    [~/WRF-CTSM/WPS]$ export WRF_DIR=../
    [~/WRF-CTSM/WPS]$ ./configure

The automatic creation of the configuration (option 19 is a good choice) fails to recognize the compilers, therefore you have to make some changes in `configure.wps` manually.

Change 

    DM_FC= mpiifort
    DM_CC= mpiicc
    
and add `-qopenmp` to the end of the line which reads `LDFLAGS`.
Because `jasper` is one of the compiler options you have to load two additional modules beforehand
    
    [~/WRF-CTSM/WPS]$ ml JasPer/2.0.33-GCCcore-11.3.0
    [~/WRF-CTSM/WPS]$ ml libpng/1.6.37-GCCcore-11.3.0
    [~/WRF-CTSM/WPS]$ ml CMake/3.23.1-GCCcore-11.3.0
    [~/WRF-CTSM/WPS]$ ml JasPer/2.0.33-GCCcore-11.3.0
    [~/WRF-CTSM/WPS]$ ml PnetCDF/1.12.3-iimpi-2022a
    [~/WRF-CTSM/WPS]$ ./compile >& compile.log
    
## 3.4.1.5. Run WPS Programs

To create boundary conditions we use the WPS programs. 
    
## Example scripts

Reoccuring setting of environmental variables and loading of modules can be automated. We assume that you have set

`export WORK=/cluster/work/users/${USER}`

`````{tabs}

````{tab} setpaths
This example Bash script shows the environment needed to run WSP and WRF on FRAM.
You would copy it to your home directory (e.g. to a subfolder bin) and execute it

    [~] source ~/bin/setpaths

```bash
#! /bin/bash

# Location of your CTSM clone
export CTSM_ROOT=~/WRF-CTSM/CTSM/
# Location of your CTSM build
export CTSM_BUILD_DIRECTORY=~/ctsm_build_directory
# Load all modules which WRF and CTSM have in common
source $CTSM_BUILD_DIR/ctsm_build_environment.sh
# For building WRF
export WRF_ROOT=$WORKSPACE/WRF-CTSM
export WRF_CTSM_MKFILE=$CTSM_BUILD_DIR/bld/ctsm.mk
export WRF_EM_CORE=1
export WRF_DA_CORE=0
export NETCDF=${EBROOTNETCDFMINFORTRAN}
export NETCDF_classic=1
# For building WPS
export WPS_ROOT=$WORKSPACE/WPS
export WRF_DIR=$WRF_ROOT
module load JasPer/2.0.28-GCCcore-10.3.0
module load libpng/1.6.37-GCCcore-10.3.0
# For running WRF
export WRF_GEOG_PATH=/cluster/shared/wrf/geog_wrfv4

```
````

````{tab} run_wps
This example Bash script shows how to use WSP to process WRF input data (e.g. atmosphere and sea surface temperature).

```bash
#! /bin/bash
# Before running this script for the first time 
# make a new case directory in your home and
# put a copy of $WPS_ROOT/namelist.wps into it.
# Edit namelist.wps according to your domain.
# Usage: ./run_wps <case_name>
# Specify data source
data_dir=$WORKSPACE/DATA/matthew
# Reference to the case
case=${1}

case_root=${HOME}/wrf_cases
tmp_dir=(atm sst)

# Loop through the data
for each in ${tmp_dir[@]}; do
    if [ ! -d ${case_root}/${case}/${case}.${each} ]; then
        echo "Make new directory ${case_root}/${case}/${case}.${each}"
        mkdir -p ${case_root}/${case}/${case}.${each}
    fi

    cd  ${case_root}/${case}/${case}.${each}

    if [ $each == atm ]; then
        echo "Ungribing the atmosphere"
        ln -sf ${WPS_ROOT}/ungrib/Variable_Tables/Vtable.GFS Vtable
        ${WPS_ROOT}/link_grib.csh ${data_dir}/fnl
        ln -sf ${case_root}/${case}/namelist.wps.atm namelist.wps
    else 
        echo "Ungribing the the sea surface temperatures"
        ln -sf ${WPS_ROOT}/ungrib/Variable_Tables/Vtable.SST Vtable
        ${WPS_ROOT}/link_grib.csh ${data_dir}_sst/rtg_sst_grb
        ln -sf ${case_root}/${case}/namelist.wps.sst namelist.wps
    fi

    #if [ ls FILE | grep -c FILE == 0 -o ]
    ${WPS_ROOT}/ungrib.exe

    for exe in (geogrid metgrid); do
        mkdir ${exe} && ln -s ${WPS_ROOT}/${exe}/${exe^^}.TBL.ARW ${exe}/${exe^^}.TBL
        mpirun -np 2 ${WPS_ROOT}/${exe}.exe
    done
done

```
````

````{tab} submit_wrf
This example Bash script shows how to submit a WRF case to the queueing system on FRAM.


```bash
#! /bin/bash
# Script for running on Abel. (c) Johanne Rydsaa
# 2021-09-14 Update for fram. (c) Stefanie Falk
# --------------------------------------------------------------------------
# Job name:
#SBATCH --job-name=wrf_exe
#
# Project:
#SBATCH --account=nn2806k
#
# Number of cores/nodes:
#SBATCH --ntasks=4
#
# Wall clock limit:
#SBATCH --time=1:59:00
#
# Development queue for testing
#SBATCH --qos=short
#
#
## Recommended safety settings:
set -o errexit # Make bash exit on any error
#set -o nounset # Treat unset variables as errors
# Must set large stack size (unlimited for simplicity)
ulimit -s unlimited
#
# Set environment variables specifically used by WRF
source ~/bin/setpaths
# --------------------------------------------------------------------------
# Make your changes here
# WRF case name (create subcases by writing, e.g. matthew/01)
CASE=matthew
# Directory of WRF executable
WRF_EXE=${WRF_DIR}/test/em_real

#----------------------------------------------------------------------------
# No need to change this section
# Output directory
SCRATCH=${WORK}/${CASE}
# Make the work directory if not existent
if [ ! -d $SCRATCH ]; then mkdir -p $SCRATCH; fi 

## Copy files to work directory:
cp ${WRF_EXE}/ozone*            $SCRATCH
cp ${WRF_EXE}/RRTM*             $SCRATCH
cp ${WRF_EXE}/wrf.exe           $SCRATCH
## Change to work directory
cd ${SCRATCH}

## Run command
mpirun -np 4 ./wrf.exe

## Finish the script
exit 0


```
````
`````

