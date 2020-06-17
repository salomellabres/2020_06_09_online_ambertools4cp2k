---
title: Parameterising your ligands
teaching: 15
exercises: 0
questions:
- "How do I generate parameters for non-conventional residues?"
objectives:
- "Generate parameters for GWS ligand using `antechamber` and `parmchk2`."
keypoints:
- "Generating parameters for non-conventinal residues is not easy and there are different ways to parameterise them."
- "The quality of the parameters is important to obtain valid results."
- "`antechamber` creates parameters for a non-conventional residues."
- "`parmchk2` checks for missing parameters in the forcefield."
- "ALWAYS check the parameters for your system and test that they behave as expected."
---

In this tutorial, we will make use of the `antechamber` and `parmchk2` tools from AmberTools package to generate AMBER force field parameters. We will use the "**general AMBER force field 2 [(GAFF2)](http://ambermd.org/antechamber/gaff.html)**" parameters. This force field was designed to provide atom types and atom parameters needed to parameterise most pharmaceutical molecules, and is compatible with the traditional AMBER force fields for proteins. To assign partial charges, we are going to use **[AM1-BCC2 charge method](https://pubmed.ncbi.nlm.nih.gov/12395429/)**. It is an inexpensive and fast method for calculating partial charges and be we don't need to worry too much about its limitations as partial charges won't be used during production runs due to the QM treatment of the ligand in the QM/MM simulation.  

***

## Generating point charges and assigning atomtypes with `antechamber`

`antechamber` allows the rapid generation of topology files for use with the AMBER simulation programs. It is a higher-level wrapper around other AmberTools programs (`sqm`, `divcon`, `atomtype`, `am1bcc`, `bondtype`, `espgen`, `respgen` and `prepgen`) and is able to:

- Automatically identify bond and atom types
- Judge atomic equivalence
- Generate residue topology files
- Find missing force field parameters and supply reasonable suggestions

We will use `antechamber` to assign atom types to the GWS ligand and calculate a set of point charges. We will have to specify GAFF2 force field (`-at gaff2`, more information on the parameters here: `$AMBERHOME/dat/leap/parm/gaff2.dat`), the [AM1-BCC2](https://pubmed.ncbi.nlm.nih.gov/12395429/) charge method (`-c bcc`), the net charge of the molecule (`-nc 0`) and the name of the new residue generated (We are going to keep GWS: `-rn GWS`). We are going to output a mol2-type file (`-fo mol2`) containing the atomtypes and the point charges.  

~~~
antechamber -i GWS.H.pdb -fi pdb -o GWS.mol2 -fo mol2 -c bcc -nc 0 -rn GWS -at gaff2
~~~
{: .language-bash}

~~~
Welcome to antechamber 20.0: molecular input file processor.

acdoctor mode is on: check and diagnosis problems in the input file.
-- Check Format for pdb File --
   Status: pass
-- Check Unusual Elements --
   Status: pass
-- Check Open Valences --
   Status: pass
-- Check Geometry --
      for those bonded   
      for those not bonded   
   Status: pass
-- Check Weird Bonds --
   Status: pass
-- Check Number of Units --
   Status: pass
acdoctor mode has completed checking the input file.

Info: Total number of electrons: 118; net charge: 0

Running: /home/d118/d118/shared/amber20/bin/sqm -O -i sqm.in -o sqm.out
~~~
{: .output}

This step generates a lot of intermediate files (in capital letters). You can safely delete them as these are used in antechamber and we no longer require them. The `sqm.xxx` files are input and output from the `sqm` quantum mechanics code used by `antechamber` to calculate the atomic point charges. We are not interested in the data here except to check that the sqm calculation completed successfully.

~~~
tail  sqm.out
~~~
{: .language-bash}

~~~
           --------- Calculation Completed ----------
~~~
{: .output}

Once we are sure that the point charges were generated The MOL2 file contains tridimensional information of the molecule as well as atom type and point charges. You can find more information on the MOL2 format [here](http://chemyang.ccnu.edu.cn/ccb/server/AIMMS/mol2.pdf). The resulting MOL2 file is shown below:

~~~
<TRIPOS>MOLECULE
GWS 
   34    35     1     0     0
SMALL
bcc


@<TRIPOS>ATOM
      1 N1           7.5340     0.3440    17.5810 nb      1001 GWS      -0.643000
      2 C4          13.4880    -0.5140    24.0620 c3      1001 GWS      -0.075900
      3 C5          13.9760    -1.6820    23.2210 c3      1001 GWS      -0.078400
                                       ...
~~~
{: .output}

> ## TIP
>
> The GAFF and GAFF2 atom types are in lower case. This is the mechanism by which the GAFF force field is kept independent from the macromolecular AMBER force fields. All of the traditional AMBER force fields use uppercase atom types. In this way the GAFF and traditional force fields can be mixed in the same calculation.
{: .callout}

***

## Looking for missing force field parameters with `parmchk2`

While the most likely combinations of bond, angle and dihedral parameters are defined in the parameter file it is possible that our molecule might contain combinations of atom types for bonds, angles or dihedrals that have not been parameterised. If this is the case, we will have to specify any missing parameters before we can create our prmtop and inpcrd files in LEap. 

We will use `parmchk2` to test if all the parameters we require are available.

~~~
parmchk2 -i GWS.mol2 -f mol2 -o GWS.frcmod -s gaff2
~~~
{: .language-bash}

`parmchk2` generates a parameter file that can be loaded into Leap in order to add missing parameters. It contains all of the missing parameters. If possible, it will fill in these missing parameters by analogy to a similar parameter. Let's look at the generated GWS.frcmod file: 

~~~
Remark line goes here
MASS

BOND
ca-ns  315.20   1.412       same as ca- n, penalty score=  0.0
c -ns  356.20   1.379       same as  c- n, penalty score=  0.0
hn-ns  527.30   1.013       same as hn- n, penalty score=  0.0

ANGLE
c -ns-ca   65.700     123.710   same as c -n -ca, penalty score=  0.0
ca-ns-hn   48.000     116.000   same as ca-n -hn, penalty score=  0.0
ns-c -o   113.800     123.050   same as n -c -o , penalty score=  0.0
c -ns-hn   48.700     117.550   same as c -n -hn, penalty score=  0.0
ca-ca-ns   85.600     120.190   same as ca-ca-n , penalty score=  0.0
c3-c -ns   84.300     115.180   same as c3-c -n , penalty score=  0.0

DIHE
o -c -ns-ca   4   10.000       180.000           2.000      same as X -c -n -X , penalty score=  0.0
c3-c -ns-ca   1    0.750       180.000          -2.000      same as c3-c -n -ca
c3-c -ns-ca   1    0.500         0.000           3.000      same as c3-c -n -ca, penalty score=  0.0
o -c -ns-hn   1    2.500       180.000          -2.000      same as hn-n -c -o
o -c -ns-hn   1    2.000         0.000           1.000      same as hn-n -c -o , penalty score=  0.0
ca-ca-ns-c    1    0.950       180.000           2.000      same as c -n -ca-ca, penalty score=  0.0
ns-c -c3-c3   1    0.000       180.000          -4.000      same as n -c -c3-c3
ns-c -c3-c3   1    0.710       180.000           2.000      same as n -c -c3-c3, penalty score=  0.0
ca-ca-ns-hn   4    1.800       180.000           2.000      same as X -ca-n -X , penalty score=  0.0
c3-c -ns-hn   4   10.000       180.000           2.000      same as X -c -n -X , penalty score=  0.0

IMPROPER
ca-ca-ca-ns         1.1          180.0         2.0          Using the default value
ca-ca-ca-ha         1.1          180.0         2.0          Using general improper torsional angle  X- X-ca-ha, penalty score=  6.0)
c3-ns-c -o         10.5          180.0         2.0          Using general improper torsional angle  X- X- c- o, penalty score=  6.0)
c -ca-ns-hn         1.1          180.0         2.0          Same as X -X -n -hn, penalty score=  6.0 (use general term))
ca-h4-ca-nb         1.1          180.0         2.0          Same as X -X -ca-ha, penalty score= 44.3 (use general term))

NONBON

~~~
{: .output}

> ## TIP
> 
> You should check these parameters carefully before running a simulation. If antechamber can't empirically calculate a value or has no analogy it will either add a default value that it thinks is reasonable or alternatively insert a place holder (with zeros everywhere) and the comment "**ATTN: needs revision**". In this case you will have to manually parameterise this yourself. See the links at the beginning of the section.  
{: .callout}

***

> ## FURTHER INFORMATION:
>
>This step is not trivial and there are many ways to do it. In this example, as we are aiming to use a QM description of the ligand, we can use a simple approach. If you have trouble parameterising your ligands here are some useful links:
>
> * [AMBER tutorial on simple ligand parameterisation](https://ambermd.org/tutorials/basic/tutorial4/index.htm)
> * [AMBER tutorial on advanced ligand parameterisation](https://ambermd.org/tutorials/advanced/tutorial1/section1.htm)
> * [Parameterisation of dihedral angles](http://www.ub.edu/cbdd/?q=content/small-molecule-dihedrals-parametrization)
> * [AMBER parameter database](http://research.bmh.manchester.ac.uk/bryce/amber/)
> * [Paramfit tutorial](http://ambermd.org/tutorials/advanced/tutorial23/)
>
{: .challenge}


