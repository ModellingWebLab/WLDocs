# Subprojects

The cardiac part of the Web Lab is handled by a number of subprojects.
While some of these modules depend on each other, they are written to be relatively independent and somewhat re-usable.

## Proposed workflow

- The Web Lab (specifically `weblab-fc`) will use `cellmlmanip` to load a model, manipulate it in ways specified by the protocol, and then pass it to `weblab_cg` to print Cython code. This code is then compiled by `weblab-fc`, and used to run an experiment.

- Chaste users will get some kind of Python script that replaces `PyCml`, and has a nice command-line interface that lets you load a CellML model (via `cellmlmanip`), and then export it to any number of formats. The export step consists of (1) any required code manipulations (e.g. calculating a Jacobian with `cellmlmanip`) and (2) printing code, which may either be quite vanilla C++, or something more advanced e.g. with lookup tables (implemented in `weblab-cg`).

## `cellmlmanip`

[Cellmlmanip](https://github.com/ModellingWebLab/cellmlmanip) reads a CellML model, currently version 1.0/1.1, with plans for 2.0.
The CellML component structure is stripped, leaving a graph of [SymPy](https://www.sympy.org/en/docs.html) expressions, with units handled by [pint](https://pint.readthedocs.io).

The `manip` part of the name is mostly yet to come.
This will include:

- Adding or replacing parts of models
- Calculating derivatives (e.g. the Jacobian, or derivatives w.r.t. parameters)
- Unit conversion
- See e.g. https://github.com/ModellingWebLab/cellmlmanip/issues/21

Cellmlmanip can read and preserve RDF variable annotations, e.g. in oxmeta syntax.

## `weblab_cg`

[Weblab CG](https://github.com/ModellingWebLab/weblab-cg) accepts a `cellmlmanip` model as input, and then spits out code in any number of formats.

It's probably a lot simpler than cellmlmanip, with fewer responsibilities.

At the moment we haven't decided exactly how to structure CG, but there are two obvious choices:

1. Write it as a fairly generic tool, but stick our own templates and tests in. This means all the CG code is in a single place, and that e.g. chaste developers can benefit from updates made by web lab developers. It's slightly messy.

2. Write it as a very generic tool, and create new projects *that use the CG infrastructure* to print code, with templates stored inside these new projects. If we go down this road CG becomes very small indeed and we might want to merge it with cellmlmanip.

## `weblab-fc`

[weblab-fc](https://github.com/ModellingWebLab/weblab-fc) implements the functional curation (FC) language.
The new version will use `cellmlmanip` and `weblab_cg` to parse annotated CellML and create model files, writen in a Cython syntax.

Cython is a tool to interface with C/C++ code, in this case CVODE.
Unlike Python files, Cython-files need to be compiled, which happens partly static (for the CVODE wrappers) and partly on-the-fly, via distutils/setuptools (for the models).

### Some current implementation details

- Parses FC protocols (using fc's `CompactSyntaxParser` module)
- Checks model and protocol compatibility
- Manipulates the model, e.g. adding/removing variables or stimulus protocols
  - Again should transition from PyCml to cellmlmanip
- Generates model-specific Python code (extending the `fc` `AbstractModel` class) that can run simulations.
For more information, see [https://chaste.cs.ox.ac.uk/trac/wiki/FunctionalCuration/ProtocolSyntax].

### Fitting

Fitting will be handled using [PINTS](https://github.com/pints-team/pints).
Probably some part of FC will have to learn how to call PINTS.