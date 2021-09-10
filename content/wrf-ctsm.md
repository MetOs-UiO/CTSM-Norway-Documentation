# WRF-CTSM 

```{keypoints} Info
A detailed documentation](https://escomp.github.io/ctsm-docs/versions/master/html/lilac/specific-atm-models/index.html)] about how to set up and run CTSM with WRF can be found in the original. 
```

Please follow the steps described there! 
In the following, special advices for the supported HPC machines at UiO are given.

## 3.4.1.2 Building CTSM

```{discussion} ESMF
A 3. party software (ESMF) is needed for coupling to WRF. 
- The version required (currently >= 8.1.0, 2018-09-10) is only available on FRAM.
- Update your dotcime to include ESMF ([experimental, checkout branch wrf_ctsm](https://github.com/ziu1986/dotcime)).
- Building CTSM with LILAC requires the path variable `ESMFMKFILE` to be set beforehand.
```
To set `ESMFMKFILE`. Login to fram and

    [~/HOME]$ load module ESMF/same_version_as_in_config_compilers
  
Set the path variable

    [~/HOME]$ export ESMFMKFILE=$EBROOTESMF/lib/esmf.mk

Clear all loaded modules

    [~/HOME]$ module purge
    
## 3.4.1.3. Building WRF with CTSM

```{discussion} NETCDF
In addition to the decribed steps, it is necessary to set the path to the Fortran NetCDF on FRAM before running `./configure`.
```
Set the path variables

    [~/HOME]$ export NETCDF=${EBROOTNETCDFMINFORTRAN}
    [~/HOME]$ export NETCDF_classic=1    

```{discussion} Compiling WRF
Compiling WRF takes a long time. Therefore, you should put the compiling job into the background. If your conection to FRAM breaks the job will still run!
```
    [~/WRF-CTSM]$ nohup ./compile em_real 2>&1 > compile.log & 
    
