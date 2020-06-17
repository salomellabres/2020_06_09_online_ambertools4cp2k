---
title: Adding missing LJ parameters and recentering coordinates
teaching: 10
exercises: 0
questions:
- "Do I have all the necessary parameters to run a QM/MM simulation?"
objectives:
- "Discover missing parameters in the AMBER forcefields."
- "Use `parmed` to change parameters in AMBER7 topologies."
- "Use `cpptraj` to recenter the system in the simulation box."
keypoints:
- "Adding missing Lennard-Jones parameters to the topology will prevent having unphysical interactions between the QM and MM subsystems."
---

## Adding Lennard-Jones parameters using `parmed`

Before running any QM/MM simulation, we need to amend the Amber forcefield before because hydrogen atoms in the forcefield do not have Lennard-Jones parameters. In particular, we are going to add the values of the hydrogen atom that correspond to the hydroxyl in the GAFF forcefield to TIP3P water model and to the hydroxyl groups from serine and tyrosine residues. 

To do that, we use the `parmed` tool of the AmberTools package. First, we load the topology file in `parmed` and then we do have a lot of options to modify it (More information [here](http://parmed.github.io/ParmEd/html/index.html)).

~~~
parmed system.parm7
~~~
{: .language-bash} 

We can check if there are missing parameter, by printing an atom index (4685 corresponds to the hydrogen atom of the first water molecule) using the `printDetails` command. Here we can see that the LJ Radius and LJ Depth values are set to 0.  

~~~
printDetails @4685
~~~
{: .source}

~~~
The mask @4685 matches 1 atoms:

   ATOM    RES  RESNAME  NAME  TYPE   At.#   LJ Radius    LJ Depth      Mass    Charge GB Radius GB Screen
   4685    310      WAT    H1    HW      1      0.0000      0.0000    1.0080    0.4170    0.8000    0.8500
~~~
{: .output}


To modify them we need to use the `changeLJSingleType` command on different atom types (:WAT@H1, :SER@HG and :TYR@HH). 

~~~
changeLJSingleType :WAT@H1 0.3019 0.047
changeLJSingleType :WAT@H2 0.3019 0.047
changeLJSingleType :SER@HG 0.3019 0.047
changeLJSingleType :TYR@HH 0.3019 0.047
~~~
{: .source}

`parmed` prints the following information everythime you change a LJ value.

It is very important to do this before running your QM/MM simulations. This step prevents creating unphysical interactions between the hydrogen atoms without LJ paramaters and QM subsystem. After this topology correction, the thermostat should stay stable during the QM/MM calculation. 

~~~
Changing :WAT@H1 Lennard-Jones well depth from 0.0000 to 0.0470 (kal/mol) and radius from 0.0000 to 0.3019 (Angstroms)
~~~
{: . output}

Finally, we save a new topology using the command `outparm` and exit.

~~~
outparm system_LJ_mod.parm7
quit
~~~
{: .source}

***

## Fixing the coordinates of the protein within the simulation box.

Last but not least, we need to recenter the protein to the center of the simulation box using `cpptraj`. `cpptraj` is the trajectory analysis tool of Amber package and it can do a lot of different analysis (you can find more information [here](https://amber-md.github.io/cpptraj/CPPTRAJ.xhtml)). In this tutorial, we are only going to use to recenter the system and make sure the coordinate file is in the correct format:
cpptraj to fix format of the rst7 file.

First, we need to load the topology file:

~~~
cpptraj -p system_LJ_mod.parm7
~~~
{: .language-bash}

Then, we feed cpptraj three different commands: 
- `trajin`: input trajectories
- `trajout`: output trajectories specifying the desired format.
- `autoimage`: recenter protein coordinates and fix the simulation box around it. 

After you ahve typed all your analysis commands you have to type `go` or `run` to actuallt make cpptraj run the analysis. 

~~~
trajin system.equil.rst7
autoimage
trajout system.equil0.rst7 inpcrd
go 
quit
~~~
{: .source}

***

Once we have the corrected topology and coordinates, we can start equilibration our QM/MM system. Before you go to the next step, make sure you have the following files: 
- system_LJ_mod.parm7
- system.equil0.rst7


