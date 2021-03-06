# What is this?

In 2020, I was asked to help a colleague install WRF-Hydro.  The dockerfile at the time wasn't operating as expected.  
The final docker image that I've built here is equivalent 
to `wrfhydro/dev:conda` except some modifications from my own end that included 
some fixes.  This has been uploaded to Github simply for personal documentation reasons.  Use at your own risk.  

## I NEED HELP!

Use this [PDF document](https://ral.ucar.edu/sites/default/files/public/projects/Technical%20Description%20%26amp%3B%20User%20Guides/howtobuildrunwrfhydrov511instandalonemode.pdf)

Some other useful links are:

- [https://ral.ucar.edu/projects/wrf_hydro/model-code](https://ral.ucar.edu/projects/wrf_hydro/model-code)
- [https://github.com/NCAR/wrf_hydro_nwm_public](https://github.com/NCAR/wrf_hydro_nwm_public)
- [https://github.com/NCAR/wrf_hydro_docker](https://github.com/NCAR/wrf_hydro_docker)


# Setting Up the Environment

## Download the Docker Image

Previously there was a docker image (not pushed to Docker Hub) hosted on one of my webservers.  

Since then it's been removed.  Please compile manually from here or use the wrf-hydro base docker file.  

Another point to note is that this doesn't fully work still.  I still need to modify it to use Intel compilers (gcc works but... doesn't...).  I've ported it to Github for personal documentation/storage purposes.  Use at your own risk.  

## Loading Docker Image

Load the docker image into your local Docker Registry.

```bash
docker load -i donwrfhydro.tar
```

## Running Docker Image (Without Volume Mount)

If you're running WRF-Hydro instance without needing a volume mount, then use this command.  

```bash
docker run -it donwrfhydro/dev:latest
```

## Optional: Running Docker Image with a Volume Mount

This is similar to how WRF-Hydro says you should handle volume mount.  I'm just adding it here for my own version.

```bash
docker run -v <path-to-your-local-mount-folder>:<path-to-the-desired-docker-folder> -it donwrfhydro/dev:latest
```

An example of this is:

```bash
docker run -v WRFHydro:/mnt/WRFHydro -it donwrfhydro/dev:latest
```

# Installing WRF-Hydro Standalone

## Download and Configuring

The last command `docker run -it donwrfhydro/dev:latest` that you used above should have you enter the docker container.  Within it, you now have to build WRF-Hydro itself.  Quick note though is that you should NOT be building WRF-Hydro in your mounted volume.  

I like to make a folder first that WRF-Hydro will live in.  Then I download, extract, and configure.  In "command line" form, this is:

```bash
mkdir wrfhydro
cd wrfhydro
wget https://github.com/NCAR/wrf_hydro_nwm_public/archive/v5.2.0.tar.gz
tar -zxvf v5.2.0.tar.gz
cd wrf_hydro_nwm_public-5.2.0/trunk/NDHMS
./configure 2
cp template/setEnvar.sh .
```

Now this is the part where you need to have some manual configuration before building WRF-Hydro.  You need to edit the `setEnvar.sh` file based on what you need.  The PDF document I linked above about building WRF-Hydro for Standalone run suggests the configuration I've pasted below.  

Basically, I run one of the following:

```bash
nano setEnvar.sh
vi setEnvar.sh
```

Within the document, you can set your configuration any way you want.  Here is an example:

```bash
#!/bin/bash

# WRF-Hydro compile time options
# Always set WRF_HYDRO to 1
export WRF_HYDRO=1

# Enhanced diagnostic output for debugging: 0=Off, 1=On.
export HYDRO_D=0

# Spatially distributed parameters for NoahMP: 0=Off, 1=On.
export SPATIAL_SOIL=1 

# RAPID model: 0=Off, 1=On.
export WRF_HYDRO_RAPID=0 

# Large netcdf file support: 0=Off, 1=On.
export WRFIO_NCD_LARGE_FILE_SUPPORT=1 

# WCOSS file units: 0=Off, 1=On.
export NCEP_WCOSS=0 

# Streamflow nudging: 0=Off, 1=On.
export WRF_HYDRO_NUDGING=0 

```

Honestly, you should read the manual a bit further and the code regarding what's going on here, but I went with the default configuration.  You can close the file using either CTRL+X (for Nano editor) or :wq! (for Vi Editor)

## Building

So now you have to figure out if you're going to use Noah or NoahMP land surface model.  I'm assuming NoahMP is just the parallelized version of Noah (I don't know but I'm assuming they're basically the same).  So use any of the following lines of code based on your CPU commit and usage.

**REMEMBER:** Use one or the other.  Don't run both of these commands

```bash
./compile_offline_Noah.sh setEnvar.sh
```

**OR**

```bash
./compile_offline_NoahMP.sh setEnvar.sh
```

Now find the binary compiled and ready at:

```bash
cd Run/
```

Based on what you chose above, the executable binary will be named:

```bash
wrf_hydro_Noah.exe
wrf_hydro_NoahMP.exe
```

However, they'll be symlinked as:

```bash
wrf_hydro.exe
```

You can run `wrf_hydro.exe` with the demo/base case and go from there.  

You're done!  The instructions below are for building the docker image from scratch.  **You don't need what's below here.**

# Building WRF-Hydro Docker Image from Scratch

These are the steps.

I first used the `Dockerfile` I have uploaded into this repository.  The `Dockerfile` has been modified from `wrfhydro/dev:base` to have the following:

- Commands to build `wrfhydro/dev:conda` 
- Installing sudo as this is not present in base Ubuntu 16.04 docker image (and therefore you can't sudo anything)
- Enables passwordless sudo for `docker` user as the password is usually automatically generated and not available for the end user

Did:

```bash
docker image build . -t donwrfhydro/dev:latest
```

Then save to a docker image.

```bash
docker save donwrfhydro/dev:latest -o donwrfhydro.tar
```

Then I uploaded the docker tarball onto a web server instead of using a docker hub or registry.  

# TODO

1. Automatically install and build WRF-Hydro (Standalone)
2. Push to Docker Hub(?)
3. Upgrade to Ubuntu Focal (20.04) due to Ubuntu Xenial (16.04) LTS support ends April 2021
4. Fix the container registry that comes built-in with GitLab.  
5. Fix with Intel compiler
