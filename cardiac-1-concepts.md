# Concepts

This page describes various concepts to do with the cardiac electrophysiology web lab.
It's usually out of date (see various tickets etc.).

![A schematic overview of the cardiac electrophysiology web lab](img/overview.png)

The Cardiac Electrophysiology Web Lab (WL2) links models of cellular electrophysiology with protocols describing experimental procedures, and with experimental data obtained from the same procedures.
Users can simulate and view the outcome of applying any protocol to any (compatible) model, and can compare the results to stored experimental data files.
Using the same interface, users can compare data files to each other, or inspect the predictions of different models for the same experiment.
Moving beyond qualitative comparison, numerical error measures can be defined and then minimised by algorithmically tuning model parameters.

The figure above shows the general outline of WL2.
The individual components and their connections are described below.

## Models

A model is a system of ordinary differential equations (ODEs) that describes (some aspects of) the electrophysiology (EP) of a cell, plus a physiologically viable set of initial states.
Most models describe either a single ionic current, or a set of currents plus the ionic diffusion mechanisms that give rise to the cellular action potential (AP).
Some models also include other aspects such as signalling or contraction mechanics.

Models are written in [CellML 1.0 or 1.1](http://cellml.org/).

In the current implementation, models are stored on the website, but we'd like to move to a situation where they're kept in external repositories, e.g. the Physiome Model Repository (PMR, also known as the [CellML Model Repository](https://models.physiomeproject.org/cellml)).

The WL2 uses Git for version control of models and protocols: see [version control](#version-control).
Where the reference copy is stored externally, the Web Lab will store a clone of the repository.

### Annotation

Models are linked to protocols via an ontology that lists common model variables, e.g. the major currents (INa, ICaL, IKr, etc.) and their maximum conductances.
Currently, annotation happens by modifying the CellML files.

_The annotation mechanism may change, with annotations living inside the model and/or in a [triplestore](https://en.wikipedia.org/wiki/Triplestore)._

We currently use the [`oxmeta` ontology](https://github.com/Chaste/Chaste/blob/release/python/pycml/oxford-metadata.ttl), which is distributed as part of Chaste.

 _We'd like a community ontology, but seems best way to achieve this is to just do whatever we feel like and wait until there are enough other people who want this too_

- Note 1: There is also an [rdf file](https://github.com/Chaste/Chaste/blob/release/python/pycml/oxford-metadata.rdf).
- Note 2: The XML namespace for the annotations is `https://chaste.comlab.ox.ac.uk/cellml/ns/oxford-metadata` (not a link! for historical reasons).
- Note 3: [ICEPO](https://academic.oup.com/database/article/doi/10.1093/database/baw017/2630205) is an ontology of _qualitative_ effects of mutations on ion channel function: ftp://ftp.nextprot.org/pub/current_release/controlled_vocabularies/cv_modification_effect.obo . 

See e.g. 
- https://github.com/ModellingWebLab/project_issues/issues/63
- https://github.com/ModellingWebLab/project_issues/issues/64

It would also be great to move beyond shared variable names, and start documenting model provenance, using _relations_ to (oxmeta?) variables, such as `is_parameter_for`.
Once we have a way to reference our models and data sets, we should also be able to add _weak provenance data_, such as `ModelX:VariableY is_derived_from DataSetZ` or `ModelX:Component1 is_inherited_from ModelY:Component2` or `is_adapted_from`.
The fitting specification (see #fitting) will provide _strong provenance data_, in the form of a complete description of how to obtain a parameter value from a model, protocol, and data set.

See e.g.
- https://github.com/ModellingWebLab/project_issues/issues/60
- https://github.com/ModellingWebLab/project_issues/issues/63

Linking models to annotated data would automatically solve the problem of annotating models with e.g. species info (we could even say things like "27% human" if the annotations were really complete).
See [previous feedback](https://bitbucket.org/joncooper/fcweb/issues/99/sort-models-by-species).

## Protocols

Protocols are written using the [Functional Curation language](https://chaste.cs.ox.ac.uk/trac/wiki/FunctionalCuration), using [this syntax](https://chaste.cs.ox.ac.uk/trac/wiki/FunctionalCuration/ProtocolSyntax).

_We might move some post-processing into sandboxed Python_

### SED-ML

SED-ML is a community standard to describe experiments, but its capabilities are much more limited than functional curation

### MIASE

The Minimum Information About a Simulation Experiment (MIASE) project describes the minimum things that any standard encoding simulation experiments should contain.
([paper](http://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1001122) | [wiki](https://en.wikipedia.org/wiki/Minimum_information_about_a_simulation_experiment) | [website](http://co.mbine.org/standards/miase))
It is not a format for representing experiments, however.

### Cardiac protocol library

The current Functional Curation protocols rely on a cardiac specific and a cardiac-non-specific library of functions (written in FC syntax).
If we continue down this road these need to be documented and stored somewhere as some kind of standard library.

## Experimental data

WL2 needs the capability to display and process experimental data.
See e.g. 
- https://github.com/ModellingWebLab/WebLab/issues/29

### Format

The current prototypes use CSV data.

### Annotation

See e.g.
- https://github.com/ModellingWebLab/project_issues/issues/62
- https://github.com/ModellingWebLab/project_issues/issues/63
- https://github.com/ModellingWebLab/WebLab/issues/29

### MICEEE

The MICEE standard lists a bunch of things that should be stored _about_ experimental data.

([paper](https://www.sciencedirect.com/science/article/pii/S0079610711000642))

- Note 1: The [MICEE website](https://micee.org) seems to be offline.

### Hosting

_We need someone to host data (annotated) files. Initially, we can do this locally_

### Pre and post-processing

WL2 does _not_ include initial preprocessing such as capacitance artefact removal, leak correction, subtraction protocols etc.
It _does_ include secondary preprocessing such as calculating IV curves from (cleaned up) current recordings.
To allow initial preprocessing to be inspected and/or re-done, we would ideally store (1) cleaned up, annotated data in an approved exchange format, and (2) free-form raw data along with pre-processing code (e.g. proprietary data formats and matlab scripts).

Further pre and post-processing currently happens via functional curation (see [Protocols](#protocols)).

## Simulations

### Loading and manipulating models

Models will be read using our own CellML reading code, [cellmlmanip](https://github.com/ModellingWebLab/cellmlmanip), that reads CellML, creates [SymPy](http://sympy.org/) equations, and can manipulate them (e.g. adding a protocol-defined stimulus).

Next [weblab_cg](https://github.com/ModellingWebLab/weblab-cg) will use cellmlmanip to read CellML files and then print code for use with the web lab.

### LibCellML

[LibCellML](https://github.com/cellml/libcellml) is the planned new library for reading (but not manipulating) CellML 2.0 files.
We've added Python bindings to it, and would like to use it eventually, provided it has real benefits over a simple plain-Python reader.

### Running simulations

Currently, simulations can be run using either Chaste or a Cython back-end.
Ultimately, we will only use [weblab_fc](https://github.com/ModellingWebLab/weblab-fc).

## Fitting

WL2 will include fitting, or _inference_ algorithms, e.g. derivative-free non-linear methods for optimisation, and methods to estimate parameter distributions in a Bayesian framework.

This will happen using [PINTS](https://github.com/pints-team/pints).

### Processing power

Fitting takes a lot of processing power, especially when you do it repeatedly (which is often needed to test the quality of the fit).
This means we'll probably need some sort of offline component for people to experiment with, after which users can upload a fitting spec and ask us to run it.

## Version control

We will use version control throughout WL2, so that all _entities_ (models, protocols, data sets, results) can have multiple versions.
To this end, each model and protocol will have its own Git repository.
[BiVeS](https://github.com/binfalse/BiVeS) is used to compare model versions.

### Identifiers

Every entity requires a unique identifier, which can be used to create links on the web site, but is also accessible to fitting specifications and model and data annotations.
These take the form of a uniform resource identifier ([URI](https://en.wikipedia.org/wiki/Uniform_Resource_Identifier)), more specifically, a uniform resource locator ([URL](https://en.wikipedia.org/wiki/URL)).
We currently have unique canonical URIs for each version of each entity type that Web Lab handles (see **todo, add link**).

## Combine archives

Specific versions of Git repos for entities can be downloaded as COMBINE archives.
We could define subtypes for this, but it's not strictly required.

## Sharing control

We will allow all entities (models, protocols, data sets) to be either private, shared with a limited audience, or fully public.

See e.g.
- https://github.com/ModellingWebLab/WebLab/issues/40

## Comments and discussion

At one point, we mentioned it would be nice to allow comments / discussion threads of all entities.

See e.g.
- https://github.com/ModellingWebLab/WebLab/issues/61

## REST API

WL1 provides a hidden [REST API](https://en.wikipedia.org/wiki/Representational_state_transfer) to allow other services to communicate with it.
WL2 will keep this feature, but extend and document it.
See the infrastructure page for details.

