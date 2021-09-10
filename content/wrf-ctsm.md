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
  
Set the environmental variable

    [~/HOME]$ export ESMFMKFILE=$EBROOTESMF/lib/esmf.mk

Clear all loaded modules

    [~/HOME]$ module purge
    
