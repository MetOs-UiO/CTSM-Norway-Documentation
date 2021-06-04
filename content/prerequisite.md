# Prerequisites
This page is where you should start. You will have problems with following the steps in the documentation if some of these concepts are unfamiliar to you or you haven't been granted all the needed access. 


## Technical concepts
CTSM is a big code written in modern Fortran. This requires its users to have high technical competence. 

### Git
To start working with CTSM you need some basic understanding of the version control system `git` and the web interface `GitHub`. 
```{discussion} Not familiar with git?  
  We recommend you to take a look at CodeRefinery's [Git intro](https://coderefinery.github.io/git-intro/), [Git collaborative](https://coderefinery.github.io/git-collaborative/), and [GitHub](https://coderefinery.github.io/github-without-command-line/) lessons before you start working with CTSM.
```
 
### The Unix shell
To start working with CTSM you need some basic understanding of how to work with the terminal. You also recommend you to set up [ssh-keys](https://documentation.sigma2.no/getting_started/create_ssh_keys.html) for login. 
```{discussion} Not familiar with a terminal?  
  We recommend taking a look at Software Carpentry's [unix shell](https://swcarpentry.github.io/shell-novice/) episode. 
```

## Needed accesses  
This section is a guide to the setup and use of CTSM on the Norwegian clusters
run by Sigma2. 

### The Sigma2-clusters
To get access to these you need to create an account by
filling out [this](https://www.metacenter.no/user/application/form/notur/) form. You will
need a project code that you can ask your project manager( ie.
supervisor) for. Once this form is filled out the project manager and
sigma2 will need to approve the application. This will take some days,
and you\'re good to go.

```{discussion} First time on a HPC-cluster? 
  If this is your first time using a remote HPC-system, or you want to
  know more about Sigma2\'s set up this
  [tutorial](https://sabryr.github.io/hpc-intro/12-cluster/index.html) is
  a great place to start.
```

### Input data folders
All the runs of CTSM require certain input data for forcing data, grids, etc. Usually, these files are downloaded automatically from [NCAR's service](https://escomp.github.io/CESM/release-cesm2/downloading_cesm.html#downloading-input-data), but be aware of the size of the files! 

To avoid duplication of input files in the order of TB the users and developers of NorESM, CTSM, and other related climate models in Norway share disk space on the clusters. To get access to this folder you need to ask about the permission of the group owner. 

