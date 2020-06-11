# What is this?

Basically, WRF-Hydro's docker containers (in my opinion) are a bit messed up and 
needed some fixing.  The final docker image that I've built here is equivalent 
to `wrfhydro/dev:conda` except some modifications from my own end that included 
some fixes since last time the WRF-Hydro docker images were built.  

## I NEED HELP!

If you need help.  Feel free to contact me (Don) or use this [PDF document](https://ral.ucar.edu/sites/default/files/public/projects/Technical%20Description%20%26amp%3B%20User%20Guides/howtobuildrunwrfhydrov511instandalonemode.pdf)

Some other useful links are:

- [https://ral.ucar.edu/projects/wrf_hydro/model-code](https://ral.ucar.edu/projects/wrf_hydro/model-code)
- [https://github.com/NCAR/wrf_hydro_nwm_public](https://github.com/NCAR/wrf_hydro_nwm_public)
- [https://github.com/NCAR/wrf_hydro_docker](https://github.com/NCAR/wrf_hydro_docker)


# Setting Up the Environment

## Download the Docker Image

Download the most updated docker image here: [http://donpark.me/donwrfhydro.tar](http://donpark.me/donwrfhydro.tar) [3.66 GB]

Or if you're on linux, you can download using `wget`. 

```bash
wget http://donpark.me/donwrfhydro.tar
```

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
wget https://github.com/NCAR/wrf_hydro_nwm_public/archive/v5.1.1.tar.gz
tar -zxvf v5.1.1.tar.gz
cd wrf_hydro_nwm_public-5.1.1/trunk/NDHMS
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

So you want to go through the steps I took eh?  Or this is just for Future Don to try and troubleshoot and remember what he did.  Either way, these are the steps.

I first used the `Dockerfile` I have uploaded into this repository.  The `Dockerfile` has been modified from `wrfhydro/dev:base` to have the following:

- Commands to build `wrfhydro/dev:conda` 
- Enables sudo for `docker` user (previous wrfhydro docker image does not have this)
- Enables passwordless sudo for `docker` user

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