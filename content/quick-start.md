# Quick start

This guide helps you set up your workspace and submit a case on FRAM machine. See the following sections for more details of how to get CTSM and create and submit cases.

The guide works with ESCOMP CTSM v5.0.34. All commands should work as they are, so you can just copy and paste them into your terminal. If there's a parameter in a command wrapped in `<` and `>`, it means it should be replaced with your own values.

## Create and run a case on FRAM:

- If you don't already have a Sigma2 account, request one here (talk to your supervisor for project name to include in the application form): https://www.metacenter.no/user/application/.
- After your account is activated, ssh into FRAM:

    `ssh <your-sigma2-username>@fram.sigma2.no`.
    ```{keypoints} Note
    This command works on Linux and Mac systems.
    For access on Windows see [here]((https://documentation.sigma2.no/getting_started/create_ssh_keys.html#login-via-ssh-keys)).
    The link also has detailed description of ssh access for all systems.
    ```
- [*Optional*] Create a Workspace folder in your home directory and switch to it:

    `mkdir ~/Workspace && cd ~/Workspace`.
    ```{keypoints} Note
    This step is optional, but the guide assumes you have created this folder and are working in it.
    If you skip this, make sure to adjust any path that mentions `Workspace`.
    ```
- Clone [ESCOMP/CTSM](https://github.com/escomp/ctsm) into `~/Workspace/ctsm_escomp`:

    `git clone https://github.com/escomp/ctsm ~/Workspace/ctsm_escomp`.
- Go to the cloned repo:

    `cd ~/Workspace/ctsm_escomp`.
- Switch to `release-clm.5.0.34`:

    `git fetch --all && git checkout release-clm.5.0.34`.
- Edit `Externals.cfg` in the repo root and update the cime config section so it uses v5.6.10 by NorESMhub:

    ```bash
    sed -i 's/ESMCI/NorESMhub/' Externals.cfg
    sed -i 's/cime5.6.33/cime5.6.10_cesm2_1_rel_06-Nor-dev/' Externals.cfg
    ```

    These commands should replace the cime config. Check the content of `Externals.cfg` with `cat Externals.cfg` to make sure cime config looks like:

    ``` 
    [cime] 
    local_path = cime 
    protocol = git 
    repo_url = https://github.com/NorESMhub/cime 
    tag = cime5.6.10_cesm2_1_rel_06-Nor-dev 
    required = True 
    ``` 
- Get the external dependencies/repos:

    `./manage_externals/checkout_externals`.
- Adjust and export the following variables in your workspace:

    ```bash
    export CESM_ACCOUNT=nn2806k
    export PROJECT=nn2806k
    export CTSM_ROOT=/cluster/home/$USER/Workspace/ctsm_escomp
    export CESM_DATA=/cluster/shared/noresm/inputdata
    ```
- Go to `cime/scripts/`:
    `cd cime/script`
- Run the following to create a new case:

    `./create_newcase --case ~/Workspace/cases/I2000Clm50Sp --compset I2000Clm50Sp --res f19_g17 --machine fram --run-unsupported --project $CESM_ACCOUNT`.
- If everything goes fine, there should be a new folder for your case in `~/Ẁorkspace/cases`. Switch to the case folder:

    `cd ~/Workspace/cases/I2000Clm50Sp`.
- Check that `DIN_LOC_ROOT_CLMFORC` is set to `/cluster/shared/noresm/inputdata/atm/datm7`:

    `./xmlquery DIN_LOC_ROOT_CLMFORC`.
- If the previous step returns `UNSET`, you can set `DIN_LOC_ROOT_CLMFORC` with `xmlchange`:

    `./xmlchange DIN_LOC_ROOT_CLMFORC=/cluster/shared/noresm/inputdata/atm/datm7`.
- Set up the case:

    `./case.setup`.
- Build the case:

    `./case.build`.
- Submit the case:

    `./case.submit`.
- If there's no error in the previous steps, there should be a line towards the end of `CaseStatus` file that contains `case.submit success`. You can check it with this:

    `cat CaseStatus`.
- If the submission is successful, your case goes in the execution queue. You can check the latest status in the same `CaseStatus` file. After a successful run, you should see messages like this:
    ```
    2022-01-25 12:00:22: case.submit starting 
    ---------------------------------------------------
    2022-01-25 12:00:31: case.submit success case.run:4020255, case.st_archive:4020256
    ---------------------------------------------------
    2022-01-25 12:00:32: case.run starting 
    ---------------------------------------------------
    2022-01-25 12:00:37: model execution starting 
    ---------------------------------------------------
    2022-01-25 12:01:31: model execution success 
    ---------------------------------------------------
    2022-01-25 12:01:31: case.run success 
    ---------------------------------------------------
    2022-01-25 12:01:34: st_archive starting 
    ---------------------------------------------------
    2022-01-25 12:01:45: st_archive success 
    ---------------------------------------------------
    ```
- The last line means the execution was successful and the output is put in your user archive folder. You can see the output here:

    `ls /cluster/work/users/$USER/archive/I2000Clm50Sp`.
