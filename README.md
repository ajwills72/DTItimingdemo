# DTItimingdemo

This is a demonstration of how to substantially speed up a DTI analysis script on a
Linux workstation. The original [script](DTIprocessBRIC), kindly provided by
Matt Roser, processed 28 participants, and took about 5 hours to run on a 2019
12-core workstation. By using GNU parallel, Ireduced that 30 minutes
on the same workstation. Timings calculated using `time ./pipeline`.

_Context_: This project was initially intended as a test of speed up on a HPC
cluster (~1600 nodes). However, the underlying library functions (from FSL)
only parallelize down to the level of an individual participant. So, moving to
the HPC cluster would only increase speed by around a factor of 2 (there are
only 28 participants), and the 15 minutes saved do not merit the time it would
take to set up FSL on the HPC cluster. FSL also has the ability to interface
with an HPC cluster using Son of a Grid Engine software, which is not
implemented on the HPC cluster we planned to use (FOSERES), but for this job,
this would be no faster than minorly modifying the current script to work with
SLURM. So, overall, the computation performed by this script is not
sufficiently intensive to be worth running on an HPC cluster. 

## Running the demo

### Installation

1. The demo requires installation of FSL, which is a 4GB download, needs at
least 15GB spare space on the machine, and requires admin rights to install. To
install FSL, go to [this
page](https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/FslInstallation) to download an
installer script and run it. **Anti-features**: FSL is restrictively licensed -
specifically, unlike [free
software](https://en.wikipedia.org/wiki/The_Free_Software_Definition)
(e.g. [AFNI](https://afni.nimh.nih.gov/), R, Python), you do not have the right
to modify or distribute the software. It is also restricted to non-commercial
use.

2. You must also install GNU parallel `sudo apt install parallel`.

3. Download this git repository to your machine.

4. The data for this script (about 400MB) is stored in a private [OSF
repository](https://osf.io/afmc5/). Ask Andy Wills for access. Download the
.zip file, unzip, and put all the files you obtain in a directory called
'data'.

### Running the script

This script was tested on FSL 6.0.3, installing FSL to Ubuntu 18.04 in the
default location: /usr/local/fsl

To run, type './pipeline' at the command prompt.

## Understanding the speedup

Most of the functions in FSL are single-core, which means on multi-core
machines (almost every machine in the last 10 years), you can easily speed up
analysis scripts by passing each participant to a different core. The command
`parallel` makes it easy to do this in a BASH script.

## Understanding the script

Most of the processing steps are performed separately on each participant,
which means they can be parallelized. This is done by having a script `preproc`
that's called with the name of the `nii.gz` file you want to process e.g.

`preproc DTI1.nii.gz`

In the script `pipeline`, the key line is:

`ls data/*.nii.gz | parallel ./preproc`

which lists (`ls`) all files in the `data` directory ending in `.nii.gz` and
pipes (`|`) that to the `parallel` command, which the runs `preproc` as many
times as their are files, spreading this across all the cores of your machine.

The final two stages of the analysis (`tbss_3_postreg` and `tbss_4_prestats`)
cannot be parallelized in this way, and so are just called individually at the
end of the `pipeline` script.

## Components of the script

In the below, (intensive) means it makes humanly-relevant time to
run. (trivial) means it does not.

### In `preproc`

- `eddy_correct`: (intensive) Corrects for eddy currents and motion artifacts. Note
  that this command has been superceded by
  [eddy](https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/eddy)

- `fslsplit`: (trivial). Converts 4D image files to 3D. Part of the
  [FSLUTILS](https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/Fslutils) component of FSL.

- `bet`: (trivial). Deletes non-brain tissue from an image. Part of
  the [BET](https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/BET) component of FSL.

- `dtifit`: (intensive). Fits a diffusion tensor model at each voxel. Part
  of the FDT toolbox of FSL. 
 
- `tbss_1_preproc`: (trivial). Erodes FA images
  and zeros end slices. Would be better placed in Stage 1 for easier
  parallelization as it operates separately on each participant.

- `tbss_2_reg`: (intensive). Non-linear registration to align each subject to a
  standard target (FMRIB58_FA)

## After `preproc`

- `tbss_3_postreg`: single core, but too fast to worry about. Aligns above
  target to MNI152 space.  Then aligns each participant to the target image,
  and transforms to MNI152 space. Creates a single 4D image called `all_FA` of
  all participants, and a mean image called `mean_FA` and, from that,
  skeletonisation into `mean_FA_skeleton`.

- `tbss_4_prestats`: single core, but too fast to worry about. Results in a 4D
  image file (where 'timepoint' is subject ID) containing the projects
  sekeltonized FA data. It's called `all_FA_skeletonised.nii.gz`. This is what
  the subsequent analyses are performed upon.

## Viewing images remotely

If you want to view image files, use the `-X` option in `ssh` when logging in

`ssh -X andy@10.xxx.xx.xxx`

then you can use `xdg-open` to view images, e.g.:

`xdg-open DTI10_FA_FA.png`

You can move between images in the same directory within this window (click arrows). 

## Useful links

- [Basic guide to Linux and Sun Grid Engine for
  imagers](https://bioinformatics.mdc-berlin.de/intro2UnixandSGE/index.html)

- [Son of Grid Engine documentation](https://arc.liv.ac.uk/SGE/)

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

- The [HPC system](http://hpc.math-sciences.org/) at Plymouth University
