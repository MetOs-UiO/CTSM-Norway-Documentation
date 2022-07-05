# Quick start

This guide helps you set up your workspace and submit a case on a [Sigma2 machine](https://documentation.sigma2.no/index.html) (Fram, Saga). See the following sections for detailed instructions on how to get and set up CTSM and create and submit cases.

The guide works with ESCOMP CTSM ctsm5.1.dev098. All commands should work as they are, so you can just copy and paste them into your terminal. If there's a parameter in a command wrapped in `<` and `>`, it means it should be replaced with your own values.

## Create and run a case:

1. If you don't already have a Sigma2 account, request one here (talk to your supervisor for project name to include in the application form): https://www.metacenter.no/user/application/.
2. After your account is activated, ssh into one of the Sigma2 machines you have access to, e.g. fram:

    `ssh <your-sigma2-username>@fram.sigma2.no`.
    ```{keypoints} Note
    This command works on Linux and Mac systems.
    For access on Windows see [here](https://documentation.sigma2.no/getting_started/create_ssh_keys.html#login-via-ssh-keys).
    The link also has detailed description of ssh access for all systems.
    ```
3. Clone [MetOs dotcime repo](https://github.com/MetOs-UiO/dotcime):

    `git clone https://github.com/MetOs-UiO/dotcime ~/.cime`
4. Clone [ESCOMP/CTSM](https://github.com/escomp/ctsm) into `~/ctsm_escomp`:

    `git clone https://github.com/escomp/ctsm ~/ctsm_escomp`.
5. Go to the cloned repo:

    `cd ~/ctsm_escomp`.
6. Switch to `ctsm5.1.dev098`:

    `git checkout ctsm5.1.dev098`.
7. Get the external dependencies/repos:

    `./manage_externals/checkout_externals`.
8. Adjust and export the following variables in your workspace:

    ```bash
    export MACHINE=<machine_name> # e.g. saga or fram
    export PROJECT=<project_name> # e.g. nn2806k
    ```
9. Run the following to create a new case:

    `~/ctsm_escomp/cime/scripts/create_newcase --case ~/cases/I2000Clm50Sp --compset I2000Clm50Sp --res f19_g17 --machine $MACHINE --run-unsupported --handle-preexisting-dirs r --project $PROJECT`.
10. If everything goes fine, there should be a new folder for your case in `~/cases`. Switch to the case folder:

    `cd ~/cases/I2000Clm50Sp`.
11. Set up the case:

    `./case.setup`.
12. Build the case:

    `./case.build`.
13. Submit the case:

    `./case.submit`.
14. If there's no error in the previous steps, there should be a line towards the end of `CaseStatus` file that contains `case.submit success`. You can check it with this:

    `cat CaseStatus`.
15. If the submission is successful, your case goes in the execution queue. You can check the latest status in the same `CaseStatus` file. After a successful run, you should see messages like this:
    ```
    2022-06-17 14:00:01: case.setup starting 
    ---------------------------------------------------
    2022-06-17 14:00:07: case.setup success 
    ---------------------------------------------------
    2022-06-17 14:02:23: case.build starting 
    ---------------------------------------------------
    CESM version is ctsm5.1.dev098
    Processing externals description file : Externals.cfg
    Processing externals description file : Externals_CLM.cfg
    Processing externals description file : Externals_CISM.cfg
    Processing externals description file : .gitmodules
    Processing submodules description file : .gitmodules
    Processing externals description file : Externals_CDEPS.cfg
    Checking status of externals: clm, fates, cism, source_cism, rtm, mosart, mizuroute, ccs_config, cime, cmeps, cdeps, fox, genf90, cpl7, share, mct, parallelio, doc-builder, 
        ./ccs_config
            clean sandbox, on ccs_config_cesm0.0.36
        ./cime
            clean sandbox, on cime6.0.27
        ./components/cdeps
            clean sandbox, on cdeps0.12.41
        ./components/cdeps/fox
            clean sandbox, on 4.1.2.1
        ./components/cdeps/share/genf90
            clean sandbox, on genf90_200608
        ./components/cism
            clean sandbox, on cismwrap_2_1_95
        ./components/cism/source_cism
            clean sandbox, on cism_main_2.01.011
        ./components/cmeps
            clean sandbox, on cmeps0.13.63
        ./components/cpl7
            clean sandbox, on cpl7.0.12
        ./components/mizuRoute
            clean sandbox, on 34723c2e4df7caa16812770f8d53ebc83fa22360
        ./components/mosart
            clean sandbox, on mosart1_0_45
        ./components/rtm
            clean sandbox, on rtm1_0_78
    e-o ./doc/doc-builder
            -, not checked out --> v1.0.8
        ./libraries/mct
            clean sandbox, on MCT_2.11.0
        ./libraries/parallelio
            clean sandbox, on pio2_5_7
        ./share
            clean sandbox, on share1.0.11
        ./src/fates
            clean sandbox, on sci.1.56.0_api.23.0.0
    2022-06-17 14:14:50: case.build success 
    ---------------------------------------------------
    2022-06-17 14:16:17: case.submit starting 5979969
    ---------------------------------------------------
    2022-06-17 14:16:17: case.submit success 5979969
    ---------------------------------------------------
    ```
16. The last line means the execution was successful and the output is put in your user archive folder. You can see the output here:

    `ls /cluster/work/users/$USER/archive/I2000Clm50Sp`.
