---
layout: page
title: Amber Input Files
---

## Overview of `sander` input files

### Energy Minimisation:

The minimisation input file, `sander_min.in`, will include 4000 steps using two different methods: 2000 of [gradient descent](https://en.wikipedia.org/wiki/Gradient_descent) and 2000 of [conjugate gradient](https://en.wikipedia.org/wiki/Conjugate_gradient_method). The sander input file looks like this:

~~~
Minimisation of system
 &cntrl
  imin=1,        ! Perform an energy minimization.
  maxcyc=4000,   ! The maximum number of cycles of minimization.
  ncyc=2000,     ! The method will be switched from steepest descent to conjugate gradient after NCYC cycles.
 /
~~~
{: .source}

### Thermalisation (NVT):

We will use the input file `sander_heat.in` to gradually increase the temperature of the system up to our target value over the first 30 ps. To help our system accomodate to this change, we will use a temperature ramp in which we control the temperaure increase per timestep.

~~~
Heating ramp from 0K to 300K
 &cntrl
  imin=0,                   ! Run molecular dynamics.
  ntx=1,                    ! Initial file contains coordinates, but no velocities.
  irest=0,                  ! Do not restart the simulation
  nstlim=15000,             ! Number of MD-steps to be performed.
  dt=0.002,                 ! Time step (ps)
  ntf=2, ntc=2,             ! Constrain lengths of bonds having hydrogen atoms (SHAKE)
  tempi=0.0, temp0=298.0,   ! Initial and final temperature
  ntpr=500, ntwx=500,       ! Output options
  cut=8.0,                  ! non-bond cut off
  ntb=1,                    ! Periodic conditiond at constant volume
  ntp=0,                    ! No pressure scaling
  ntt=3, gamma_ln=2.0,      ! Temperature scaling using Langevin dynamics with the collision frequency in gamma_ln (ps−1)
  ig=-1,                    ! seed for the pseudo-random number generator will be based on the current date and time.
  nmropt=1,                 ! NMR options to give the temperature ramp.
 /
&wt type='TEMP0', istep1=0, istep2=12000, value1=0.0, value2=298.0 /
&wt type='TEMP0', istep1=12001, istep2=15000, value1=298.0, value2=298.0 /
&wt type='END' /
~~~
{: .source}

### Density equilibration (NPT):

The input file `sander_equil.in` is used to equilibrate the density of the system for 30ps:

~~~
Density equilibration
&cntrl
  imin= 0,                       ! Run molecular dynamics.
  nstlim=15000,                  ! Number of MD-steps to be performed.
  dt=0.002,                      ! Time step (ps)
  irest=1,                       ! Restart the simulation
  ntx=5,                         ! Initial file contains coordinates and velocities.
  ntpr=500, ntwx=500, ntwr=500,  ! Output options
  cut=8.0,                       ! non-bond cut off
  temp0=298,                     ! Temperature
  ntt=3, gamma_ln=3.0,           ! Temperature scaling using Langevin dynamics with the collision frequency in gamma_ln (ps−1)
  ntb=2,                         ! Periodic conditiond at constant pressure
  ntc=2, ntf=2,                  ! Constrain lengths of bonds having hydrogen atoms (SHAKE)
  ntp=1, taup=2.0,               ! Pressure scaling
  iwrap=1, ioutfm=1,             ! Output trajectory options
/
~~~
{: .source}

