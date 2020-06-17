---
title: Log in and environment set up in ARCHER 
teaching: 5
exercises: 0
questions:
- "How do I access ARCHER?"
- "How do I set up the correct environment in ARCHER?"
objectives:
- "Log in to ARCHER."
- "Load AmberTools module in ARCHER."
keypoints:
- "SSH to ARCHER"
- "Load Ambertools with `module load`."
---


In this session we are going to use several software packages:  

- [AmberTools](https://ambermd.org/AmberTools.php) (We will use the latest version of Ambertools. If you are using an older version, commands might run slightly different.)
- [CP2K](https://www.cp2k.org)
- [PROPKA](https://github.com/jensengroup/propka-3.1) or [PROPKA server](http://server.poissonboltzmann.org/pdb2pqr) 
- [OpenBabel](http://openbabel.org/wiki/Main_Page)
- [Pymol](https://sourceforge.net/projects/pymol/) or [VMD](https://www.ks.uiuc.edu/Research/vmd/) or another molecular visualisation tool of your preference.
- [Marvin Sketch](https://chemaxon.com/products/marvin).

We are going to use AmberTools and CP2K on ARCHER. 

***


## Log in to ARCHER:

For your convenience, all the required software has been installed in  ARCHER. Therefore, you have to log in with the credential that have been sent to you.

~~~
ssh -XY user@login.archer.ac.uk
~~~
{: .language-bash}


***

## Load the environment

First, to set up the proper environment you should run the following commands on ARCHER. These set of commands will give you access to the installed software:

~~~
module swap PrgEnv-cray PrgEnv-gnu
module load amber-tools/20 
module load propka
module load openbabel/2.4.1
~~~
{: .language-bash}

Then we should move to out `work` folder. You should substitute XXX for the project name and YYY for your username:

~~~
pwd
cd /work/XXX/XXX/YYY/
~~~
{: .language-bash}

***

## Downloading data

Next, you need to download some files to follow this workshop:
~~~
git config --global http.sslverify false
git clone https://git.ecdf.ed.ac.uk/sllabres/files_ambertools4cp2k.git
~~~
{: .language-bash}


> ## About the data
>
> It provides:
> * PDB file  with the initial coordinates. For this workshop we will use a X-ray structure of the main protease of the SARS-CoV2 virus (PDB id [5R84](https://www.rcsb.org/structure/5R84)) with a small-molecule bound to its catalytic site.
> * ARCHER submission scripts.
> * AmberTools input files (LEap, sander and cpptraj input files)
> * CP2K input file.
>
{: .prereq}

Once you have downloaded the files you can go into the `files_ambertools4cp2k` folder and proceed to the next section:

~~~
cd files_ambertools4cp2k
~~~
{: .language-bash}
