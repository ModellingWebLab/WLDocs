# The Cardiac Electrophysiology Web Lab

- [Click here for links to code, prototypes, and more information](links.md).
- [Click here for the WL1/WL2/Web Lab terminology disambiguation](terminology.md).

_The following document describes the **intended** features of the new web lab (2018-05-21)._

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

**Point of action:** Collaborate with PMR so that we can access its models.

### Annotation

Models are linked to protocols via an ontology that lists common model variables, e.g. the major currents (INa, ICaL, IKr, etc.) and their maximum conductances.
Currently, annotation happens by modifying the CellML files.

**Point of action**: decide whether to keep annotations inside the CellML files or use an external mechanism.
External annotation is harder to work with, but more flexible as it allows us to annotate models not stored on our own servers.

We currently use the [`oxmeta` ontology](https://github.com/Chaste/Chaste/blob/release/python/pycml/oxford-metadata.ttl), which is distributed as part of Chaste .

**Point of action**: We need to either find a community ontology, or work towards standardising our one (e.g. move from chaste to its own repo, get some input from others, given them access etc.).

- Note 1: There is also an [rdf file](https://github.com/Chaste/Chaste/blob/release/python/pycml/oxford-metadata.rdf).
- Note 2: The XML namespace for the annotations is https://chaste.comlab.ox.ac.uk/cellml/ns/oxford-metadata (not a link!).

It would also be great to move beyond shared variable names, and start documenting model provenance, using _relations_ to (oxmeta?) variables, such as `is_parameter_for`.
Once we have a way to reference our models and data sets, we should also be able to add _weak provenance data_, such as `ModelX:VariableY is_derived_from DataSetZ` or `ModelX:Component1 is_inherited_from ModelY:Component2` or `is_adapted_from`.
The fitting specification (see **TODO**) will provide _strong provenance data_, in the form of a complete description of how to obtain a parameter value from a model, protocol, and data set.

### Stimulus current

Typically, the CellML version of a model includes a stimulus current.
This is essential a default protocol, and so it can be argued this should not be part of a model.

- Note 1: The same holds for physical constants and physiological parameters such as temperature or membrane area: in a modular set-up these should really not be part of the _cell_ model.
- Note 2: In addition, most stimulus protocols are discontinuous, and consist of a small jump of 0.5, 1, or 2ms in a 1000ms period.
Unless the solver somehow knows when these jumps are, simulations need to be run with a maximum step size of e.g. 1ms.
This is inefficient during the systolic/refractory phase, where steps of over 100ms could otherwise be made.


## Protocols

At the moment, protocols are written using the [Functional Curation syntax](https://chaste.cs.ox.ac.uk/trac/wiki/FunctionalCuration).

**Point of action:** We need to decide whether to stick with this or come up with an easier-to-use alternative.

This might involve:

- Replacing FC with something procedural e.g. sandboxed Python (but then do we lose platform/tool independence?)
- Coming up with tools that simplify working with FC (e.g. a Python/Myokit library that users can use procedurally, but that then generates FC code)

### SED-ML

SED-ML is a community standard to describe experiments, but its capabilities are much more limited than functional curation

### MIASE

The Minimum Information About a Simulation Experiment (MIASE) project describes the minimum things that any standard encoding simulation experiments should contain.
([paper](http://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1001122) | [wiki](https://en.wikipedia.org/wiki/Minimum_information_about_a_simulation_experiment) | [website](http://co.mbine.org/standards/miase))


## Experimental data

WL2 needs the capability to display and process experimental data.

### Format

The current prototypes use CSV data.

**Point of action:** We need to decide on a suitable free, easy-to-read exchange format.
CSV is very easy to read, but bulky (25 bytes per float) and can suffer from rounding errors.
HDF5 is more compact and structured, but cannot be read/written without a special library (in theory it can, but the spec is 150 pages long).

Some types of hosting (e.g. PhysioNet) come with their own format.

### Annotation

Some formats (e.g. HDF5) support annotation, but others (CSV) don't.
As with model annotation, we might also want to annotate files without modifying them, so again external annotation seems best.

**Point of action:** Decide how to annotate data files (internal/external).

There is no ontology for experimental meta-data, but we really need one, so that users can perform structured queries to find data!
The MICEE standard lists a bunch of things that should be stored _about_ experimental data.

([paper](https://www.sciencedirect.com/science/article/pii/S0079610711000642))

- Note 1: The [MICEE website](https://micee.org) seems to be offline.

### Hosting

**Point of action**: We need someone to host data (annotated) files!

Initially, we can do this locally.

### Pre and post-processing

WL2 does _not_ include initial preprocessing such as capacitance artefact removal, leak correction, subtraction protocols etc.
It _does_ include secondary preprocessing such as calculating IV curves from (clean up) current recordings.
To allow initial preprocessing to be inspected and/or re-done, we would ideally store (1) cleaned up, annotated data in an approved exchange format, and (2) free-form raw data along with pre-processing code (e.g. proprietary data formats and matlab scripts).

Further pre and post-processing currently happens via functional curation (see [Protocols](#protocols)).


## Simulations

### Loading and manipulating models

Models will be read using our own CellML reading code, [cellmlmanip](https://github.com/ModellingWebLab/cellmlmanip), that reads CellML, creates [SymPy](http://sympy.org/) equations, and can manipulate them (e.g. adding a protocol-defined stimulus).

**Point of action:** Asif is writing this.

**Point of action:** Consider CellML 2.0 support.

### Running simulations

Currently, simulations can be run using either Chaste or a Cython back-end.

**Point of action:** Replace this with a Cython-only back-end.

## Fitting

**Point of action:** Michael is co-writing Pints.

**Point of action:** The current fitting specification uses model-specific variable annotations, we should experiment with model-agnostic versions instead

**Point of action:** Depending on what we decide for the experimental protocol specification language, we need to either replace or extend Aidan's prototype code.

