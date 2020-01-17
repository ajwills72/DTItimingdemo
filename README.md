# DTItimingdemo

## FSL installation

The demo requires installation of FSL, which is a 4GB download, needs at least
15GB spare space on the machine, and requires admin rights to install.

To install FSL, go to [this
page](https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/FslInstallation) to download an
installer script and run it.

**Anti-features**: FSL is restrictively licensed - specifically, unlike [free
software](https://en.wikipedia.org/wiki/The_Free_Software_Definition) (e.g. R,
Python), you do not have the right to modify or distribute the software. It is
also restricted to non-commercial use. 

## Data location

Data for this script is stored in an [OSF repository](https://osf.io/afmc5/). 

## Script

This script was tested on FSL 6.0.3, installing FSL to Ubuntu 18.04 in the
default location: /usr/local/fsl

### Stage 1

- `eddy_correct`: single-core. Corrects for eddy currents and motion
  artifacts. Note that this command has been superceded by
  [eddy](https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/eddy)

- `fslsplit`: trivial execution time. Converts 4D image files to 3D. Part of the
  [FSLUTILS](https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/Fslutils) component of FSL.

- `bet`: trivial execution time. Deletes non-brain tissue from an image. Part of
  the [BET](https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/BET) component of FSL.

- `dtifit`: single-core. Fits a diffusion tensor model at each voxel. Part
  of the FDT toolbox of FSL. 

## Improving script for workstation use

Stage 1 of the analysis is single-core. This can be speeded by a factor of the
number of cores on the workstation, by using GNU parallel. Scripts `preproc`
and `pipeline` show how. You may need to `sudo apt install parallel`. 

## Cluster computing notes

### Beyond FOSERES

FOSERES uses SLURM, while a few FSL tools (feat, melodic, tbss, bedpostx,
fslvbm, possum) use Grid Engine variants where available.

bedpostx can use CUDA GPUs where available, for a 100x speedup.

Maybe these are things for the new system...

## Useful links

- [FSL main page](https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/FSL)

- [FDT main page](https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/FDT), FDT is FSL's
  diffusion toolbox.

- A wrapper for FSL in R
  [FSLR](https://johnmuschelli.com/neuroc/DTI_analysis_fslr/index.html)

- [Visualizing MRI data in R](https://www.alexejgossmann.com/MRI_viz/) 

- [Python package to handle OSF from command
  line](https://osfclient.readthedocs.io/en/latest/cli-usage.html)

- For getting data onto to a remote machine [scp
  cheatsheet](https://github.com/tldr-pages/tldr/blob/master/pages/common/scp.md)

