---
layout: page
title: CP2K Input Files
---

## Overview of the `CP2K` input

The CP2K input file, `cp2k_qmmm_equil.inp`, is divided in different sections that we will explain separately:

### `&GLOBAL` section
It contains general information regarding which kind of simulation to perform.

~~~
&GLOBAL
  PROJECT MD_QMMM                ! Name of the calculation
  PRINT_LEVEL LOW                ! Verbosity of the output
  RUN_TYPE MD                    ! Calculation type: MD
&END GLOBAL
~~~
{: .source}


### `&FORCE_EVAL` section

It contains the parameters needed to calculate energy and forces and describe the system you want to analyze. It has several important subsections:

#### `&DFT`, which specifies the parameter needed by LCAO DFT programs

~~~
&FORCE_EVAL
  METHOD QMMM                    ! Hybrid quantum classical
  STRESS_TENSOR ANALYTICAL       ! Compute the stress tensor analytically (if available).

  &DFT                           ! Parameter needed by LCAO DFT programs
    CHARGE 0                     ! The charge of the QM system

    &QS                          ! parameters needed to set up the Quickstep framework
      METHOD PM3                 ! Specifies the electronic structure method that should be employed
      &SE                        ! Parameters needed to set up the Semi-empirical methods
         &COULOMB                ! parameters for the evaluation of the COULOMB term
           CUTOFF [angstrom] 10.0
         &END
         &EXCHANGE               ! parameters for the evaluation of the EXCHANGE and core Hamiltonian terms
           CUTOFF [angstrom] 10.0
         &END
      &END
    &END QS

    &SCF                         ! Parameters needed to perform an SCF run
      MAX_SCF 30                 ! Maximum number of SCF iterations
      EPS_SCF 1.0E-6             ! Target accuracy for the SCF convergence
      SCF_GUESS ATOMIC           ! initial guess for the wavefunction: Generate an atomic density using the atomic code
      &OT                        ! options for the orbital transformation (OT) method
        MINIMIZER DIIS           ! Minimizer to be used with the OT method
        PRECONDITIONER FULL_SINGLE_INVERSE
      &END
      &OUTER_SCF                 ! parameters controlling the outer SCF loop
        EPS_SCF 1.0E-6           ! Target gradient of the outer SCF variables
        MAX_SCF 10               ! Maximum number of outer loops
      &END
    &END SCF

  &END DFT
~~~
{: .source}



#### `&MM`, which includes the parameters to run a MM calculation (such as the forcefield). Here we have to manually edit the &MM subsection PARM_FILE_NAME and add the modified topology name system_LJ_mod.parm7.

~~~
  &MM                            ! Parameters to run a MM calculation
    &FORCEFIELD                  ! Set up a force_field for the classical calculations
      PARMTYPE AMBER
      PARM_FILE_NAME system_LJ_mod.parm7
      &SPLINE                    ! Parameters to set up the splines used in the nonboned interactions
        EMAX_SPLINE 1.0E8        ! Maximum value of the potential up to which splines will be constructed
        RCUT_NB [angstrom] 10    ! Cutoff radius for nonbonded interactions
      &END SPLINE
    &END FORCEFIELD
    &POISSON
      &EWALD
      ! Ewald parameters controlling electrostatic
        EWALD_TYPE SPME          ! Type of ewald
        ALPHA .40                ! Alpha parameter associated with Ewald (EWALD|PME|SPME)
        GMAX 80                  ! Number of grid points (SPME and EWALD)
      &END EWALD
    &END POISSON
  &END MM
~~~
{: .source}


#### `&SUBSYS`, which specifies the information of the system: coordinates, topology, molecules and cell. Here we have to manually edit the following sections:
  - `&CELL` subsection with the simulation box size: ABC and ALPHA_BETA_GAMMA.
  - `&TOPOLOGY` subsection with the Amber topology and coordinate files.
  - We must add any atomtype that CP2K doesn’t recognise such as the counterions (Na+ and NS atom type) and specify their element.

~~~
  &SUBSYS                        ! a subsystem: coordinates, topology, molecules and cell
    &CELL                        !Set box dimensions here
      ABC [angstrom] XXX XXX XXX
      ALPHA_BETA_GAMMA XXX XXX XXX
    &END CELL
    &TOPOLOGY                    ! Topology for classical runs
      CONN_FILE_FORMAT AMBER
      CONN_FILE_NAME system_LJ_mod.parm7
      COORD_FILE_FORMAT CRD
      COORD_FILE_NAME system.equil0.rst7
    &END TOPOLOGY
    !NA+ is not recognized by CP2K, so it is necessary to define it here using KIND
    &KIND NA+
     ELEMENT Na
    &END KIND
    &KIND NS
     ELEMENT N
    &END KIND
  &END SUBSYS
~~~
{: .source}

#### `&QMMM`, which contains all the information on the QM/MM calculation. This includes the definition of QM region which we must specify manually.

We define the QM region here

~~~
  &QMMM                            ! Input for QM/MM calculations
    ECOUPL COULOMB                 ! type of the QM - MM electrostatic coupling
    &CELL                          ! Set box dimensions here
      ABC 40 40 40
      ALPHA_BETA_GAMMA 90 90 90
    &END CELL
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
  &END QMMM
&END FORCE_EVAL
~~~
{: .source}

### `&MOTION` section

It contains all the parameters for the MD run (or motion of nuclei).

~~~
&MOTION                            ! Parameter for the motion of the nuclei
  &MD                              ! set of parameters needed perform an MD run
  ENSEMBLE NPT_I                   ! Ensemble/integrator that you want to use for MD
  TIMESTEP [fs] 0.5                ! Time step
  STEPS    200                     ! Number of MD steps to perform
  TEMPERATURE 298                  ! Temperature in K
  &BAROSTAT                        ! Parameters of barostat.
    TIMECON [fs] 100
    PRESSURE [bar] 1.0
  &END BAROSTAT
  &THERMOSTAT                      ! Parameters of Thermostat.
    REGION GLOBAL                  ! region each thermostat is attached to.
    TYPE CSVR                      ! canonical sampling through velocity rescaling
    &CSVR
      TIMECON [fs] 10.
    &END CSVR
  &END THERMOSTAT
  &END MD
  &PRINT                           ! Printing properties during an MD run
    &RESTART                       ! Printing of restart files
      &EACH                        ! A restart file will be printed every 2000 md steps
        MD 2000
      &END
    &END
    &TRAJECTORY                    ! Controls the output of the trajectory
      FORMAT DCD                   ! Format of the output trajectory is DCD
      &EACH                        ! New trajectory frame will be printed each 100 md steps
        MD 100
      &END
    &END
    &RESTART_HISTORY               ! Controls printing of unique restart files during the run keeping all of them.
      &EACH                        ! A new restart file will be printed every 5000 md steps
        MD 5000
      &END
    &END
    &CELL                          ! Controls the output of cell dimensions
      &EACH                        ! Cell dimensions will be printed each 100 md steps
        MD 100
      &END
    &END
  &END PRINT
&END MOTION
~~~
{: .source}


