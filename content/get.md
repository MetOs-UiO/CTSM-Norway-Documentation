# Get CTSM
Dependent on what you want to do you might want to use different strategies for starting your CTSM work at the right place. 

```{keypoints} Path variables
Throughout all this documentation we define some path variables:  
- `HOME`; your home directory on the computer you are currently working on (remote or local). Typically predefined on a Unix system.
- `CTSM_ROOT`; the top folder or wherever you cloned CTSM 
```


## Supported machines
This tutorial assumes that you are logged into one of the clusters of Sigma2. (See [here](https://metos-uio.github.io/CTSM-Norway-Documentation/prerequisite/#needed-accesses) for how to get cluster access). 

Currently (June 2021), we support machine configurations for: 
-   saga (sigma2, Norway)
-   fram (sigma2, Norway)

If your machine is not on the list and you would like us to support it,
please contact us.

## How to get CTSM (for users)
A user is a person that runs CTSM without modifying the source code. To get the FATES EMERALD platform version, CLONE from NordicESM hub

    [~/HOME]$ git clone -b release-clm5.0 https://github.com/NordicESMhub/ctsm.git ${HOME}/ctsm_fates_emerald

In this example, we are checking out the release-clm5.0 tag and create a
new local branch (recommended). The destination of the `checkout` is a
directory (e.g. ctsm\_fates\_emerald) in our home directory. From now on, we will call this directory `CTSM_ROOT`.

### How to get a specific branch
Change into the created ctsm directory

    [~/HOME]$ cd ctsm_fates_emerald

Check all existing branches

    [~/CTSM_ROOT]$ git branch --all

To check out the FATES EMERALD platform (in this example release 2.0.1)
into a new local branch (e.g. new\_branch\_name)

    [~/CTSM_ROOT]$ git checkout release-emerald-platform2.0.1 -b new_branch_name

For later reference, it is useful to choose new\_branch\_name according
to function and include the version and your username, e.g.
username\_release-emerald-platform2.0.1.

Some components of CTSM are maintained and developed independently of the main model. We call these externals.
To fetch the proper externals (CIME, FATES, etc.) run the below command from the main CTSM directory.

    [~/CTSM_ROOT]$ ./manage_externals/checkout_externals

All should be set by this and you should be able to create your first case.


## How to get CTSM (for developers)
A developer is someone that wishes to change the source code of CTSM. 
Depending on which project you wish to contribute to you might want to
start your development from different versions of CTSM. For the
CLM-Norway team we have mainly two starting points:

-   The [NordicESM-hub](https://github.com/NordicESMhub/ctsm) (note
    that this is a project for developers in the Nordics)
-   The latest version of the original
    [CTSM](https://github.com/ESCOMP/CTSM) (this is the original
    version of CTSM developed by NCAR)

### From the [NordicESM-hub](https://github.com/NordicESMhub/ctsm)

Follow the steps [above](https://metos-uio.github.io/CTSM-Norway-Documentation/get/#how-to-get-ctsm-for-users), but `checkout` the fates\_emerald\_api instead

    [~/CTSM_ROOT]$ git checkout fates_emerald_api -b new_branch_name

For later reference, it is useful to choose new\_branch\_name according
to function and include the version and your username, e.g.
username\_fates\_emerald\_api. After checking out the externals, change
to cime directory and create your own branch to record all your changes

    [~/CTSM_ROOT]$ cd externals/cime
    [~/CTSM_ROOT/externals/cime]$ git checkout -b username_cime

Change to fates directory and create your own branch to record all your
changes

    [~/CTSM_ROOT/externals/cime]$ cd ../externals/fates
    [~/CTSM_ROOT/externals/fates]$ git checkout -b username_fates

If you do not create your own branch for `cime` and `fates`, running
`./manage_externals/checkout_externals`, will overwrite your
previous `cime` and `fates`. You should now be ready to create your
first case.

### From the latest version of [CTSM](https://github.com/ESCOMP/CTSM)
Start from your home folder and clone CTSM from ESCOMP 

    [~/HOME]$ git clone --origin escomp <https://github.com/ESCOMP/CTSM.git> CTSM

Change into the new directory

    [~/HOME]$ cd CTSM

Create a local branch

    [~/CTSM_ROOT]$ git checkout master -b my_branch_name

For later reference, it is useful to choose `my_branch_name` according
to function and include the version and your username.

To fetch the proper externals (CIME, FATES, etc.) run

    [~/CTSM_ROOT]$ ./manage_externals/checkout_externals

#### Porting of cime

Now you need to add machine specifics for the Norwegian clusters. This
can be done in two ways (check the
[original](https://esmci.github.io/cime/versions/master/html/users_guide/porting-cime.html#steps-for-porting)
documentation for a detailed explanation):

1. You can replace some default configuration files with configuration files that contain details for these clusters.
2. You can create a **.cime** folder with the machine configurations under your home directory.


##### Replace the default configurations
Execute the following steps:
1. Go into the cime-config folder

        [~/CTSM_ROOT]$ cd cime/config/cesm/machines

2. Delete the default files:

        [~/CTSM_ROOT/cime/config/cesm/machine]$ rm config_machines.xml config_batch.xml config_compilers.xml

3. Fetch replacementfiles from [this](https://github.com/gunnartl/config_files_sigma2.git) repository by typing

        [~/CTSM_ROOT/cime/config/cesm/machine]$ git init
        [~/CTSM_ROOT/cime/config/cesm/machine]$ git remote add origin <https://github.com/gunnartl/config_files_sigma2.git>
        [~/CTSM_ROOT/cime/config/cesm/machine]$ git pull origin main

##### Create a dotcime folder
Clone [this](https://github.com/MetOs-UiO/dotcime) repository and
consult the `README.md` file for the details when making a new
case.
