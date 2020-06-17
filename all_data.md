# Preparing to run biomolecular QM/MM simulations with CP2K using AmberTools

## Introduction:

QM/MM simulations allow us to model different biological processes at atomistic level and study specific properties that no other method can provide (such as photochemical effects, ligand binding, reactivity, and a long etcetera). Also they benefit from the accuracy of quantum mechanics (QM) and the speed of molecular mechanics (MM) methods. However, the preparation of such simulations are not easy and tedious. Additionally often subtle changes in the setup process can affect the validity of the results. This course aims to showcase how set up your system in order to run QM/MM simulations using [CP2K](https://www.cp2k.org).

This course covers basic tools and technologies needed to succeed in the setting up a QM/MM simulation with AmberTools suite, with hands-on exercises. It provides background to preparation of biological systems for molecular modelling and practical experience in successfully creating topologies and running simple QM/MM simulations using CP2K in a HPC environment ([ARCHER](https://www.archer.ac.uk)).You will learn the intricacies of molecular modelling of biological systems, as well as practical aspects of handling and preparing PDB files, creating topologies ready to run using CP2K.

At the end of this session you will be able to…
- Describe the main features of the AmberTools package
- Build topologies and coordinates ready to use in CP2K
- Run simple QM/MM simulations in a HPC environment

This online course consists of an interactive hands-on practical session presented remotely using Collaborate. Attendees will be given access to ARCHER to execute the practical.

The duration of the sessions will be between **2 and 3 hours**, depending on the level of interactivity.

#### Prerequisites:
- Comfortable using the Linux command line
- Prior experience with HPC computing or QM/MM knowledge are not required.

#### Technical requirements:
- a microphone (webcam is optional) to take part
- Google Chrome or Firefox to access the session
- your laptop should have an SSH client (for window user Putty or any choice e.g. WDL) and a valid public SSH key.
Access to AmberTools and CP2K will be provided on ARCHER

#### Trainers:
Salome Llabres (EPCC), Julien Sindt (EPCC), Holly Judge (EPCC), Arno Proeme (EPCC)

This online course is a collaboration between [ARCHER2 Training](https://www.archer2.ac.uk/training/) and [BioExcel](https://bioexcel.eu).

![Sponsor logos](./images/logos.png)

***

## Introduction of AmberTools:

[AmberTools suite](https://ambermd.org/AmberTools.php) is a set of several independently developed packages developed as part of the Amber20 software package and distributed for free of charge, and its components are mostly released under the GNU General Public License (GPL). They work well by themselves, and with Amber20 itself. 

**Overview of AmberTools20:** (released on April 31, 2020)

- **NAB/sff**: a program build molecules, run MD or apply distance geometry restraints using generalized Born, Poisson-Boltzmann or 3D-RISM implicit solvent models
- **antechamber and MCPB**: programs to create force fields for general organic molecules and metal centers
- **tleap and parmed**: basic preparatory tools for Amber simulations
- **sqm**: semiempirical and DFTB quantum chemistry program
- **pbsa**: performs numerical solutions to Poisson-Boltzmann models
- **3D-RISM**: solves integral equation models for solvation
- **sander**: workhorse program for molecular dynamics simulations
- **gem.pmemd**: tools for using advanced force fields
- **mdgx**: a program for pushing the boundaries of Amber MD, primarily through parameter fitting. Also includes customizable virtual sites and explicit solvent MD capabilities.
- **cpptraj and pytraj**: tools for analyzing structure and dynamics in trajectories
- **MMPBSA.py**: energy-based analyses of MD trajectories

We are going to use a subset of these tools: **antechamber (sqm), tleap, parmed, sander and cpptraj**. 

***

## 0. Log in ARCHER and load the environment

Accessing ARCHER:

```
ssh -XY user@login.archer.ac.uk
```

**WIP**: I have installed AMBERTOOLS20 in the shared folder ```/home/d118/d118/shared/amber20```. I think you should be able to access it and use it. Let me know if you cannot!

**Software we will be using in this session:**
- [AmberTools](https://ambermd.org/AmberTools.php) (We will use Ambertools20. If you are using an older version, commands might run slightly different.)
- [propKa](http://propka.org)
- [OpenBabel](http://openbabel.org/wiki/Main_Page)
- [Pymol](https://sourceforge.net/projects/pymol/) or [VMD](https://www.ks.uiuc.edu/Research/vmd/) or another molecular visualisation tool of your preference. 
- [Marvin Sketch](https://chemaxon.com/products/marvin). 

To set up the proper environment you should run the following commands on ARCHER:

```
module load propka
module load python-compute/3.4.3
module swap PrgEnv-cray PrgEnv-gnu
export AMBERHOME="/home/d118/d118/shared/amber20/"
source /home/d118/d118/shared/amber20/amber.sh
```

***

## 1. Preparing the PDB file

### Understanding your PDB file

We are going to use the X-ray structure of Mpro protein bound to a ligand from the PanDDA library. The PDB id we are going to use is [5R84](http://www.rcsb.org/structure/5R84). If you open this entry from the [Protein DataBank](https://www.rcsb.org) you will find a lot of information of the protein structure:
- Experimental Data Snapshot
- wwPDB Validation
- Literature
- Macromolecules
- Small Molecules
- Experimental Data & Validation
- Entry History 

![PDB page 5r84](./images/PDBpage.png)

We are interested in the quality of the structure and the contents of the structure:
- **Experimental Data Snapshot** and **wwPDB Validation** give us information about the quality of the model. In this case, the X-ray structure is not so great but this fragment screening was resolved really fast to enable computational efforts. So it will have to do. For more information you can check out Bioexcel webinar on [Assessing structure quality in the PDB archive](https://bioexcel.eu/webinar-10-assessing-structure-quality-in-the-pdb-archive/). 
- The sections **Macromolecules** and **Small Molecules** give us information about the contents of the PDB file. We can see that there is only one macromolecule (only one chain) that corresponds to SARS-CoV-2 main protease. There are two small molecules in the structure: GWS and DMS. 
   - DMS is a crystallisation product and it doesn't interest us. 
   - GWS is the fragment we want to model.  

Once we know which molecules in the PDB file we want to model and which should be deleted from the PDB file, we can continue. 
You can download the PDB file now or use the one we are providing in this repository. 

***

### Splitting the PDB in protein, ligand and water molecules

First we will split the structure in the PDB in 3 different groups: protein, GWS ligand and water molecules as we need to treat them separately: 

```
grep -v -e "GWS" -e "DMS" -e "CONECT" -e "HOH" 5r84.pdb >  protein.pdb
grep "GWS" 5r84.pdb > GWS.pdb
grep "HOH" 5r84.pdb > waters.pdb
```

![Full system](./images/fullsystem.png)

***

### Replicating the experimental/target conditions

It is important to take into account the experimental conditions such as the pH of the system we want to simulate, salinity... Since Mpro protein from SARS-CoV2 will is located in the cytosol of the infected human cell so we are going to assume a **pH=7.4**. Please check the literature on your target protein before you start to run your simulations.

As we mentioned before, we will have two molecules: a protein and a ligand. We will have to take care of the effect of pH in each one of them. 

##### Assessing the protonation states of the protein and preparing the protein for AMBERTools

There are several ways to assess the protonation of the residues in the protein ([H++ server](http://biophysics.cs.vt.edu), [propka](http://propka.org)). We are going to use propKa command-line in this tutorial:

```
propka31 protein.pdb 
```

PropKa returns the calculated pKa values for each residue. We have to compare each one of the pKa values with the chosen pH: if the pKa is below the chosen pH the residue should be deprotonated and above the chosen pH it will be protonated. PropKa returns the model-pKa for each kind of residue, which determines the *usual* protonation state for that type of residue. 

> **TIP**: This step is not trivial when you are simulating catalytic sites, where the protonation changes can be part of the reaction mechanism. Please check the literature on your target protein before you start to run your simulations.

This is the summary of the propKa output for our protein:

```
--------------------------------------------------------------------------------------------------------
SUMMARY OF THIS PREDICTION
       Group      pKa  model-pKa   ligand atom-type
   ASP  33 A     3.79       3.80                      
   ASP  34 A     3.63       3.80                      
   ASP  48 A     2.69       3.80                      
   ASP  56 A     4.00       3.80                      
   ASP  92 A     3.11       3.80                      
   ASP 153 A     3.88       3.80                      
   ASP 155 A     3.00       3.80                      
   ASP 176 A     3.90       3.80                      
   ASP 187 A     3.97       3.80                      
   ASP 197 A     3.81       3.80                      
   ASP 216 A     3.52       3.80                      
   ASP 229 A     2.15       3.80                      
   ASP 245 A     4.03       3.80                      
   ASP 248 A     3.37       3.80                      
   ASP 263 A     3.53       3.80                      
   ASP 289 A     3.28       3.80                      
   ASP 295 A     3.91       3.80                      
   GLU  14 A     4.00       4.50                      
   GLU  47 A     4.67       4.50                      
   GLU  55 A     4.73       4.50                      
   GLU 166 A     3.98       4.50                      
   GLU 178 A     4.88       4.50                      
   GLU 240 A     4.60       4.50                      
   GLU 270 A     4.67       4.50                      
   GLU 288 A     4.42       4.50                      
   GLU 290 A     5.79       4.50                      
   HIS  41 A     4.60       6.50                      
   HIS  64 A     6.26       6.50                      
   HIS  80 A     5.71       6.50                      
   HIS 163 A     2.04       6.50                      
   HIS 164 A     1.44       6.50                      
   HIS 172 A     5.51       6.50                      
   HIS 246 A     5.37       6.50                      
   CYS  16 A    11.92       9.00                      
   CYS  22 A    10.26       9.00                      
   CYS  38 A    12.83       9.00                      
   CYS  44 A    11.04       9.00                      
   CYS  85 A    11.68       9.00                      
   CYS 117 A    11.97       9.00                      
   CYS 128 A    12.86       9.00                      
   CYS 145 A    11.82       9.00                      
   CYS 156 A     9.54       9.00                      
   CYS 160 A    13.16       9.00                      
   CYS 265 A    12.18       9.00                      
   CYS 300 A    10.29       9.00                      
   TYR  37 A    11.88      10.00                      
   TYR  54 A    15.15      10.00                      
   TYR 101 A    12.87      10.00                      
   TYR 118 A    10.28      10.00                      
   TYR 126 A    13.10      10.00                      
   TYR 154 A    10.17      10.00                      
   TYR 161 A    15.42      10.00                      
   TYR 182 A    13.69      10.00                      
   TYR 209 A    13.21      10.00                      
   TYR 237 A    10.15      10.00                      
   TYR 239 A    13.26      10.00                      
   LYS   5 A    10.57      10.50                      
   LYS  12 A    10.41      10.50                      
   LYS  61 A    10.72      10.50                      
   LYS  88 A    10.14      10.50                      
   LYS  90 A    10.56      10.50                      
   LYS  97 A    10.37      10.50                      
   LYS 100 A    11.37      10.50                      
   LYS 102 A    10.76      10.50                      
   LYS 137 A    10.33      10.50                      
   LYS 236 A    10.69      10.50                      
   LYS 269 A    11.49      10.50                      
   ARG   4 A    12.41      12.50                      
   ARG  40 A    14.21      12.50                      
   ARG  60 A    12.46      12.50                      
   ARG  76 A    12.59      12.50                      
   ARG 105 A    13.13      12.50                      
   ARG 131 A    15.69      12.50                      
   ARG 188 A    12.33      12.50                      
   ARG 217 A    12.15      12.50                      
   ARG 222 A    12.45      12.50                      
   ARG 279 A    12.06      12.50                      
   ARG 298 A    12.61      12.50                      
   N+    1 A     7.91       8.00  
```

We can see that all the residues are in their usual protonation state, therefore we do not need to modify residue names in the PDB. If you perform this step using the [H++ server](http://biophysics.cs.vt.edu), it will give you the same information and also a new PDB file with the necessary changes applied. 

> **Histidines** are tricky residues, they have three possible protonation states: HID, HIE and HIP. HIP is the protonated residue. HID and HIE correspond to the neutral histidine protonated in *delta* or *epsilon* position. In solution, the most common conformer is HIE but always check your structure before assuming it will be HIE. 

![H163 hbond DWS](./images/H163.png)
**WIP**: REDO this figure. Pymol session in the webinar. 

Last but not least, we need to clean the PDB file before feeding it to AmberTools. So we are going to use the first AmberTools tool: ```pdb4amber``` which cleans/prepares your PDB file. 

```
pdb4amber -i protein.pdb -o protein4amber.pdb  
```

It returns the following output:

```
==================================================
Summary of pdb4amber for: protein.pdb
===================================================

----------Chains
The following (original) chains have been found:
A

---------- Alternate Locations (Original Residues!))

The following residues had alternate locations:
VAL_73
ARG_217
ASN_221
-----------Non-standard-resnames


---------- Mising heavy atom(s)

None
The alternate coordinates have been discarded.
Only the first occurrence for each atom was kept.
```

```pdb4amber``` informs us about the duplicity of several sidechains, disulphide bonds and missing heavy atoms. It does not check protonation states for protein residues and therefore we **ALWAYS** must run a pKa calculation.  

#### Assessing the protonation states of the ligand

We have check the protonation state of the protein but we have omited the ligand. We have to check the pKa of the ligand. There are several options to do so, the most reliable one if available is [PubChem](https://pubchem.ncbi.nlm.nih.gov), where you can find experimental pKa values. 

In this example, there are no pKa values (nor experimental nor calculated) reported for this molecule, so we need to use another resource. I like [Marvin Sketch](https://chemaxon.com/products/marvin): it is *free* to use, gives calculated pKa values and a nice visualisation of the chemical species at a certain pH.  

![pKa calculation GWS](./images/pKa_gws.png)

After assessing the protonation state of the GWS ligand using Marvin Sketch, we can proceed to protonate it. The easiest way to protonate it is using [OpenBabel](http://openbabel.org).

```
/home/d118/d118/shared/amber20/miniconda/bin/obabel -ipdb GWS.pdb -opdb -O GWS.H.pdb -h 
```

You should always check the result of this step and ammend it if needed. In this case, it gives the correct protonation state:

![protonated GWS ligand](./images/GWS.png)

***

## 2. Parameterising your ligand

This step is not trivial and there are many flavours and ways to do it. In this example, we are going to use a simple approach because we aim to use a QM description of the ligand in the QM/MM runs. If you have trouble parameterising your ligands here are some useful links:

- [AMBER tutorial on simple ligand parameterisation](https://ambermd.org/tutorials/basic/tutorial4/index.htm)
- [AMBER tutorial on advanced ligand parameterisation](https://ambermd.org/tutorials/advanced/tutorial1/section1.htm)
- [Parameterisation of dihedral angles](http://www.ub.edu/cbdd/?q=content/small-molecule-dihedrals-parametrization)
- [AMBER parameter database](http://research.bmh.manchester.ac.uk/bryce/amber/)
- [Paramfit tutorial](http://ambermd.org/tutorials/advanced/tutorial23/)

In this example, we will make use of the ```antechamber```and ```parmchk2``` tools from AmberTools package to generate AMBER force field parameters. We will use the "**general AMBER force field 2 [(GAFF2)](http://ambermd.org/antechamber/gaff.html)**". This force field was designed to provide atom types andf atom parameters needed to parameterise most pharmaceutical molecules and is compatible with the traditional AMBER force fields. To assign partial charges, we are going to use **AM1-BCC2 charge method**. It is an unexpensive and fast method to calculate partial charges and any limitations of this method would be fixed by the QM treatment of the ligand in the QM/MM simulation.  

```antechamber``` allows the rapid generation of topology files for use with the AMBER simulation programs. It runs a series of other AmberTools programs (```sqm```, ```divcon```, ```atomtype```, ```am1bcc```, ```bondtype```, ```espgen```, ```respgen``` and ```prepgen```) to be able to:

- Automatically identify bond and atom types
- Judge atomic equivalence
- Generate residue topology files
- Find missing force field parameters and supply reasonable suggestions

We will use ```antechamber``` to assign atom types to the GWS ligand and calculate a set of point charges. We will have to specify GAFF2 force field (**-at gaff2**, more information on the parameters here: ```$AMBERHOME/dat/leap/parm/gaff.dat```), the charge method (**-c bcc**), the net charge of the molecule (**-nc 0**) and the name of the new residue generated (We are going to keep GWS: **-rn GWS**). We are going to output a mol2 file containing the atomtypes and the point charges.  

``` 
antechamber -i GWS.H.pdb -fi pdb -o GWS.mol2 -fo mol2 -c bcc -nc 0 -rn GWS -at gaff2
```

The output goes as follows: 

```
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

Running: /Users/sllabres/miniconda3/envs/py3/bin/sqm -O -i sqm.in -o sqm.out
```

The MOL2 file contains tridimensional information of the molecule as well as atomtype and point charges. You can find more information on the MOL2 format [here](http://chemyang.ccnu.edu.cn/ccb/server/AIMMS/mol2.pdf). The resulting MOL2 file is shown below:

```
<TRIPOS>MOLECULE
GWS 
   34    35     1     0     0
SMALL
bcc


@<TRIPOS>ATOM
      1 N1           7.5340     0.3440    17.5810 nb      1001 GWS      -0.643000
      2 C4          13.4880    -0.5140    24.0620 c3      1001 GWS      -0.075900
      3 C5          13.9760    -1.6820    23.2210 c3      1001 GWS      -0.078400
      4 C6          13.4570    -1.5250    21.8030 c3      1001 GWS      -0.075900
      5 C7          11.9470    -1.3430    21.7400 c3      1001 GWS      -0.075900
                                       ...
```

> **TIP:** The GAFF and GAFF2 atom types are in lower case. This is the mechanism by which the GAFF force field is kept independent of the macromolecular AMBER force fields. All of the traditional AMBER force fields use uppercase atom types. In this way the GAFF and traditional force fields can be mixed in the same calculation.

While the most likely combinations of bond, angle and dihedral parameters are defined in the parameter file it is possible that our molecule might contain combinations of atom types for bonds, angles or dihedrals that have not been parameterised. If this is the case, we will have to specify any missing parameters before we can create our prmtop and inpcrd files in Leap. 

We will use ```parmchk2``` to test if all the parameters we require are available.

```
parmchk2 -i GWS.mol2 -f mol2 -o GWS.frcmod -s gaff2
```

```parmchk2``` generates a parameter file that can be loaded into Leap in order to add missing parameters. It contains all of the missing parameters. If possible, it will fill in these missing parameters by analogy to a similar parameter. Let's look at the generated GWS.frcmod file: 

```
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

```

> **TIP**: You should check these parameters carefully before running a simulation. If antechamber can't empirically calculate a value or has no analogy it will either add a default value that it thinks is reasonable or alternatively insert a place holder (with zeros everywhere) and the comment "**ATTN: needs revision**". In this case you will have to manually parameterise this yourself. See the links at the beginning of the section.  


***


## 3. System set up

Once we have parameters for our ligand and a prepared protein PDB file, we can start building our system and creating topology and coordinate files for AMBER. To create the mentioned files we will use ```Leap```. there are two versions of leap: the GUI version ```xLeap``` and the terminal version ```tLeap```. ```xLeap``` is better to learn Leap because you can visualise the result of each command you try. However, ```xleap``` is quite buggy and it can be frustrating before you get used to all its quirks. 

In this session, we are going to use ```tleap``` and I am going to walk you through every command we use. If you type tleap you should enter in the tleap command-line. If you want to check what you did in previous session of Leap, Leap is quite verbose and informs you of everything it does as well writing it into the **leap.log** file. 

We have prepared a leap script to set up our system and we are going to explain briefly.  

We are going to use default force fields (amberff14SB, tip3p and GAFF2) in this tutorial, so we can use the leaprc scripts already included in AMBER: 
- leaprc.protein.ff14SB for the protein
- leaprc.gaff2 for the ligand molecule
- leaprc.water.tip3p for the water molecules.

```
# Load force field parameters
source leaprc.protein.ff14SB
source leaprc.gaff2
source leaprc.water.tip3p
```

Now we have to include the following units: protein, ligand and the water molecules in ```tLeap```. We are going to start with the ligand using the two files we have created before: **GWS.frcmod** and **GWS.mol2** (mol2 file already includes the coordinates of the ligand). After, we load the coordinates for the protein and the water molecules with the ```loadPDB``` command.

```
# Load extraparameters for the ligand
loadamberparams GWS.frcmod

# Load protein, ligand and water molecules
protein = loadPDB protein4amber.pdb
GWS = loadmol2 GWS.mol2
waters = loadPDB waters.pdb
```

From the very beginning of this series, we have splitted the system into these three pieces, now we join them again with the ```combine``` command. 

> **TIP**: It is good practice to check the resulting structure of this combination step (```savePDB``` command to save a PDB file) and to perform a sanity check on the unit we have created with ```check``` command: it determines the charge of the system, close contacts (it is normal to have them after adding hydrogen atoms) and checks if there are any missing parameters. 

```
# Build system
system = combine {protein GWS waters}
savepdb system  system.dry.pdb

check system
```

If the unit is OK, we can continue! Now we need to solvate it and add counter ions to neutralise the system. 

To solvate the system, we can use 2 different commands: ```solvateOct``` and ```solvateBox```. These commands differ in the shape of the resulting simulation box: the first command creates an octaedric solvation box and the second a cubic solvation box. Since octaedric boxes are smaller and compatible with globular proteins, we are going to use ```solvateOct```. 

Last but not least, we need to neutralise the system by adding counterions. It is easy and simple to use the ```addions2``` command like this: ```addions2 system Na+ 0```.

```
# Solvate
solvateOct system TIP3PBOX 12 iso

# Neutralise
addions2 system Cl- 0
addions2 system Na+ 0
```

Now we can proceed to save our topology and coordinate files with ```saveamberparm system system.parm7 system.rst7```. Before doing anything else, Leap checks that the unit it is OK and then proceeds to build the topology and write the coordinates. Finally it prints a summary of the ends of the system molecules. We have a SER in the N terminal end and THR on the C terminal end of the protein, a ligand named GWS and 20097 water molecules. 

```
# Save AMBER input files
savePDB system system.pdb
saveAmberParm system system.parm7 system.rst7

quit
```

To execute it:

```
tleap -f leapin
```

You can find more information about Leap usage in the [AMBER documentation](https://ambermd.org/doc12/Amber20.pdf). Other Leap related FAQ can be found here: http://ambermd.org/Questions/leap.html

***

# COFFEE BREAK

***

Before running any QM/MM simulation, we must **ALWAYS** equilibrate the system using only molecular mechanics (MM). To do so we must minimise the structure first to avoid bad contacts that might blow-up our simulation. 

To do so, we are going to minimise and equilibrate the system using ```sander``` tool from AmberTools suite. Commented input files and ARCHER submission scripts will be provided to make these steps easier to follow. You can find more information on the sander input options in [Chapter 19](https://ambermd.org/doc12/Amber20.pdf) of the Amber10 manual.

***

## 4. System minimisation (MM)

```sander``` needs the initial coordinates (```-c system.rst7```), topology (```-p system.parm7```) and input file (```-i sander_min.in```). Also, we can specify the name of output files such as output file (```-o```) and final coordinates (```-r```). 

```
sander -O -i sander_min.in -o min_classical.out -p system.parm7 -c system.rst7 -r system.min.rst7
```

We will minimise in 4500 steps using two different methods: steepest descent and conjugate gradient. The sander input file looks like this:

```
Minimisation of system 
 &cntrl
  imin=1,        ! Perform an energy minimization.
  maxcyc=4500,   ! The maximum number of cycles of minimization. 
  ncyc=2000,     ! The method will be switched from steepest descent to conjugate gradient after NCYC cycles.
 /
```

The final results are here: **TO UPDATE**

```
   NSTEP       ENERGY          RMS            GMAX         NAME    NUMBER
   4500      -2.6498E+05     1.9059E-01     2.1137E+01     CG       4025

 BOND    =    20234.2575  ANGLE   =      606.0604  DIHED      =     3423.4910
 VDWAALS =    49177.3328  EEL     =  -351445.8065  HBOND      =        0.0000
 1-4 VDW =      952.7618  1-4 EEL =    12074.5353  RESTRAINT  =        0.0000
```

***

## 5. Thermalisation and Pressure equilibration (MM)

After the minimisation step, we will heat the system up to the target temperature 310K (36°C) at constant volume and then after equilibrate the pressure and density of the system at contant pressure.  

**Thermalisation**

We will use the input file **sander_heat.in** to thermalisation, where we use a temperature ramp that will slowly increase the temperature and allow to allow the system to accomodate.

```
Heating ramp from 100K to 300K 
 &cntrl
  imin=0,                   ! Run molecular dynamics.
  ntx=1,                    ! Initial file contains coordinates, but no velocities.
  irest=0,                  ! Do not restart the simulation
  nstlim=10000,             ! Number of MD-steps to be performed.
  dt=0.002,                 ! Time step (ps)
  ntf=2, ntc=2,             ! Constrain lengths of bonds having hydrogen atoms (SHAKE)
  tempi=0.0, temp0=300.0,   ! Initial and final temperature
  ntpr=100, ntwx=100,       ! Output options
  cut=8.0,                  ! non-bond cut off
  ntb=1,                    ! Periodic conditiond at constant volume
  ntp=0,                    ! No pressure scaling
  ntt=3, gamma_ln=2.0,      ! Temperature scaling using Langevin dynamics with the collision frequency in gamma_ln (ps−1)
  ig=-1,                    ! seed for the pseudo-random number generator will be based on the current date and time.
  nmropt=1,                 ! NMR options to give the temperature ramp.
 /
&wt type='TEMP0', istep1=0, istep2=9000, value1=0.0, value2=300.0 /
&wt type='TEMP0', istep1=9001, istep2=10000, value1=300.0, value2=300.0 /
&wt type='END' /
```

The final results are here: **TO UPDATE**

```
 NSTEP =    10000   TIME(PS) =      20.000  TEMP(K) =   298.87  PRESS =     0.0
 Etot   =   -156391.2390  EKtot   =     39292.6561  EPtot      =   -195683.8951
 BOND   =       949.1198  ANGLE   =      2397.4235  DIHED      =      3895.6936
 1-4 NB =      1090.0599  1-4 EEL =     12054.3523  VDWAALS    =     24789.0486
 EELEC  =   -240859.5929  EHBOND  =         0.0000  RESTRAINT  =         0.0000
 Ewald error estimate:   0.3083E-04
```

***

**Pressure equilibration**

Then, we will use the input file **sander_equil.in** to equilibrate the density of the system:

```
Density equilibration
&cntrl
  imin= 0,                       ! Run molecular dynamics.
  nstlim=25000,                  ! Number of MD-steps to be performed.
  dt=0.002,                      ! Time step (ps)
  irest=1,                       ! Restart the simulation
  ntx=5,                         ! Initial file contains coordinates and velocities.
  ntpr=100, ntwx=100, ntwr=100,  ! Output options
  cut=8.0,                       ! non-bond cut off
  temp0=298,                     ! Temperature
  ntt=3, gamma_ln=3.0,           ! Temperature scaling using Langevin dynamics with the collision frequency in gamma_ln (ps−1)
  ntb=2,                         ! Periodic conditiond at constant pressure
  ntc=2, ntf=2,                  ! Constrain lengths of bonds having hydrogen atoms (SHAKE)
  ntp=1, taup=2.0,               ! Pressure scaling
  iwrap=1, ioutfm=1,             ! Output trajectory options
/
```

The final results are here: **TO UPDATE**

```
 NSTEP =    10000   TIME(PS) =      20.000  TEMP(K) =   298.87  PRESS =     0.0
 Etot   =   -156391.2390  EKtot   =     39292.6561  EPtot      =   -195683.8951
 BOND   =       949.1198  ANGLE   =      2397.4235  DIHED      =      3895.6936
 1-4 NB =      1090.0599  1-4 EEL =     12054.3523  VDWAALS    =     24789.0486
 EELEC  =   -240859.5929  EHBOND  =         0.0000  RESTRAINT  =         0.0000
 Ewald error estimate:   0.3083E-04
```

## 6. Ammend the forcefield before running QM/MM runs

We need to ammend the forcefield before starting QM/MM calculations because the amber forcefiled does not have Lennard-Jones parameters for all the hydrogen atoms. To do that we need to modify the topology with the tool ```parmed```. 

```parmed system.parm7
printDetails @4685
changeLJSingleType :WAT@H1 0.3019 0.047
changeLJSingleType :WAT@H2 0.3019 0.047
printDetails @11
changeLJSingleType :SER@HG  0.3019 0.047
printDetails @549
changeLJSingleType :TYR@HH   0.3019 0.047
outparm systemx_LJ_mod.parm7
quit
```

This step prevents creating unphysical interactions between the hydrogen atoms of water and serine and QM subsystem. After topology correction the thermostat should stay stable during the QMMM calculation. 

## 7. QM/MM Production runs



```
CP2K input
```
