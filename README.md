# The Cardiac Electrophysiology Web Lab

- [Click here for links to code, prototypes, and more information](links.md).
- [Click here for the WL1/WL2/Web Lab terminology disambiguation](terminology.md).
- [Click here for the generic Web Lab infrastructure documentation](infrastructure.md).
- [Click here for the Cardiac Electrophysiology back-end implementation](implementation.md).

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

**Point of action:** _Collaborate with PMR so that we can access its models._

The WL2 uses Git for version control of models and protocols: see [version control](#version-control).
Where the reference copy is stored externally, the Web Lab will store a clone of the repository.


### Annotation

Models are linked to protocols via an ontology that lists common model variables, e.g. the major currents (INa, ICaL, IKr, etc.) and their maximum conductances.
Currently, annotation happens by modifying the CellML files.

**Almost settled:** _Decide whether to keep annotations inside the CellML files or use an external mechanism._
_We probably want to allow both, so add the ability to read/write separate files containing annotations, for increased flexibility._
_For performance, we will probably want to cache all annotations of everything the Web Lab knows in a [triplestore](https://en.wikipedia.org/wiki/Triplestore) of some kind._

We currently use the [`oxmeta` ontology](https://github.com/Chaste/Chaste/blob/release/python/pycml/oxford-metadata.ttl), which is distributed as part of Chaste.

**Point of action:** _We need to either find a community ontology, or work towards standardising ours (e.g. move from chaste to its own repo, get some input from others, given them access etc.)._

- Note 1: There is also an [rdf file](https://github.com/Chaste/Chaste/blob/release/python/pycml/oxford-metadata.rdf).
- Note 2: The XML namespace for the annotations is `https://chaste.comlab.ox.ac.uk/cellml/ns/oxford-metadata` (not a link! for historical reasons).
- Note 3: [ICEPO](https://academic.oup.com/database/article/doi/10.1093/database/baw017/2630205) is an ontology of _qualitative_ effects of mutations on ion channel function: ftp://ftp.nextprot.org/pub/current_release/controlled_vocabularies/cv_modification_effect.obo . 

It would also be great to move beyond shared variable names, and start documenting model provenance, using _relations_ to (oxmeta?) variables, such as `is_parameter_for`.
Once we have a way to reference our models and data sets, we should also be able to add _weak provenance data_, such as `ModelX:VariableY is_derived_from DataSetZ` or `ModelX:Component1 is_inherited_from ModelY:Component2` or `is_adapted_from`.
The fitting specification (see #fitting) will provide _strong provenance data_, in the form of a complete description of how to obtain a parameter value from a model, protocol, and data set.

Linking models to annotated data would automatically solve the problem of annotating models with e.g. species info (we could even say things like "27% human" if the annotations were really complete).
See [previous feedback](https://bitbucket.org/joncooper/fcweb/issues/99/sort-models-by-species).

### Stimulus current

Typically, the CellML version of a model includes a stimulus current.
This is essential a default protocol, and so it can be argued this should not be part of a model.

- Note 1: The same holds for physical constants and physiological parameters such as temperature or membrane area: in a modular set-up these should really not be part of the _cell_ model.
- Note 2: In addition, most stimulus protocols are discontinuous, and consist of a small jump of 0.5, 1, or 2ms in a 1000ms period.
Unless the solver somehow knows when these jumps are, simulations need to be run with a maximum step size of e.g. 1ms.
This is inefficient during the systolic/refractory phase, where steps of over 100ms could otherwise be made.




## Protocols

At the moment, protocols are written using the [Functional Curation syntax](https://chaste.cs.ox.ac.uk/trac/wiki/FunctionalCuration).

**Point of action:** _We need to decide whether to stick with this or come up with an easier-to-use alternative._

This might involve:

- Replacing FC with something procedural e.g. sandboxed Python (but then do we lose platform/tool independence?)
- Coming up with tools that simplify working with FC (e.g. a Python/Myokit library that users can use procedurally, but that then generates FC code)

### SED-ML

SED-ML is a community standard to describe experiments, but its capabilities are much more limited than functional curation

### MIASE

The Minimum Information About a Simulation Experiment (MIASE) project describes the minimum things that any standard encoding simulation experiments should contain.
([paper](http://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1001122) | [wiki](https://en.wikipedia.org/wiki/Minimum_information_about_a_simulation_experiment) | [website](http://co.mbine.org/standards/miase))
It is not a format for representing experiments, however.

### Cardiac protocol library

The current Functional Curation protocols rely on a cardiac specific and a cardiac-non-specific library of functions (written in FC syntax).
If we continue down this road these need to be documented and stored somewhere as some kind of standard library.

Previous feedback: [Importing partial protocols](https://bitbucket.org/joncooper/fcweb/issues/56/allow-import-of-protocols).

### Visualisation

Previous feedback: [It would be cool to (automatically?) generate flow diagrams for protocols.](https://bitbucket.org/joncooper/fcweb/issues/83/support-facilitate-richer-documentation-of).




## Experimental data

WL2 needs the capability to display and process experimental data.

### Format

The current prototypes use CSV data.

**Almost settled:** _We need to decide on a suitable free, easy-to-read exchange format._
_CSV is very easy to read, but bulky (25 bytes per float) and can suffer from rounding errors._
_HDF5 is more compact and structured, but cannot be read/written without a special library (in theory it can, but the spec is 150 pages long)._
_Perhaps people won't mind what we use, as long as we provide an export option on the website?_
_We can also compress CSV, of course._

The current Python backend already uses HDF5.

Some types of hosting (e.g. PhysioNet) come with their own format.

There is [a Python package called Neo](https://github.com/NeuralEnsemble/python-neo) that aims to provide a unified interface to several electrophysiology formats.

### Annotation

Some formats (e.g. HDF5) support annotation, but others (CSV) don't.
As with model annotation, we might also want to annotate files without modifying them, so again external annotation seems best.

**Point of action:** _Decide how to annotate data files (internal/external)._

There is no ontology for experimental meta-data, but we really need one, so that users can perform structured queries to find data!
The MICEE standard lists a bunch of things that should be stored _about_ experimental data.

([paper](https://www.sciencedirect.com/science/article/pii/S0079610711000642))

- Note 1: The [MICEE website](https://micee.org) seems to be offline.

### Hosting

**Point of action:** _We need someone to host data (annotated) files!_

Initially, we can do this locally.

### Pre and post-processing

WL2 does _not_ include initial preprocessing such as capacitance artefact removal, leak correction, subtraction protocols etc.
It _does_ include secondary preprocessing such as calculating IV curves from (cleaned up) current recordings.
To allow initial preprocessing to be inspected and/or re-done, we would ideally store (1) cleaned up, annotated data in an approved exchange format, and (2) free-form raw data along with pre-processing code (e.g. proprietary data formats and matlab scripts).

Further pre and post-processing currently happens via functional curation (see [Protocols](#protocols)).




## Simulations

### Loading and manipulating models

Models will be read using our own CellML reading code, [cellmlmanip](https://github.com/ModellingWebLab/cellmlmanip), that reads CellML, creates [SymPy](http://sympy.org/) equations, and can manipulate them (e.g. adding a protocol-defined stimulus).

**Point of action:** _Asif is writing this._

**Point of action:** _Consider CellML 2.0 support._

[LibCellML](https://github.com/cellml/libcellml) is the planned new library for reading (but not manipulating) CellML 2.0 files.
We've added Python bindings to it, and would like to use it eventually, provided it has real benefits over a simple plain-Python reader.

### Running simulations

Currently, simulations can be run using either Chaste or a Cython back-end.

**Point of action:** _Replace this with a Cython-only back-end._




## Fitting

WL2 will include fitting, or _inference_ algorithms, e.g. derivative-free non-linear methods for optimisation, and methods to estimate parameter distributions in a Bayesian framework.

**Point of action:** _Michael is co-writing Pints._

**Point of action:** _The current fitting specification uses model-specific variable annotations, we should experiment with model-agnostic versions instead._

**Point of action:** _Depending on what we decide for the experimental protocol specification language, we need to either replace or extend Aidan's prototype code._

### Processing power

Fitting takes a lot of processing power, especially when you do it repeatedly (which is often needed to test the quality of the fit).
This means we'll probably need some sort of offline component for people to experiment with, after which users can upload a fitting spec and ask us to run it.




## Version control

We will use version control throughout WL2, so that all _entities_ (models, protocols, data sets, results) can have multiple versions.
To this end, each model and protocol will have its own Git repository.

Git is not well suited to storing large datasets (and these also change less frequently) so instead for these (including simulation results) we just store the files (compressed) on disk, with metadata in the database.

- Note 1: This Git repo set-up makes everything very flexible, but also harder to learn.
The web interface will hide the complexity from users that don't care, presenting just a linear list of versions.
- Note 2: GIT is line-based, and so not ideal for XML. It still works though! And for human diff'ing we'll provide a visualiser that understands the mathematics ([BiVeS](https://github.com/binfalse/BiVeS) and its [web interface](https://github.com/binfalse/BiVeS-WebApp)).

### Identifiers

Every entity requires a unique identifier, which can be used to create links on the web site, but is also accessible to fitting specifications and model and data annotations.
These take the form of a uniform resource identifier ([URI](https://en.wikipedia.org/wiki/Uniform_Resource_Identifier)), more specifically, a uniform resource locator ([URL](https://en.wikipedia.org/wiki/URL)).
We currently have unique canonical URIs for each version of each entity type that Web Lab handles (see **todo, add link**).




## Combine archives

Specific versions of Git repos for entities can be downloaded as COMBINE archives.
We could define subtypes for this, but it's not strictly required.




## Sharing control

We will allow all entities (models, protocols, data sets) to be either private, shared with a limited audience, or fully public.

**Point of action:** _What will the limited audience be? Groups etc.? Or simply logged in users or a big group of admin/power users? Should we start thinking about the case where even admins shouldn't have (easy) access to all data?_

[Previous feedback on limited audience sharing](https://bitbucket.org/joncooper/fcweb/issues/68/allow-sharing-entities-with-specific-users) and [this](https://bitbucket.org/joncooper/fcweb/issues/107/allow-admins-to-unpublish-models) and [this](https://bitbucket.org/joncooper/fcweb/issues/108/give-users-some-way-to-request-new).




## Comments and discussion

At one point, we mentioned it would be nice to allow comments / discussion threads of all entities.
I still like this idea!
(Would have to refer to a repo though, not a version within a repo).
This also ties in with "exposures", or homepages for each entity where users can see the different versions etc.




## REST API

WL1 provides a hidden [REST API](https://en.wikipedia.org/wiki/Representational_state_transfer) to allow other services to communicate with it.
WL2 should keep this feature, but extend and document it.






