# WRF-CTSM 

```{keypoints} Info
A detailed documentation](https://escomp.github.io/ctsm-docs/versions/master/html/lilac/specific-atm-models/index.html)] about how to set up and run CTSM with WRF can be found in the original. 
```

Please follow the steps described there!
Special advices for the supported HPC machines at UiO are given in the following.

```{discussion} Build directory
- It is good practice to NOT build your software in the source directory.
- Choose a build directory, e.g. in your home or project directory
```
    
    [~/HOME]$ export CTSM_BUILD_DIRECTORY=~/ctsm_build_directory
    

## 3.4.1.2 Building CTSM

```{discussion} ESMF
A 3. party software (ESMF) is needed for coupling to WRF. 
- The version required (currently >= 8.1.0, 2018-09-10) is only available on FRAM.
- Update or clone `dotcime` to include ESMF (commit 092d9f9).
```
For building CTSM with LILAC the path variable `ESMFMKFILE` has to be set in `config_machines`. 

## 3.4.1.3. Building WRF with CTSM

```{discussion} NETCDF
In addition to the decribed steps, it is necessary to set the path to the Fortran NetCDF on FRAM before running `./configure`.
```
Set the path variables

    [~/HOME]$ export NETCDF=${EBROOTNETCDFMINFORTRAN}
    [~/HOME]$ export NETCDF_classic=1
    
When running `./configure` you will be asked for a set of compiler options [1-75]. A good choice is 15.

```{discussion} Compiling WRF
Compiling WRF takes a long time. Therefore, you should put the compiling job into the background. If your conection to FRAM breaks the job will still run!
```
    [~/WRF-CTSM]$ nohup ./compile em_real 2>&1 > compile.log & 
    
## 3.4.1.4 Building WPS

The automatic creation of the configuration (option 19 is a good choice) fails to recognize the compilers, therefore you have to make some changes in `configure.wps` manually.

    [~/WRF-CTSM/WPS] nano configure.wps

Change 

    `DM_FC= mpiifort`
    `DM_CC= mpiicc`
    
and add `-gopenmp` to the end of the line which reads `LDFLAGS`.
Because `jasper` is one of the compiler options you have to load two additional modules beforehand
    
    [~/WRF-CTSM/WPS] module load JasPer/2.0.28-GCCcore-10.3.0
    [~/WRF-CTSM/WPS] module load libpng/1.6.37-GCCcore-10.3.0
    
