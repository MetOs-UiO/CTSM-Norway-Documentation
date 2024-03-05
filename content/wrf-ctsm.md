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


change the content of WRF-CTSM/CTSM/src/cpl/utils/lnd_import_export_utils.F90 and WRF-CTSM/phys/module_sf_mynn.F 

WRF-CTSM/phys/module_sf_mynn.F from line 1147 should look like this (that is insert the if-blocks in the middle):

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

WRF-CTSM/CTSM/src/cpl/utils/lnd_import_export_utils.F90 from line 128 should look like this: 

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

Edit the &share section of namelist.wps : 

    &share
     wrf_core = 'ARW',
     max_dom = 1,
     start_date = '2016-10-06_00:00:00',
     end_date   = '2016-10-08_00:00:00',
     interval_seconds = 21600
    /

Link the appropriate table to the folder, for ERA5 this is the ECMWF table. For other data google will be your friend. 

    [~/WRF-CTSM/WPS]$ ln -sf ungrib/Variable_Tables/Vtable.ECMWF Vtable

make a folder for the data you need and link the data there : 

    [~/WRF-CTSM/WPS]$ mkdir ../DATA
    [~/WRF-CTSM/WPS]$ cd ../DATA
    [~/WRF-CTSM/DATA]$ ln -sf /cluster/shared/wrf/climate/ERA5_EUR/ERA5_grib1_2016/*20161006* .
    [~/WRF-CTSM/DATA]$ ln -sf /cluster/shared/wrf/climate/ERA5_EUR/ERA5_grib1_2016/*20161007* .
    [~/WRF-CTSM/DATA]$ ln -sf /cluster/shared/wrf/climate/ERA5_EUR/ERA5_grib1_2016/*20161008* .

Link the grib data: 

    [~/WRF-CTSM/WPS]$ ./link_grib.csh ../DATA/

Rung ungrib.exe

    [~/WRF-CTSM/WPS]$ ./ungrib.exe

Edit the geogrid-setion of namelist.wps to fit your domain and , here a portion of South Norway: 

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
Link the metfiles to the run directory. 

    [~/WRF-CTSM/run]$ ln -sf ../WPS/met_em.d* .

Edit namelist.input sections time_controll and domains so i fits your times and your domain. Here: 

    &time_control
    run_days                            = 0,
    run_hours                           = 48,
    run_minutes                         = 0,
    run_seconds                         = 0,
    start_year                          = 2016,
    start_month                         = 10,   
    start_day                           = 06,  
    start_hour                          = 00,  
    end_year                            = 2016,
    end_month                           = 10,  
    end_day                             = 08,  
    end_hour                            = 00, 
    interval_seconds                    = 21600,
    input_from_file                     = .true.,
    history_interval                    = 180,
    frames_per_outfile                  = 1000,
    restart                             = .false.,
    restart_interval                    = 1449,
    io_form_history                     = 2
    io_form_restart                     = 2
    io_form_input                       = 2
    io_form_boundary                    = 2
    /

    &domains
    time_step                           = 90,
    time_step_fract_num                 = 0,
    time_step_fract_den                 = 1,
    max_dom                             = 1,
    e_we                                = 121,  
    e_sn                                = 91,   
    e_vert                              = 35,
    !dzstretch_s                         = 1.1
    p_top_requested                     = 5000,
    num_metgrid_levels                  = 38,
    num_metgrid_soil_levels             = 4,
    dx                                  = 5000,
    dy                                  = 5000,
    grid_id                             = 1,     
    parent_id                           = 0,     
    i_parent_start                      = 1,     
    j_parent_start                      = 1,     
    parent_grid_ratio                   = 1,     
    parent_time_step_ratio              = 1,   
    feedback                            = 1,
    smooth_option                       = 0
    /

run real.exe 

    [~/WRF-CTSM/run]$ mpirun -np 2 ./real.exe

Check rsl.error.0000 for the line "real_em: SUCCESS COMPLETE REAL_EM INIT" to know if the program successfully completed. 

## Creating a mapping files, domains and surface data. 


The first part of this step has do be done on saga currently as saga has the modules NCL and NCO installed. So log in  to saga and install CTSM as above. 

### Creating Mapping files

Copy geo_em.d01.nc to Saga for instance done from Saga: 

    [~]$ scp gunnartl@fram.sigma2.no:~/WRF-CTSM/WPS/geo_em.d01.nc .

cd to the contrib folder

    [~]$ cd WRF-CTSM/CTSM/tools/contrib

and edit create_scrip_file.ncl to fit the newly added geo_em file. 

    line 15ish: wrf_file  = addfile("~/geo_em.d01.nc", "r")

laod NCL and run create_scrip_file.ncl

    [~/WRF-CTSM/CTSM/tools/contrib]$ ml NCL/6.6.2-foss-2021a
    [~/WRF-CTSM/CTSM/tools/contrib]$ ncl create_scrip_file.ncl

this creates two files: wrf2clm_land.nc and wrf2clm_ocean.nc. Now cd to ../site_and_regional where the lines 74-78 are commented out in mkunitymap.ncl: 

    ;if ( any(ncb->grid_imask .ne. 1.0d00) )then
    ;   print( "ERROR: the mask of the second file isn't identically 1!" );
    ;   print( "(second file should be land grid file)");
    ;   exit
    ;end if

set the names of the inputfiles just created and the name of the outputfile: 

    [~/WRF-CTSM/CTSM/tools/contrib]$ export GRIDFILE1='/cluster/home/$USER/WRF-CTSM/CTSM/tools/contrib/wrf2clm_ocean.nc'
    [~/WRF-CTSM/CTSM/tools/contrib]$ export GRIDFILE2='/cluster/home/$USER/WRF-CTSM/CTSM/tools/contrib/wrf2clm_land.nc'
    [~/WRF-CTSM/CTSM/tools/contrib]$ export MAPFILE='wrf2clm_mapping.nc'
    [~/WRF-CTSM/CTSM/tools/contrib]$ PRINT=TRUE
    [~/WRF-CTSM/CTSM/tools/contrib]$ ncl mkunitymap.ncl


cd to ../mkmapdata and create the file regridbatch.sh that looks like this: 

    #!/bin/bash
    #
    #
    # Batch script to submit to create mapping files for all standard
    # resolutions.  If you provide a single resolution via "$RES", only
    # that resolution will be used. In that case: If it is a regional or
    # single point resolution, you should set '#PBS -n' to 1, and be sure
    # that '-t regional' is specified in cmdargs.
    #
    #SBATCH --account=nn2806k 
    #SBATCH --job-name=mkmapdata
    #SBATCH --mem-per-cpu=124G
    #SBATCH --ntasks=1
    #SBATCH --time=03:00:00

    source /cluster/bin/jobsetup
    module load ESMF/8.1.1-foss-2021a
    module load NCO/5.0.1-foss-2021a
    module load NCL/6.6.2-foss-2021a

    export ESMF_NETCDF_LIBS="-lnetcdff -lnetcdf -lnetcdf_c++"
    #export ESMF_DIR=/usit/abel/u1/huit/ESMF/esmf
    export ESMF_COMPILER=intel
    export ESMF_COMM=openmpi
    #export ESMF_NETCDF="test"
    export ESMF_NETCDF_LIBPATH=/cluster/software/ESMF/8.1.1-foss-2021a/lib
    export ESMF_NETCDF_INCLUDE=/cluster/software/ESMF/8.1.1-foss-2021a/include
    ulimit -s unlimited
    
    export ESMFBIN_PATH=/cluster/software/ESMF/8.1.1-foss-2021a/bin
    export CSMDATA=/cluster/shared/noresm/inputdata
    export MPIEXEC=mpirun

    RES=1x1
    GRIDFILE=/cluster/home/gunnartl/WRF-CTSM/CTSM/tools/contrib/wrf2clm_land.nc
    phys="clm4_5"

    #----------------------------------------------------------------------
    # Set parameters
    #----------------------------------------------------------------------

    #----------------------------------------------------------------------
    # Begin main script
    #----------------------------------------------------------------------

    if [ -z "$RES" ]; then
    echo "Run for all valid resolutions"
    resols=`../../bld/queryDefaultNamelist.pl -res list -silent`
    if [ ! -z "$GRIDFILE" ]; then
        echo "When GRIDFILE set RES also needs to be set for a single resolution"
        exit 1
    fi
    else
    resols="$RES"
    fi
    if [ -z "$GRIDFILE" ]; then
    grid=""
    else
    if [[ ${#resols[@]} > 1 ]]; then
        echo "When GRIDFILE is specificed only one resolution can also be given (# resolutions ${#resols[@]})"
        echo "Resolutions input is: $resols"
        exit 1
    fi
    grid="-f $GRIDFILE"
    fi

    if [ -z "$MKMAPDATA_OPTIONS" ]; then
    echo "Run with standard options"
    options=" "
    else
    options="$MKMAPDATA_OPTIONS"
    fi
    echo "Create mapping files for this list of resolutions: $resols"

    #----------------------------------------------------------------------

    for res in $resols; do
    echo "Create mapping files for: $res"
    #----------------------------------------------------------------------
    cmdargs="-r $res $grid $options"

    # For single-point and regional resolutions, tell mkmapdata that
    # output type is regional
    if [[ `echo "$res" | grep -c "1x1"` -gt 0 || `echo "$res" | grep -c "5x5_"` -gt 0 ]]; then
        res_type="regional"
    else
        res_type="global"
    fi
    # Assume if you are providing a gridfile that the grid is regional
    if [ $grid != "" ];then
        res_type="regional"
    fi

    cmdargs="$cmdargs -t $res_type"

    echo "$res_type"
    if [ "$res_type" = "regional" ]; then
        echo "regional"
        # For regional and (especially) single-point grids, we can get
        # errors when trying to use multiple processors - so just use 1.
        # We also do NOT set batch mode in this case, because some
        # machines (e.g., yellowstone) do not listen to REGRID_PROC, so to
        # get a single processor, we need to run mkmapdata.sh in
        # interactive mode.
        regrid_num_proc=1
    else
        echo "global"
        regrid_num_proc=8
        if [ ! -z "$LSFUSER" ]; then
            echo "batch"
        cmdargs="$cmdargs -b"
        fi
        if [ ! -z "$PBS_O_WORKDIR" ]; then
            cd $PBS_O_WORKDIR
        cmdargs="$cmdargs -b"
        fi
    fi

    echo "args: $cmdargs"
    echo "time env REGRID_PROC=$regrid_num_proc ./mkmapdata.sh $cmdargs\n"
    time env REGRID_PROC=$regrid_num_proc ./mkmapdata.sh $cmdargs
    done

Submit the job : 

    [~/WRF-CTSM/CTSM/tools/mkmapdata]$ sbatch regridbatch.sh

This will take some hours depending on the size and resolution of your domain you might have to change the available memory per cpu and allocated time and such. 
ie. : 

#SBATCH --mem-per-cpu=128G --partition=bigmem                                                                                                                                                                      
#SBATCH --ntasks=8                                                                                                                                                                                                 
#SBATCH --time=15:00:00   

When this is done you should copy the resulting files back to fram and place them in the corresponding directories there, and the rest is fiished on Fram. 

### Generating domains: 

Create a file in ~/WRF-CTSM/CTSM/cime/tools named configure that contains this: 

    #!/usr/bin/env python3

    """This script writes CIME build information to a directory.

    The pieces of information that will be written include:

    1. Machine-specific build settings (i.e. the "Macros" file).
    2. File-specific build settings (i.e. "Depends" files).
    3. Environment variable loads (i.e. the env_mach_specific files).

    The .env_mach_specific.sh and .env_mach_specific.csh files are specific to a
    given compiler, MPI library, and DEBUG setting. By default, these will be the
    machine's default compiler, the machine's default MPI library, and FALSE,
    respectively. These can be changed by setting the environment variables
    COMPILER, MPILIB, and DEBUG, respectively.
    """

    # pylint: disable=W1505

    import os
    import sys

    real_file_dir = os.path.dirname(os.path.realpath(__file__))
    cimeroot = os.path.abspath(os.path.join(real_file_dir, ".."))
    sys.path.insert(0, cimeroot)

    from CIME.Tools.standard_script_setup import *
    from CIME.utils import expect, get_model
    from CIME.BuildTools.configure import configure
    from CIME.XML.machines import Machines

    logger = logging.getLogger(__name__)


    def parse_command_line(args):
        """Command line argument parser for configure."""
        description = __doc__
        parser = argparse.ArgumentParser(description=description)
        CIME.utils.setup_standard_logging_options(parser)

        parser.add_argument(
            "--machine", help="The machine to create build information for."
        )
        parser.add_argument(
            "--machines-dir",
            help="The machines directory to take build information "
            "from. Overrides the CIME_MODEL environment variable, "
            "and must be specified if that variable is not set.",
        )
        parser.add_argument(
            "--macros-format",
            action="append",
            choices=["Makefile", "CMake"],
            help="The format of Macros file to generate. If "
            "'Makefile' is passed in, a file called 'Macros.make' "
            "is generated. If 'CMake' is passed in, a file called "
            "'Macros.cmake' is generated. This option can be "
            "specified multiple times to generate multiple files. "
            "If not used at all, Macros generation is skipped. "
            "Note that Depends files are currently always in "
            "Makefile format, regardless of this option.",
        )
        parser.add_argument(
            "--output-dir",
            default=os.getcwd(),
            help="The directory to write files to. If not "
            "specified, defaults to the current working directory.",
        )

        parser.add_argument(
            "--compiler",
            "-compiler",
            help="Specify a compiler. "
            "To see list of supported compilers for each machine, use the utility query_config in this directory",
        )

        parser.add_argument(
            "--mpilib",
            "-mpilib",
            help="Specify the mpilib. "
            "To see list of supported mpilibs for each machine, use the utility query_config in this directory. "
            "The default is the first listing in MPILIBS in config_machines.xml",
        )

        parser.add_argument(
            "--clean",
            action="store_true",
            help="Remove old Macros and env files before attempting to create new ones",
        )

        parser.add_argument(
            "--comp-interface",
            default="mct",
            help="""The cime driver/cpl interface to use.""",
        )

        argcnt = len(args)
        args = parser.parse_args()
        CIME.utils.parse_args_and_handle_standard_logging_options(args)

        opts = {}
        if args.machines_dir is not None:
            machines_file = os.path.join(args.machines_dir, "config_machines.xml")
            machobj = Machines(infile=machines_file, machine=args.machine)
        else:
            model = get_model()
            if model is not None:
                machobj = Machines(machine=args.machine)
            else:
                expect(
                    False,
                    "Either --mach-dir or the CIME_MODEL environment "
                    "variable must be specified!",
                )

        opts["machobj"] = machobj

        if args.macros_format is None:
            opts["macros_format"] = []
        else:
            opts["macros_format"] = args.macros_format

        expect(
            os.path.isdir(args.output_dir),
            "Output directory '%s' does not exist." % args.output_dir,
        )

        opts["output_dir"] = args.output_dir

        # Set compiler.
        if args.compiler is not None:
            compiler = args.compiler
        elif "COMPILER" in os.environ:
            compiler = os.environ["COMPILER"]
        else:
            compiler = machobj.get_default_compiler()
            os.environ["COMPILER"] = compiler
        expect(
            opts["machobj"].is_valid_compiler(compiler),
            "Invalid compiler vendor given in COMPILER environment variable: %s" % compiler,
        )
        opts["compiler"] = compiler
        opts["os"] = machobj.get_value("OS")
        opts["comp_interface"] = args.comp_interface

        if args.clean:
            files = [
                "Macros.make",
                "Macros.cmake",
                "env_mach_specific.xml",
                ".env_mach_specific.sh",
                ".env_mach_specific.csh",
                "Depends.%s" % compiler,
                "Depends.%s" % args.machine,
                "Depends.%s.%s" % (args.machine, compiler),
            ]
            for file_ in files:
                if os.path.isfile(file_):
                    logger.warn("Removing file %s" % file_)
                    os.remove(file_)
            if argcnt == 2:
                opts["clean_only"] = True
                return opts

        # Set MPI library.
        if args.mpilib is not None:
            mpilib = args.mpilib
        elif "MPILIB" in os.environ:
            mpilib = os.environ["MPILIB"]
        else:
            mpilib = machobj.get_default_MPIlib(attributes={"compiler": compiler})
            os.environ["MPILIB"] = mpilib

        expect(
            opts["machobj"].is_valid_MPIlib(mpilib, attributes={"compiler": compiler}),
            "Invalid MPI library name given in MPILIB environment variable: %s" % mpilib,
        )
        opts["mpilib"] = mpilib

        # Set DEBUG flag.
        if "DEBUG" in os.environ:
            expect(
                os.environ["DEBUG"].lower() in ("true", "false"),
                "Invalid DEBUG environment variable value (must be 'TRUE' or "
                "'FALSE'): %s" % os.environ["DEBUG"],
            )
            debug = os.environ["DEBUG"].lower() == "true"
        else:
            debug = False
            os.environ["DEBUG"] = "FALSE"
        opts["debug"] = debug

        return opts


    def _main():
        opts = parse_command_line(sys.argv)
        if "clean_only" not in opts or not opts["clean_only"]:
            configure(
                opts["machobj"],
                opts["output_dir"],
                opts["macros_format"],
                opts["compiler"],
                opts["mpilib"],
                opts["debug"],
                opts["comp_interface"],
                opts["os"],
            )


    if __name__ == "__main__":
        _main()

Give the file permission s to execute: 

    [~/WRF-CTSM/CTSM/cime/tools]$ chmod +x configure

go to ~/WRF-CTSM/CTSM/cime/tools/mapping/gen_domain_files/src and run it from there

    [~/WRF-CTSM/CTSM/cime/tools/mapping/gen_domain_files/src]$ ../../../configure --machine fram --compiler intel --mpilib impi --macros-format Makefile

    [~/WRF-CTSM/CTSM/cime/tools/mapping/gen_domain_files/src]$ . ./.env_mach_specific.sh ; make

move one folder up and renerate the domain with the mapping file as input : 

    [~/WRF-CTSM/CTSM/cime/tools/mapping/gen_domain_files/src]$ cd ..
    [~/WRF-CTSM/CTSM/cime/tools/mapping/gen_domain_files]$ ./gen_domain -m ~/WRF-CTSM/CTSM/tools/mkmapdata/wrf2clm_mapping.nc -o wrf2clm_ocn_noneg -l wrf2clm_lnd_noneg

### Generating Surface data

cd to ~/WRF-CTSM/CTSM/tools/mksurfdata_map/src and edit Makefile.common change lines 33 and 37: sp they say;  

    LIB_NETCDF := /cluster/software/netCDF-Fortran/4.6.0-iimpi-2022a/lib
    INC_NETCDF := /cluster/software/netCDF-Fortran/4.6.0-iimpi-2022a/include

laod NETcdf module and make: 

    [~/wrf-ctsm/CTSM/tools/mksurfdata_map/src]$ ml netCDF-Fortran/4.6.0-iimpi-2022a
    [~/wrf-ctsm/CTSM/tools/mksurfdata_map/src]$ gmake
    [~/wrf-ctsm/CTSM/tools/mksurfdata_map/src]$ cd ..

edit mksurfdata.pl line 830 so that it says (just remove the underscore):

    if (index($res, '1x1') != -1) {

and run mksurfdata.pl to create surface files:

    ./mksurfdata.pl -res usrspec -usr_gname "1x1" -usr_gdate "240208" -usr_mapdir "/cluster/home/$USER/WRF-CTSM/CTSM/tools/mkmapdata" -y 2000 -exedir "/cluster/home/$USER/WRF-CTSM/CTSM/tools/mksurfdata_map" -no-crop -dinlc /cluster/shared/noresm/inputdata

"1x1" denotes the regional run, "240208" should be changed according to the date when mapping files resulting from ./regridbatch.sh were created which can be checked in their .nc names under /cluster/home/$USER/WRF-CTSM/CTSM/tools/mkmapdata, "-y 2000" refers to the present day year, "-dinlc" denotes the location of the rawdata

