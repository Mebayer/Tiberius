.. _fitting:
Tiberius Fitting
==================


This guide provides an overview of the general fitting workflow in ``Tiberius``. It explains the purpose of the main input files, the typical fitting procedure, and the most important concepts.
For a complete worked example, see :ref:`example_wasp94`.



Overview/Workflow
------------------
.. _workflow:
The basic workflow also described in `workflow.txt` broadly follows the main steps:

1) Configure the fit in ``fitting_input.txt``.
2) Define priors in ``example_prior_def_file.txt``. 
3) (Optional) Generate limb-darkening coefficients with ``generate_LDCS.py``.
4) Fit the white-light curve.
5) Fit the spectroscopic light curves.
6) Iterate through various systematics models (polynomials/GPs) (and possibly wavelength bins) via running ``run_gppm_fit.py``.


Input Files
^^^^^^^^^^^^

The fitting is controlled by two main files:

- ``fitting_input.txt`` contains the general fitting configuration,
  data paths, model settings, and fitting options.

- ``example_prior_def_file.txt`` defines all free and fixed model parameters together
  with their prior distributions.

Limb Darkening
^^^^^^^^^^^^^^^^

There are two ways to generate limb darkening coefficients (LDCs) for the transit model.

- **LDTk**: needs the stellar parameters (Teff, logg, [Fe/H]) in the ``fitting_input.txt`` file, but no other addtional input. 
- **exotic-ld**: needs to be installed first (see ExoTiC-LD's `installation instructions <https://exotic-ld.readthedocs.io/en/latest/views/installation.html>`_), it supports spectroscopic (JWST, HST), photometric (Spitzer, TESS), and custom instrument modes.

The LDCs are generated using the ``generate_LDCS.py`` script. This script creates output files that can be used directly and, if chosen, overwrite any other priors given in the ``example_prior_def_file.txt`` file.

Fit the White-Light Curve
^^^^^^^^^^^^^^^^^^^^^^^^^^^
The white-light curve is fitted first, and the results are used to inform the spectroscopic light curve fits. The white-light curve is fitted using the ``light_curve_fit.py``script.

For white-light curve fitting, the argument 0 must be given:

.. code-block:: bash

    python /path_to_your_TiberiusFolder/Tiberius/src/fitting_utils/light_curve_fit.py 0



