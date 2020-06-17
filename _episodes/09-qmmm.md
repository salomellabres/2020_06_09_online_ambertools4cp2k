---
title: QM/MM monitorisation runs
teaching: 25
exercises: 0
questions:
- "What is CP2K?"
- "What is the first step of a QM/MM simulation?"
objectives:
- "Choose the QM region effectively"
- "Overview of a simple CP2K input file."
- "Submit a simple CP2K monitorisation calculation."
keypoints:
- "Choosing the QM region is in not trivial."
- "You should also equilibrate the system using QM/MM ensemble before running your QM/MM production runs."
- "Instabilities between the QM and the MM regions are likely to show up in this monitorisation run."
---

## Moving from MM to QM/MM...

We want to address a few points before we start running a QM/MM simulation.

#### What is CP2K?

[CP2K](https://www.cp2k.org) is a quantum chemistry and solid state physics software package that can perform atomistic simulations of solid state, liquid, molecular, periodic, material, crystal, and biological systems. CP2K provides a general framework for different modeling methods such as DFT using the mixed Gaussian and plane waves approaches GPW and GAPW. Supported theory levels include DFTB, LDA, GGA, MP2, RPA, semi-empirical methods (AM1, PM3, PM6, RM1, MNDO, …), and classical force fields (AMBER, CHARMM, …). CP2K can do simulations of molecular dynamics, metadynamics, Monte Carlo, Ehrenfest dynamics, vibrational analysis, core level spectroscopy, energy minimization, and transition state optimization using NEB or dimer method.

You can find more information on its usage [here](https://manual.cp2k.org/). 

#### Which theory level are we going to use?

We will use of [PM3](https://en.wikipedia.org/wiki/PM3_(chemistry)), a semiempirical method, to define the QM region and the amended Amber forcefield to define the MM subsystem. We will run 500fs (1000steps * 0.5 fs) of simulation on an ARCHER node. 

#### How do we define the QM region?

In this example, we are going to define the ligand GWS as the QM region. This implies 34 QM atoms embedded within a MM envinronment. Despite this is a simple and convenient example, it is not uncommon. It also has the advantage that we do not need to split any covalent bonds between the QM subsystem and the MM subsystem. This topic will be covered in future sessions. 

***

## Running a QM/MM monitorisation run

After we have equilibrated the system using molecular mechanics, we need to equilibrate the system running the chosen QM/MM approach. It is important to run a monitorisation run before any production run to make sure our system behaves as expected.  

This step might take some time, so we will also submit the calculation to the ARCHER short queue:

~~~
qsub send_qmmm_equil.pbs
~~~
{: .language-bash}

~~~
!/bin/bash --login
#PBS -N QM_equil

#PBS -q short
#PBS -l select=1
#PBS -l walltime=00:20:00

# Make sure you change this to your budget code
#PBS -A y14-amber2020

module load cp2k/7.1

# Move to directory that script was submitted from
export PBS_O_WORKDIR=$(readlink -f $PBS_O_WORKDIR)
cd $PBS_O_WORKDIR

aprun -n 24 cp2k.popt -i cp2k_qmmm_equil.inp > cp2k_qmmm_equil.out
~~~
{: .source}

***

## Overview of the CP2K input
 
In the meantime, we will go over the CP2K input file, `cp2k_qmmm_equil.inp`. It is divided in different sections that we will explain separately:

#### `&GLOBAL` section: 

It contains general information regarding which kind of simulation to perform. 

#### `&FORCE_EVAL` section: 

It contains the parameters needed to calculate energy and forces and describe the system you want to analyze. It has several important subsections:
  - `&DFT`, which specifies the parameter needed by LCAO DFT programs
  - `&MM`, which includes the parameters to run a MM calculation (such as the forcefield). Here we have to manually edit the &MM subsection PARM_FILE_NAME and add the modified topology name system_LJ_mod.parm7.
  -  `&SUBSYS`, which specifies the information of the system: coordinates, topology, molecules and cell. Here we have to manually edit the following sections:
     - `&CELL` subsection with the simulation box size: ABC and ALPHA_BETA_GAMMA.
     - `&TOPOLOGY` subsection with the Amber topology and coordinate files.
     - We must add any atomtype that CP2K doesn’t recognise such as the counterions (Na+ and NS atom type) and specify their element.
  - `&QMMM`, which contains all the information on the QM/MM calculation. This includes the definition of QM region which we must specify manually.

#### `&MOTION` section: 
It contains all the parameters for the MD run (or motion of nuclei). 

***

## Sections to modify from the CP2K input file

We have shared a working version of the CP2K input file ( `cp2k_qmmm_equil.inp`) with some values from a previous run, but it is important that you add the following information from your input files.

####  Amber topology and coordinate files

We have to add the name of the amber topology and coordinate files in the sections: 

`&FORCE_EVAL > &SUBSYS > &TOPOLOGY`

~~~
  &SUBSYS                        ! a subsystem: coordinates, topology, molecules and cell 
    . . .
    &TOPOLOGY                    ! Topology for classical runs
      CONN_FILE_FORMAT AMBER     
      CONN_FILE_NAME system_LJ_mod.parm7
      COORD_FILE_FORMAT CRD
      COORD_FILE_NAME system.equil0.rst7
    &END TOPOLOGY  
    . . .
  &END SUBSYS
~~~
{: .source}

`&FORCE_EVAL > &MM > &FORCEFIELD`

~~~
  &MM    
    . . .                        ! Parameters to run a MM calculation
    &FORCEFIELD                  ! Set up a force_field for the classical calculations
      PARMTYPE AMBER             
      PARM_FILE_NAME system_LJ_mod.parm7   
    &END FORCEFIELD
    . . .
  &END MM
~~~
{: .source}


#### Box size on `&CELL`

It is very important that you use the current size of your simulation box in `&FORCE_EVAL > &SUBSYS > &CELL`. To get it you should do the following:

~~~
tail -1 system.equil0.rst7
~~~
{: .language-bash}

and add the **X Y Z** values in the `ABC` section and the **alpha beta gamma** to the `ALPHA_BETA_GAMMA` section.

#### Atom indices of the QM region.

You have to add the indices of the atoms of the QM region in the section `&FORCE_EVAL > &QMMM > &QM_KIND` separated by element (N, O , C, H, ...). We have provided a completed version of this section by selectiong only the GWS ligand as the QM region. More complex QM regions can be set including part of the protein, but they are beyond the scope of this tutorial. 

~~~
  &QMMM                            ! Input for QM/MM calculations
    ...
    &QM_KIND N                     ! N atoms in the QM region 
      MM_INDEX 1 10 
    &END QM_KIND
    &QM_KIND O                     ! O atoms in the QM region
      MM_INDEX 8 
    &END QM_KIND
    &QM_KIND C                     ! C atoms in the QM region
      MM_INDEX 2 3 4 5 6 7 9 11 12 13 14 15 16
    &END QM_KIND
    &QM_KIND H                     ! H atoms in the QM region
      MM_INDEX 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34
    &END QM_KIND
    ...
  &END QMMM
~~~
{: .source}

***

## CP2K output

CP2K produces the following output: 
- `MD_QMMM-1.restart`: file to restart the calculation if needed. It looks a lot like the CP2K input file. 
- `MD_QMMM-pos-1.dcd`: Trajectory file. DCD format is read in VMD. 
- `MD_QMMM-1.cell`: Time evolution of the box size. 
- `MD_QMMM-1.ener`: Time evolution of important quantities (Kinetic energy, Potential energy, Temperature...)
- `MD_QMMM-RESTART.wfn`: Stored wavefunction to restart the simulation.
- `cp2k_qmmm_equil.out`: output of the CP2K simulation.

If we open the `cp2k_qmmm_equil.out`, the output format for each step of the QM/MM monitorisation is shown here: 

~~~
 Number of electrons:                                                         86
 Number of occupied orbitals:                                                 43
 Number of molecular orbitals:                                                43

 Number of orbital functions:                                                 82
 Number of independent orbital functions:                                     82

 Extrapolation method: ASPC


 SCF WAVEFUNCTION OPTIMIZATION

  ----------------------------------- OT ---------------------------------------
  Minimizer      : DIIS                : direct inversion
                                         in the iterative subspace
                                         using   7 DIIS vectors
                                         safer DIIS on
  Preconditioner : FULL_SINGLE_INVERSE : inversion of
                                         H + eS - 2*(Sc)(c^T*H*c+const)(Sc)^T
  Precond_solver : DEFAULT
  stepsize       :    0.08000000                  energy_gap     :    0.08000000
  eps_taylor     :   0.10000E-15                  max_taylor     :             4
  ----------------------------------- OT ---------------------------------------

  Step     Update method      Time    Convergence         Total energy    Change
  ------------------------------------------------------------------------------
     1 OT DIIS     0.80E-01    0.0     0.00014885       -90.7600029357 -9.08E+01

                                   . . .

  *** SCF run converged in    10 steps ***


  Core-core repulsion energy [eV]:                          13325.24439430390339
  Core Hamiltonian energy [eV]:                             -3727.10071149166743
  Two-electron integral energy [eV]:                       -24066.45132450176970
  Electronic energy [eV]:                                  -15760.32637374255319
  QM/MM Electrostatic energy:                                  -1.27239505853144

  Total energy [eV]:                                        -2469.70560979353240

  Atomic reference energy [eV]:                              2435.26626602509441
  Heat of formation [kcal/mol]:                              -794.19019719481355

  outer SCF iter =    1 RMS gradient =   0.69E-06 energy =        -90.7600151030
  outer SCF loop converged in   1 iterations or   10 steps


 ENERGY| Total FORCE_EVAL ( QMMM ) energy (a.u.):           -317.657151913923542



*******************************************************************************
 ENSEMBLE TYPE                =                                            NPT_I
 STEP NUMBER                  =                                               10
 TIME [fs]                    =                                         5.000000
 CONSERVED QUANTITY [hartree] =                              -0.270581824510E+03

                                              INSTANTANEOUS             AVERAGES
 CPU TIME [s]                 =                        0.75                 0.99
 ENERGY DRIFT PER ATOM [K]    =         -0.775538584027E+00  -0.832073060556E+00
 POTENTIAL ENERGY[hartree]    =         -0.317657151914E+03  -0.317295888439E+03
 TOTAL KINETIC ENERGY[hartree]=          0.521871783418E+02   0.499887046552E+02
 QM KINETIC ENERGY[hartree]   =          0.448103776058E-01   0.441758973655E-01
 TOTAL TEMPERATURE[K]         =                     259.667              248.728
 QM TEMPERATURE[K]            =                     277.451              273.522
 PRESSURE [bar]               =          0.861089290171E+04   0.292238809599E+04
 BAROSTAT TEMP[K]             =          0.571694908987E+05   0.658325595919E+05
 VOLUME[bohr^3]               =          0.299749368381E+07   0.299003810396E+07
 CELL LNTHS[bohr]             =    0.1437398E+03   0.1443978E+03   0.1444178E+03
 AVE. CELL LNTHS[bohr]        =    0.1436205E+03   0.1442779E+03   0.1442979E+03
 *******************************************************************************
~~~
{: .output}


> ## Further information:
> 
> CP2K uses different unit from AmberTools and it can be confusing sometimes. Such as:
> * Bohrs for Length.
>
> Here is a list of [available unit in CP2K](https://manual.cp2k.org/trunk/units.html).  
>
{: .challenge}
