---
title: System set up 
teaching: 10
exercises: 0
questions:
- "How do I create topology and coordinate files?"
objectives:
- "Generate topology and coordinate files with `tleap`."
keypoints:
- "LEap creates topologies and initial coordinates in AMBER format."
- "LEap can generate initial coordinates from scratch if needed."
- "`xLEap` (with GUI) and `tLEap` (through the terminal) are the two flavours of LEap."
- "Your MM system should always be neutral, add counterions to neutralise your system."
- "ALWAYS visualise and check the structures created by LEap."
---

Once we have parameters for our ligand and have prepared a protein PDB file, we can start building our system and creating topology and coordinate files for AMBER. To create the mentioned files we will use `LEaP`. there are two versions of leap: the GUI version `xLEaP` and the terminal version `tLEaP`. `xLEaP` is better to learn Leap because you can visualise the result of each command you try. However, `xLEaP` is quite buggy and it can be frustrating before you get used to all its quirks. 

In this session, we are going to use `tleap` and I am going to walk you through every command we use. If you type `tleap` in your terminal you should enter in the `tLEaP` command-line. If you want to check what you did in previous session of LEaP, LEaP is quite verbose and informs you of everything it does as well as writing it into the **leap.log** file. 

We have prepared a LEaP script to set up our system, which we are going to explain briefly before executing it.

***

## Build your system with `tLEap`  

We are going to use default force fields (amberff14SB, tip3p and GAFF2) in this tutorial, so we can use the leaprc scripts already included in AMBER to load them into `tleap`: 
- `leaprc.protein.ff14SB` for the protein
- `leaprc.gaff2` for the ligand molecule
- `leaprc.water.tip3p` for the water molecules.

~~~
# Load force field parameters
source leaprc.protein.ff14SB
source leaprc.gaff2
source leaprc.water.tip3p
~~~
{: .source}

Now we have to load our three units (protein, ligand and water molecules) into `tLeap`. We are going to start with the ligand by lading the two files we have created before: **GWS.frcmod** and **GWS.mol2** (In this case the mol2 file already includes the correct coordinates of the ligand). After, we load the coordinates for the protein and the water molecules with the `loadPB` command.

~~~
# Load extraparameters for the ligand
loadamberparams GWS.frcmod

# Load protein, ligand and water molecules
protein = loadPDB protein4amber.pdb
GWS = loadmol2 GWS.mol2
waters = loadPDB waters.pdb
~~~
{: .source} 

From the very beginning of this series, we have split the system into these three pieces, now we join them again with the `combine` command. 

~~~
# Build system
system = combine {GWS protein waters}
savepdb system system.dry.pdb

check system
~~~
{: .source} 


> ## TIP
> 
> It is good practice to check the resulting structure of this combination step (`savePDB` command to save a PDB file) and to perform a sanity check on the unit we have created by using the `check` command: this determines the charge of the system, flags up close contacts (it is normal to have them after adding hydrogen atoms), and checks if there are any missing parameters. 
{: .callout}


If the unit is OK, we can continue! Now we need to solvate it and add counterions to neutralise the system. 

To solvate the system, we can use 2 different commands: `solvateOct` and `solvateBox`. These commands differ in the shape of the resulting simulation box: the first command creates a truncated octahedron solvation box and the second a cubic solvation box. Despite truncated octahedric boxes being smaller and compatible with globular proteins, we are going to use `solvateBox` because CP2K only allows orthorombic boxes. 

Last but not least, we need to neutralise the system by adding counterions. To this end, we will add Na+ and Cl- ions until our box reaches the correct charge. This is done with the `addions2` command.

~~~
# Solvate
solvateBox system TIP3PBOX 10 iso

# Neutralise
addions2 system Cl- 0
addions2 system Na+ 0
~~~
{: .source} 

Now we can proceed to save our topology and coordinate files with `saveamberparm system system.parm7 system.rst7`. Before doing anything else, LEaP checks that the unit it is OK and then proceeds to build the topology and write the coordinates. Finally, it prints a summary of the system molecules. We have a ligand named GWS and 13121 water molecules. 
It is important that you include the saveAmberParm command line, otherwise the information we have loaded will be lost once we exit `LEap`.

~~~
# Save AMBER input files
savePDB system system.pdb
saveAmberParm system system.parm7 system.rst7

quit
~~~
{: .source}

We have provided all these commands in a `leapin` file. To execute it you should type the following:

~~~
tleap -f leapin
~~~
{: .language-bash}

***

> ## FURTHER INFORMATION:
>
> You can find more information about LEaP usage in:
> * [AMBER documentation](https://ambermd.org/doc12/Amber20.pdf). 
> * [LEap FAQ](http://ambermd.org/Questions/leap.html).
> 
{: .challenge}
