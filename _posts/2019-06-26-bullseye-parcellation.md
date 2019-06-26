---
title: 'bullseye parcellation of the cerebral white matter with FreeSurfer'
date: 2019-06-26
permalink: /posts/2019/06/bullseye-parcellation/
tags:
  - neuroimaging
  - FreeSurfer
---

In this post I will explain the nits and grits on how to create a bullseye parcellation of the cerebral white matter using (part of) the FreeSurfer outputs and commands.
The software package recreating the parcellations according to this post is freely-available in [this](https://github.com/gsanroma/bullseye_pipeline) github repository.
So, let's get started!

The bullseye parcellation that we are aim at creating divides the white matter into a set of concentric regions (ie, layers) spanning from ventricles to cortex that are in turn divided into spatially contiguous regions (ie, lobes).
It can be used to obtain region-specific quantification of white matter parameters (eg, a similar approach has been used to quantify regional white matter hyperintensity load in [this](https://doi.org/10.1016/j.neurad.2017.10.001) and [this](https://doi.org/10.1016/j.jalz.2014.07.155) papers). 


The image below shows an example bullseye parcellation

![](/images/blog/2019-06-26-bullseye-parcellation/bull.png)

The bullseye parcellation is the intersection of a _concentric_ parcellation dividing the brain into equidistant concentric shells from ventricles to cortex, and a _lobar_ parcellation dividing the brain into its main lobes.

- concentric parcellation

![](/images/blog/2019-06-26-bullseye-parcellation/shells.png)

- lobar parcellation

![](/images/blog/2019-06-26-bullseye-parcellation/lobes.png)

We divide the process into 3 phases:

- lobar parcellation
- concentric parcellation
- bullseye parcellation

## lobar parcellation

We take advantage of FreeSurfer outputs and commands to create the lobar parcellation.
We do not need all but only a portion of FreeSurfer outputs.
More specifically, from the FS subject directory:

- `label/{l,r}h.aparc.annot`
- `surf/{l,r}h.white`
- `surf/{l,r}h.pial`
- `mri/ribbon.mgz`
- `mri/aseg.mgz`

In the first step, we aggregate the cortical parcellation in `aparc` into a set of lobes with the command:

```bash
mri_annotation2label --subject subject --hemi rh --lobesStrict lobes
```

This command adds the output `label/rh.lobes.annot` into the FS structure (repeat the command for the left hemisphere `lh`).

The next step, we need to map the lobes to a volume.
This we do with `mri_aparc2aseg` FS command:

```bash
mri_aparc2aseg --s subject --annot lobes --labelwm --wmparc-dmax 1000 --rip-unknown --hypo-as-wm --o lobes+aseg.nii.gz
```

This creates the volume `lobes+aseg.nii.gz` containing the projection of the lobar labels into the white matter.
A few notes on this command:

- the `--labelwm` option projects the cortical annotations (in our case the lobes) along the white matter
- the `--wmparc-dmax` specifies the distance (in mm) that the lobar labels are projected down the white matter. We use a safely large number (ie, 1000) to make sure that we reach the ventricles
- we also remove unknown labels (`--rip-unknown`) and treat hypointensities as white matter (`--hypo-as-wm`)

We note here that FS's definition of lobes is not exactly the same as the one used in the papers above on regional WMH quantification.
To improve compatibility, we do some cleaning and re-arranging of labels.
In particular, we merge the _insular_ with the _frontal_ lobe and remove a resulting lobe spanning from anterior to posterior in the superior part of the brain (sorry that I do not know the name).

This can be accomplished with the `filter_labels.py` script from my [nimg-scripts](https://github.com/gsanroma/nimg-scripts) repository, as follows:

```bash
python filter_labels.py --in_dir path/to/lobar/folder/ --in_suffix +aseg.nii.gz --out_dir path/to/output/folder --out_suffix +lobes.nii.gz --include 3001 3007 --include 4001 4007 --include 3004 --include 4004 --include 3005 --include 4005 --include 3006 --include 4006 --map 3001 11 --map 4001 21 --map 3004 12 --map 4004 22 --map 3005 13 --map 4005 23 --map 3006 14 --map 4006 24
```

Very briefly, we merge the _insular_ (`3007`, `4007`) with frontal (`3001`, `4001`) lobes.
We exclude the ~~weird~~ lobe in the superior part of the brain (`3003`, `4003`).
Lastly, we assign informative 2-digit labels to the lobes (`--map` arguments), with 1st digit denoting the hemisphere and 2nd digit the acutal lobe.


