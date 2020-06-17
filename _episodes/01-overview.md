---
title: Overview of AmberTools20
teaching: 5
exercises: 0
questions:
- "What is the AmberTools package?"
objectives:
- "Overview of AmberTools package."
- "List of AmberTools tools to prepare your biological system."
keypoints:
- "AmberTools package has multiple tools to prepare, simulate and analyse biological systems."
- " You should familiarise yourself with `tleap`, `pamred`, `nab`, `cpptraj`, `antechamber`, `parmch2` and `sander` as they will ususally be helpful to set up a biological system."
- "`tleap` is used to generate topologies and coordinates for biological molecules."
- "`cpptraj` is used to postprocess and analyse AMBER trajectories."
- "`antechamber` is used to create parameters for non-standard aminoacids and drug-like molecules."
- "`sander` runs molecular dynamics simulations"
- "`parmed` allows modifying AMBER topologies"
---

## Overview of AmberTools20: 

Latest version: released on April 31, 2020 

[AmberTools suite](https://ambermd.org/AmberTools.php) is a set of several independent packages developed as part of the Amber20 software package and distributed free of charge. Its components are mostly released under the GNU General Public License (GPL). They work well by themselves, and with Amber20 itself. The list of packages is:

- **NAB/sff**: a program for building molecules, running MD or applying distance geometry restraints using generalized Born, Poisson-Boltzmann or 3D-RISM implicit solvent models
- **antechamber and MCPB**: programs to create force fields for general organic molecules and metal centers
- **tleap and parmed**: basic preparatory tools for Amber simulations
- **sqm**: semiempirical and DFTB quantum chemistry program
- **pbsa**: Several linear and non-linear numerical solvers for Poisson-Boltzmann models
- **3D-RISM**: solves integral equation models for solvation
- **sander**: workhorse program for molecular dynamics simulations
- **gem.pmemd**: tools for using advanced force fields
- **mdgx**: a program for pushing the boundaries of Amber MD, primarily through parameter fitting. Also includes customizable virtual sites and explicit solvent MD capabilities.
- **cpptraj and pytraj**: tools for analyzing structure and dynamics in trajectories
- **MMPBSA.py**: energy-based analyses of MD trajectories

In this session, we are going to use a subset of these tools: **antechamber (sqm), tleap, parmed, sander and cpptraj**.

