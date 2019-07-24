# Cardiac Electrophysiology Web Lab infrastructure

This document describes the infrastructure that is specific to the Cardiac Electrophysiology Web Lab.
The generic Web Lab infrastructure is defined [here](./infrastructure.md).

## Models

Models are implemented in CellML.

In the future, they will be converted to [SymPy](https://www.sympy.org/en/docs.html) expressions using [cellmlmanip](https://github.com/ModellingWebLab/cellmlmanip) (with units handled in [pint](https://pint.readthedocs.io)).
The resulting 

## Protocols

Protocols are written in functional curation syntax (see below).

## Functional curation

Web Lab experiments run `Functional curation` with a model and protocol.

### FC-Runner

The Celery task to run experiments this is stored in the [fc-runner](https://github.com/ModellingWebLab/fc-runner/tree/master/fcws) repository.
The task checks input from the web interface, and fires off the `fc` experiment.
When completed, it packs up the results and sends them back to the web interface.

### FC

The actual experiment running code is stored in the [weblab-fc](https://github.com/ModellingWebLab/weblab-fc) repository.

This repo contains the Python `fc` module, which is at the heart of the cardiac EP Web Lab.
It:

- Parses CellML models.
  - Currently this is handled by PyCml, but we are converting to [cellmlmanip](https://github.com/ModellingWebLab/cellmlmanip).
- Parses FC protocols (using fc's `CompactSyntaxParser` module)
- Checks model and protocol compatibility
- Manipulates the model, e.g. adding/removing variables or stimulus protocols
  - Again should transition from PyCml to cellmlmanip
- Generates model-specific Python code (extending the `fc` `AbstractModel` class) that can run simulations.
  - Currently this is handled by PyCml, but we are converting to [weblab-cg](https://github.com/ModellingWebLab/weblab-cg).
  - Simulations are run in CVODE, using Cython to interface with it.
- Executes the protocol

For more information, see [https://chaste.cs.ox.ac.uk/trac/wiki/FunctionalCuration/ProtocolSyntax].

## Web Lab without FC?

At the start of the project, we discussed allowing arbitrary Python scripts, running in some kind of sandbox.
We should probably still look into this, as users will want to implement their own pre- and post-processing methods, at least.

Update: Jonathan says
> This was not so much 'without FC' but extending FC to allow Python for post-processing etc not just the domain-specific language.

## Fitting

Fitting will be handled using [PINTS](https://github.com/pints-team/pints).

