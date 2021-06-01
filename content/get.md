# Get CTSM


That\'s where you should start.

NordicESMhub maintains a CTSM repository with all the configuration
files for running on machines in the Nordics. For now we support:

-   saga (sigma2, Norway)
-   fram (sigma2, Norway)

If your machine is not in the list and you would like us to support it,
please contact us.

For compatibility, load git version 2.23.0 or higher on your super
computer

    git/2.23.0-GCCcore-8.3.0

## How to get CTSM (for users)


To get the FATES EMERALD platform version, CLONE from NordicESM hub

    git clone -b release-clm5.0 https://github.com/NordicESMhub/ctsm.git ${HOME}/ctsm_fates_emerald

In this example we are checking out the release-clm5.0 tag and create a
new local branch (recommended). The destination of the checkout is a
directory (e.g. ctsm\_fates\_emerald) in our home directory.

### How to get a specific branch

Change into the created ctsm directory

    cd ${HOME}/ctsm_fates_emerald

Check all existing branches

    git branch --all

To checkout the FATES EMERALD platform (in this example release 2.0.1)
into a new local branch (e.g. new\_branch\_name)

    git checkout release-emerald-platform2.0.1 -b new_branch_name

For later reference, it is useful to choose new\_branch\_name according
to function and include the version and your username, e.g.
username\_release-emerald-platform2.0.1.

To fetch the proper externals (CIME, FATES, etc.) run

    ./manage_externals/checkout_externals

from the main ctsm directory (we are going to call this \$CTSM\_ROOT
from now on). All should be set by this and you should be able to create
your first case.


## How to get CTSM (for developers)


Dependent on which project you wish to contribute to you might want to
start your development from different versions of CTSM. For the
CLM-Norway team we have to mainly two startingpoints:

> -   The [NordicESM-hub](https://github.com/NordicESMhub/ctsm) (note
>     that this is a project for developers in the Nordics)
> -   The latest version of the original
>     [CTSM](https://github.com/ESCOMP/CTSM) (this is the original
>     version of CTSM developed by NCAR)

### From the [NordicESM-hub](https://github.com/NordicESMhub/ctsm)

Follow the steps above, but checkout the fates\_emerald\_api instead

    git checkout fates_emerald_api -b new_branch_name

For later reference, it is useful to choose new\_branch\_name according
to function and include the version and your username, e.g.
username\_fates\_emerald\_api. After checking out the externals, change
to cime directory and create your own branch to record all your changes

    cd externals/cime
    git checkout -b username_cime

Change to fates directory and create your own branch to record all your
changes

    cd externals/fates
    git checkout -b username_fates

If you do not create your own branch for \"cime\" and \"fates\", running
\"./manage\_externals/checkout\_externals\", will overwrite your
previous \"cime\" and \"fates\". You should be ready to create your
first case.

### From the latest version of [CTSM](https://github.com/ESCOMP/CTSM)

This tutorial assumes that you are logged into one of the clusters (fram
or saga) at sigma2. For access to those see (future referance to
prerequisites section).

Start from your home folder and clone CTSM from ESCOMP 

    git clone --origin escomp <https://github.com/ESCOMP/CTSM.git> CTSM

Change into the new directory

    cd CTSM

Create a local branch 

    git checkout master -b my_branch_name

For later reference, it is useful to choose `my_branch_name` according
to function and include the version and your username.

To fetch the proper externals (CIME, FATES, etc.) run

    ./manage_externals/checkout_externals

#### Porting of cime

Now you need to add machine specifics for the norwegian clusters. This
can be done in two ways (check the
[original](https://esmci.github.io/cime/versions/master/html/users_guide/porting-cime.html#steps-for-porting)
documentation for a detailed explanation):

1. You can replace some default configuration files with configuration files that contain details for these clusters.
2. You can create a [.cime]{.title-ref} folder with the machine configurations under your home diretory.


##### Replace the default configurations
Execute the following steps: 
1. Go into the cime-config folder

        cd cime/config/cesm/machines 
    
2. Delete the default files: 

        rm config_machines.xml config_batch.xml config_compilers.xml

3. Fetch replacementfiles from [this](https://github.com/gunnartl/config_files_sigma2.git) repository by typing

        git init 
        git remote add origin <https://github.com/gunnartl/config_files_sigma2.git> 
        git pull origin main

##### Create a dotcime folder
Clone [this](https://github.com/MetOs-UiO/dotcime) repository and
consult the `README.md` file for details for making a new
case.
