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
In addition to the described steps, it is necessary to set the path to the Fortran NetCDF on FRAM before running `./configure`.
```
Set the path variables

    [~/WRF-CTSM/CTSM]$ export NETCDF=${EBROOTNETCDFMINFORTRAN}
    [~/WRF-CTSM/CTSM]$ export NETCDF_classic=1
    [~/WRF-CTSM/CTSM]$ cd ..
    [~/WRF-CTSM]$ ./clean -a
    [~/WRF-CTSM/CTSM]$ ./configure

When running `./configure` you will be asked for a set of compiler options [1-75]. A good choice is 16. followed by 1 as the nesting option.

```{discussion} Compiling WRF
Compiling WRF takes a long time. Therefore, you should put the compiling job into the background. If your connection to FRAM breaks the job will still run!
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
    [~/WRF-CTSM/WPS]$ ml PnetCDF/1.12.3-iimpi-2022a
    [~/WRF-CTSM/WPS]$ ./compile >& compile.log

If ungrib.exe is not created just do a module purge and load everything except libpng and compile again. 
    
## 3.4.1.5. Run WPS Programs

To create boundary conditions use the WPS programs. This is an example using ERA5 data for a 48 hour run from from October 6th-8th 2016. 

Edit namelist.wps : 

    &share
    wrf_core = 'ARW',
    max_dom = 1,
    start_date = '2016-10-06_00:00:00',
    end_date   = '2016-10-08_00:00:00',
    interval_seconds = 21600
    /

Link the apropriate table to the folder, for ERA5 this is the ECMWF table. For other data google will be your friend. 

    [~/WRF-CTSM/WPS]$ ln -sf ungrib/Variable_Tables/Vtable.ECMWF Vtable

make a folder for the data you need and link the data there : 

    [~/WRF-CTSM/WPS]$ mkdir DATA
    [~/WRF-CTSM/WPS]$ cd DATA
    [~/WRF-CTSM/WPS/DATA]$ ln /cluster/shared/wrf/climate/ERA5_EUR/ERA5_grib1_2016/*20161006* .
    [~/WRF-CTSM/WPS/DATA]$ ln /cluster/shared/wrf/climate/ERA5_EUR/ERA5_grib1_2016/*20161007* .
    [~/WRF-CTSM/WPS/DATA]$ ln /cluster/shared/wrf/climate/ERA5_EUR/ERA5_grib1_2016/*20161008* .

Link the grib data: 

    [~/WRF-CTSM/WPS]$ ./link_grib.csh DATA

Rung ungrib.exe

    [~/WRF-CTSM/WPS]$ ./ungrib.exe

Edit namelist.wps again to fit your domain and , here a portion of South Norway: 

    &geogrid
    parent_id         =   1,
    parent_grid_ratio =   1, 
    i_parent_start    =   1,
    j_parent_start    =   1,
    e_we              =   121,
    e_sn              =   91,
    geog_data_res = 'default',
    dx = 5000,
    dy = 5000,
    map_proj = 'mercator',
    ref_lat   =  61.6,
    ref_lon   =   9.4,
    truelat1  =  30.0,
    truelat2  =  60.0,
    stand_lon =   0,
    geog_data_path = '/cluster/shared/wrf/geog'
    /

Run Geogrid and metgrid

    [~/WRF-CTSM/WPS]$ mpirun -np 2 ./geogrid.exe
    [~/WRF-CTSM/WPS]$ mpirun -np 2 ./metgrid.exe

## 3.4.1.6. Run real.exe

