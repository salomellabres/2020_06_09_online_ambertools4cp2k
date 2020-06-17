---
title: Adding missing LJ parameters and recentering coordinates
teaching: 10
exercises: 0
questions:
- "Does the MM forcefield have all the parameters to run a QM/MM simulation?"
- "How do I modify an Amber topology?"
objectives:
- "Find missing parameters in the AMBER forcefields using `parmed`."
- "Use `parmed` to change parameters in AMBER7 topologies."
- "Use `cpptraj` to recenter the system in the simulation box."
keypoints:
- "In the MM forcefields, hydrogen atoms do not have Lennard-Jones parameters." 
- "Adding missing Lennard-Jones parameters to the topology will prevent having unphysical interactions int the QM/MM boundary."
- "You can modify amber topologies using `parmed`."
- "You should recenter the system to avoid having a split QM region in the QM/MM."
- "`cpptraj` is the main analysis and postprocess trajectory tool for Amber."
- "Using `autoimage` in `cpptraj` recenters the solute and fixes most of the PBC problems of the simulation."
---

## Adding Lennard-Jones parameters using `parmed`

By construction, some hydrogen atoms do not have Lennard-Jones parameters in the AMBER forcefields. However, before running any QM/MM simulation, we need to provide those parameters. In particular, we are going to add the values of the hydrogen atom that correspond to the hydroxyl in the GAFF forcefield to TIP3P water model and to the hydroxyl groups from serine and tyrosine residues. 

To do that, we will use the `parmed` tool of the AmberTools package. The usage of this program is similar to LEap. First, we load the topology file in `parmed` and then we do have a lot of options to modify it (More information [here](http://parmed.github.io/ParmEd/html/index.html)).

~~~
parmed system.parm7 
~~~
{: .language-bash} 

We can check if there are missing parameter by printing an atom index (4685 corresponds to the hydrogen atom of the first water molecule) using the `printDetails` command. Here we can see that the LJ Radius and LJ Depth values are set to 0.

~~~
printDetails @4686
~~~
{: .source}

~~~
The mask @4686 matches 1 atoms:

   ATOM    RES  RESNAME  NAME  TYPE   At.#   LJ Radius    LJ Depth      Mass    Charge GB Radius GB Screen
   4686    776      WAT    H1    HW      1      0.0000      0.0000    1.0080    0.4170    0.8000    0.8500
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

`parmed` prints the following information everytime you change a LJ value.

It is very important to do this before running your QM/MM simulations. This step prevents creating unphysical interactions between the hydrogen atoms without LJ parameters and the QM subsystem. After this topology correction, the thermostat should stay stable during the QM/MM calculation. 

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

>## TIP 
>
>You can also run `parmed` with an parmed.in input file similarly as we did with `tleap`.
>~~~
>parmed system.parm7 -i parmed.in 
>~~~
>{: .language-bash}
>
{: .callout}
***

## Centering the solutes in the simulation box

Last but not least, we need to recenter the protein to the center of the simulation box using `cpptraj`. `cpptraj` is the trajectory analysis tool of the AmberTools package and it can do a lot of different analysis (you can find more information [here](https://amber-md.github.io/cpptraj/CPPTRAJ.xhtml)). In this tutorial, we are only going to use it to recenter the system and make sure the coordinate file is in the correct format.

First, we need to load the topology file:

~~~
cpptraj -p system_LJ_mod.parm7
~~~
{: .language-bash}

Then, we provide `cpptraj` three different commands: 
- `trajin`: input trajectories. It can be repeated as many times as trajectory file you want to analyse. 
- `trajout`: output trajectories specifying the desired format.
- `autoimage`: recenters protein coordinates and fixes the simulation box around it. 

After you have typed all your analysis commands you have to type `go` or `run` to actuallt make cpptraj run the analysis. 

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
- `system_LJ_mod.parm7`
- `system.equil0.rst7`


