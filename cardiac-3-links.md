[Home](./README.md)

# Terminology

* **Web Lab** - The concept - and software for - a web-based tool that can combine model and experiment specifications to run simulations
* **Cardiac Electrophysiology Web Lab (WL)** - A Web Lab for the domain of cardiac electrophysiology.
* **WL1** - The original WL implementation. Doesn't have any data or fitting. Uses the Functional Curation syntax for experiments, and runs simulations using either Chaste or a Cython simulation back-end.
* **WL1 fitting prototype** - An extension to WL1 created by Aidan, that uses a Python backend with CMAES and a custom adaptive MCMC routine to perform fitting on cardiac models.
* **WL1 electrochemistry prototype** - An extension to WL1 to show the concept works beyond cardiac electrophysiology, and can be applied to e.g. electrochemistry problems
* **WL2** - A future implementation of the Cardiac Electrophysiology Web Lab, that includes experimental data and fitting capabilities

## Links to different versions

WL2 Demo versions

* WL2 Demo version: https://scrambler.cs.ox.ac.uk
* About WL2: https://scrambler.cs.ox.ac.uk/about
* Team/Contact: https://scrambler.cs.ox.ac.uk/contact

WL1 Live versions

* WL2 demo version: https://scrambler.cs.ox.ac.uk/
* WL1 pretty link: https://chaste.cs.ox.ac.uk/WebLab, which redirects to: https://travis.cs.ox.ac.uk/FunctionalCuration/
* WL1-fitting prototype: https://muck.cs.ox.ac.uk/FunctionalCuration/
* WL1-electrochemistry prototype: https://lofty.cs.ox.ac.uk/FunctionalCuration/

WL2 Code

* All new code: https://github.com/ModellingWebLab
* WL2 Django front-end: https://github.com/ModellingWebLab/WebLab
* WL2 issues: https://github.com/ModellingWebLab/project_issues
* WL2 project management: https://waffle.io/ModellingWebLab/project_issues

WL1 Code

* WL1 code: https://github.com/ModellingWebLab/fcweb
* WL1-fitting prototype code 1: https://github.com/ModellingWebLab/fcweb/tree/cardiac-fitting
* WL1-fitting prototype code 2: https://chaste.cs.ox.ac.uk/trac/browser/chaste/projects/AidanDaly
* WL1-electrochemistry prototype code: https://bitbucket.org/joncooper/fcweb/branch/echem
* WL1-fitting prototype code 3: https://github.com/ModellingWebLab/chaste-project-fitting-pints

Functional curation (FC)

* Wiki: https://chaste.cs.ox.ac.uk/trac/wiki/FunctionalCuration
* Code: https://chaste.cs.ox.ac.uk/trac/browser/chaste/projects/FunctionalCuration

## Papers

* WL2 plans: [Reproducible model development in the Cardiac Electrophysiology Web Lab](https://www.sciencedirect.com/science/article/pii/S0079610718300257). Daly, Clerx, Beattie, Cooper, Gavaghan, Mirams (2018) PBMB.
* WL1 description: [The Cardiac Electrophysiology Web Lab](https://www.sciencedirect.com/science/article/pii/S0006349515047530). Cooper, Scharm, Mirams (2016) Biophysical Journal.
* FC: [High-throughput functional curation of cellular electrophysiology models](https://www.sciencedirect.com/science/article/pii/S0079610711000502?via%3Dihub). Cooper, Mirams, Niederer (2011) Progress in Biophysics and Molecular Biology.
* Virtual experiments: [A call for virtual experiments: Accelerating the scientific process](https://www.sciencedirect.com/science/article/pii/S0079610714001825?via%3Dihub). Cooper, Vik, Waltemath (2015) Progress in Biophysics and Molecular Biology ([Open-access pre-print](https://peerj.com/preprints/273/)).

