---
title: Log in and environment set up in ARCHER 
teaching: 5
exercises: 0
questions:
- "How do I set up the correct environment in ARCHER?"
objectives:
- "Log in in ARCHER."
- "Load the correct envinronment in ARCHER."
keypoints:
-   "ARCHER usage for Ambertools with `module load`."
---


In this session we are going to use several software packages including AmberTools and CP2K.  

- [AmberTools](https://ambermd.org/AmberTools.php) (We will use the latest version of Ambertools. If you are using an older version, commands might run slightly different.)
- [propKa](http://propka.org)
- [OpenBabel](http://openbabel.org/wiki/Main_Page)
- [Pymol](https://sourceforge.net/projects/pymol/) or [VMD](https://www.ks.uiuc.edu/Research/vmd/) or another molecular visualisation tool of your preference.
- [Marvin Sketch](https://chemaxon.com/products/marvin).
- [CP2K](https://www.cp2k.org)

We will use these through ARCHER. Therefore, you have to log in with the credential that have been sent to you.

## Access ARCHER:

TBC 

~~~
ssh -XY user@login.archer.ac.uk
~~~
{: .language-bash}

## Load the environment

To set up the proper environment you should run the following commands on ARCHER:

~~~
module swap PrgEnv-cray PrgEnv-gnu
module add python-compute
module add cray-netcdf
module load amber-tools/20
export AMBERHOME="/home/d118/d118/shared/amber20/"
source /home/d118/d118/shared/amber20//amber.sh
module load propka
~~~
{: .language-bash}

***

## Downloading data

You need to download some files to follow this lesson:
~~~
git clone https://git.ecdf.ed.ac.uk/sllabres/files_ambertools4cp2k.git
~~~
{: .language-bash}

> ## About the data
>
> It provides:
> * For more information about the initial structure provided here, it is a X-ray structure of the main protease of SARS-CoV2 (PDB id [5R84](https://www.rcsb.org/structure/5R84)) with a fragment bound to its catalytic site.
> * ARCHER submission scripts.
> * AmberTools input files (leapin and sander input files)
> * CP2K input file.
>
{: .prereq}
