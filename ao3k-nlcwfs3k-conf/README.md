# Overview

non-Linear Curvature Wavefront Sensor(nlCWFS) onto bimorph 188.

TO BE AMENDED.


# Running the example

:warning: Check the [instructions](https://github.com/cacao-org/cacao/tree/dev/AOloopControl/examples) before running these steps

## Setting up processes


```bash
# Deploy configuration :
# download from source to current directory
cacao-loop-deploy -c ao3k-nlcwfs3k

# OPTIONAL: Change loop number, name, DM index, simulation DM index:
# CACAO_LOOPNUMBER=7 cacao-loop-deploy -c scexao-vispyr-bin2
# CACAO_LOOPNUMBER=7 CACAO_DMINDEX="03" cacao-loop-deploy -c scexao-vispyr-bin2

# OPTIONAL: Edit file scexao-vispyr-bin2-conf/cacaovars.bash as needed
# For example, change loop index, DM index, etc ...

# Run deployment (starts conf processes)
cacao-loop-deploy -r ao3k-nlcwfs3k

# Note: the copy and run steps can be done at once with:
# cacao-loop-deploy scexao-vispyr-bin2


# Go to rootdir, from which user controls the loop
cd nlcwfs3k-rootdir

# select simulation mode
#./scripts/aorun-setmode-sim
./scripts/aorun-setmode-hardw
# (alternatively, run ./scripts/aorun-setmode-hardw to connect to hardware)
```

## Run DM and WFS simulators

```bash
# Run hardware DM (optional if running in simulation mode)
# cacao-aorun-000-dm start

# Run simulation DM
#cacao-aorun-001-dmsim start

# Start simulation processes
# (skip if in hardware mode)
#cacao-aorun-002-simwfs -w start
```



## Measure WFS dark


```bash
cacao-aorun-005-takedark -n 2000
```



## Start WFS acquisition

```bash
# Acquire WFS frames
cacao-aorun-025-acqWFS -w start
```

```bash
# Acquire WFS reference
cacao-aorun-026-takeref -n 2000
```


## Measure DM to WFS latency

```bash
# Measure latency
cacao-aorun-020-mlat -w
```



## Acquire response matrix


### Prepare DM poke modes

```bash
# Create DM poke mode cubes
cacao-mkDMpokemodes -z 5 -c 32
```
The following files are written to ./conf/RMmodesDM/
| File                 | Contents                                            |
| -------------------- | --------------------------------------------------- |
| `DMmask.fits     `   | DM mask                                             |
| `FpokesC.<CPA>.fits` | Fourier modes (where \<CPA> is an integer)          |
| `ZpokesC.<NUM>.fits` | Zernike modes (where \<NUM> is the number of modes) |
| `HpokeC.fits     `   | Hadamard modes                                      |
| `Hmat.fits       `   | Hadamard matrix (to convert Hadamard-zonal)         |
| `Hpixindex.fits  `   | Hadamard pixel index                                |
| `SmodesC.fits    `   | *Simple* (single actuator) pokes                    |



### Run acquisition


```bash
# Acquire response matrix - Hadamard modes
cacao-fpsctrl setval measlinresp ampl 0.005
cacao-aorun-030-acqlinResp -n 10 HpokeC
# 20 cycles - default is 10.
```

### Decode Hadamard matrix

```bash
cacao-aorun-031-RMHdecode
```

### Make DM, WFS masks

```bash
cacao-aorun-032-RMmkmask
# ./scripts/nlcwfs-RMmkmask
```


## Compute control matrix (straight)

Compute control modes, in both WFS and DM spaces.

```bash
cacao-fpsctrl setval compstrCM svdlim 0.1
```
Then run the compstrCM process to compute CM and load it to shared memory :
```bash
cacao-aorun-039-compstrCM
```



## Running the loop


Start the 3 control loop processes :

```bash
# start WFS -> mode coefficient values
cacao-aorun-050-wfs2cmval start

# start modal filtering
cacao-aorun-060-mfilt start

# start mode coeff values -> DM
cacao-aorun-070-cmval2dm start

```

Closing the loop and setting loop parameters with mfilt:

```bash
# Set loop gain
cacao-fpsctrl setval mfilt loopgain 0.1

# Set loop mult
cacao-fpsctrl setval mfilt loopmult 0.99

# close loop
cacao-fpsctrl setval mfilt loopON ON

```


THE END
