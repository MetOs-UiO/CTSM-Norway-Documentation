# Needed accesses 

This page is a guide to setup and use of CTSM on the norwegian clusters
rund by Sigma2. 

## The Sigma2-clusters 
To get access to these you need to create an account by
filling out [this](https://www.metacenter.no/user/application/form/notur/) form. You will
need a project code that you can ask your project manager( ie.
supervisor) for. Once this form is filled out the project manager and
sigma2 will need to approve the application. This will take some days,
and you\'re good to go.

```{discussion} First time on a HPC-cluster? 

  If this is your first time using a remote HPC-system, or you want to
  know more about Sigma2\'s setup this
  [tutorial](https://sabryr.github.io/hpc-intro/12-cluster/index.html) is
  a great place to start.
```


## Input data folders 
All the runs of CTSM require certain input data for forcing data, grids, etc. Usually these files are downloaded automatically from [NCAR's service](https://escomp.github.io/CESM/release-cesm2/downloading_cesm.html#downloading-input-data), but be aware of the size of the files! 

To avoid duplication of input files in the order of TB the users and developers of NorESM, CTSM, and other related climate models in Norway share diskspace on the clusters. To get access to this folder you need to ask about the permission of the group owner. Y 