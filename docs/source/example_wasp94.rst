.. _example_wasp94:
Example: Fitting a Light Curve
==============================

This tutorial provides a step-by-step example of how to perform light curve fitting with Tiberius. 

We will use transmission spectroscopy data from the WASP-94 system. The observations were taken with NTT/EFOSC2 on the night of 14 August 2017 and are available in the ESO archive. 

If you want to follow the full data reduction procedure, refer to the data reduction documentation. Otherwise, you can directly use the example data provided in:

    Tiberius/Example/WASP94/

This allows you to go through the fitting process without performing the reduction yourself.

The example's basic workflow is structured as follows:

0. :ref:`Data preparation <data-prep>`  
   Understanding the input files and selecting the required data for the white-light fit.

1. :ref:`White-light curve fitting <wl-fit>`  
   Fitting the integrated light curve to determine global system parameters.
   1.1 :ref: edit fitting_input.txt
   1.2 :ref: generate limb darkening coefficients with generate_LDCS.py
   1.3 :ref: fit white light curve

2. :ref:`Spectroscopic fitting <spectroscopic-fit>`  
   Fitting individual wavelength bins using the white-light results as reference.



0. Data Preparation
----------

.. _DataPreparation:

Before fitting the light curve, we need to understand the structure of the example data and prepare it for Tiberius.

Example data for WASP-94 can be found in:

    Tiberius/Example/WASP94/input_files

This directory contains many files, but for the first step of the workflow (fitting the white-light curve), we focus only on the following key files, which can be found in the dictionary WL, except for the first which is directly under input_files:

- **`time_norm.pickle`** – Array of observation times.
- **`white_light_flux.pickle`** – Array of normalized flux measurements.
- **`white_light_error.pickle`** – Array of uncertainties for each flux measurement.


Notes:

- Only these files are required to perform the white-light curve fit.
- All arrays must be aligned by observation time and have the same length.
- Optional files are only needed if GP modeling of correlated noise is used.


0.1 Example Folder Structure (Flexible)
--------------------------------------
.. _FolderStructure: 

All required input files can be placed under any project folder structure that suits your workflow. 
The following is a general example for the WASP-94 EFOSC2 dataset, which you can adapt as needed::
    Tiberius/
    data/                     
    └── fitting/
        └── wasp-94/
            └── EFOSC2_20170814/
                └── input_files/
                    ├── time_norm.pickle
                    └── WL/
                        ├── white_light_flux.pickle
                        ├── white_light_error.pickle
                        ├── sky_norm.pickle
                    

Notes:

- Optional files (`background.pickle`, `x.pickle`, `y.pickle`) are only needed if you plan to use Gaussian Process (GP) modeling for correlated noise.
- All `.pickle` files should have the same length and be aligned by observation time.
- You can adapt this folder structure to fit your project organization; the pipeline will still work as long as paths are correctly specified in the configuration file.

1. White-light curve fitting
----------------------------

.. _WLFit:

Once you have a basic setup for your project folder and all your input data, in the first stage we will perform a fit to the white-light curve. This stage is structured in different steps to determine the fitting parameters, including the limb-darkening coefficients in a substructured step before performing the WL light curve fit. There are several options for different models and sampling methods to use for the fit.

1.1 Edit fitting_input.txt
---------------------------

.. _fittinginputtxt_:

We start by copying the required input text files into the project folder:

- `fitting_input.txt`   
- `example_prior_def_file.txt`

Both can be found in the folder ``Tiberius/src/fitting_utils`` in an empty format, which we will now go through and edit. The finished example for this stage can also be found in ``(TODO: Tiberius/Example/WASP94/)``.

The file `fitting_input.txt`  contains almost all of the inputs that need to be added or changed in order to run the first fitting attempt, and it includes well-described comments for each required input.

It is important to note that this `.txt` file is required not only for the fitting itself, but also for generating the limb-darkening coefficients (LDCs) before running the first light curve fit. This is also explained in the workflow. First, the `fitting_input.txt` needs to be set up.

The workflow and inputs differ depending on whether you intend to run a fit for a white-light curve, which is the first step in the general workflow, or for spectroscopic light curves at a later stage. This section focuses on the inputs required for a white-light curve fit. For spectroscopic light curves, a few changes are noted in Section (TODO) (repeat for spectroscopic light curves).

1.1.1 INPUT FILES
------------------

For the input files, you just need to provide the relative path to the `fitting_input.txt` file. The example structure is shown above (:ref:`example folder structure <FolderStructure>`), and the relative path according to that is shown in the example file.

.. code-block:: text

    # Light curve inputs
    time_file = input_files/time_norm.pickle  # pickled numpy array of times

    flux_file = input_files/WL/white_light_flux.pickle
    # pickled numpy array of fluxes
    # white-light: shape (1,)
    # spectroscopic: shape (nbins, nfluxes)

    error_file = input_files/WL/white_light_error.pickle
    # pickled numpy array of errors
    # white-light: shape (1,)
    # spectroscopic: shape (nbins, nfluxes)


The next step is to input the centre of the wavelengths and the bin width. For the white-light curve, this corresponds to the full wavelength range of the data. These values must be obtained from your data.
In this example, the wavelength centre can be obtained directly from the `white_wvl_centre.pickle` file, which contains the value used for the white-light extraction.

.. code-block:: text

    wvl_centres = 5480
    # single value (white-light) OR pickled array (spectroscopic bins)

    wvl_bin_full_width = 3320
    # FULL width of the bin (NOT half-width)

    wvls_unit = angstrom
    # angstrom or micron


#### Choice of transit model

.. code-block:: text

    transit_model = batman  # options: batman or catwoman


Several transit models are available. These differ in how the planet is geometrically and physically represented during the transit.

- **batman**  
  Standard transit model for symmetric exoplanet light curves (Kreidberg 2015).  
  Assumes a circular planet and produces symmetric transit shapes.  

- **catwoman**  
  Asymmetric transit model that allows different properties across the planetary disk.  
  The planet is represented as two semi-circles with potentially different radii.  
  Useful for probing asymmetries.

- **fleck** (optional / experimental)  
  Model for surface inhomogeneities such as spots or brightness variations.  
  Mainly used for stellar or planetary surface feature investigations.

In this example, we will use Batman.

1.1.2 LIMB DARKENING
--------------------

This section of the `fitting_input.txt` defines the inputs related to limb darkening.

**Limb darkening** describes the effect where a star appears dimmer toward its edge (limb) than at its center due to a decrease in intensity. This modifies the shape of the transit light curve, particularly during ingress and egress, and must therefore be included in the model to avoid biases in the fitted parameters.


##### Limb darkening coefficients (LDCs)

Which law to use and packages to use is defined through the `fitting_input.txt` file but LCDs are generated separately using the `generate_LDCs.py` script prior to the first white-light fit. This corresponds to step 2 in the workflow overview.

The LDC generation step is optional and controlled by a switch in the input file:
.. code-block:: text
    generate_LDCs = 1 # do you want to generate LDCs? Set to 1 (on) or 0 (off).

- If set to **0**: LDCs are not generated, and no additional limb darkening inputs are required in `fitting_input.txt`.
- If set to **1**: LDCs are generated, and the corresponding inputs must be provided.

In the latter case, the LDCs are computed before the main fitting procedure and are then used as fixed or prior inputs in the light curve fitting.

In this workflow example, we first set up the `fitting_input.txt` file to generate the LDCs, followed by the white-light curve fitting step.

Limb darkening coefficients (LDCs) require additional dependencies. 
Before generating LDCs, ensure that the required package and stellar models are installed (see Installation: https://tiberius.readthedocs.io/en/latest/installation.html and ExoTiC-LD: https://exotic-ld.readthedocs.io/en/latest/views/installation.html).

Two packages are currently supported for LDC computation:

.. code-block:: text

    LDCs_package = LDTk # options: exotic-ld or LDTk


- ExoTiC-LD
- LDTk (Limb-Darkening Toolkit; Parviainen & Aigrain, 2015)

Both packages can be used to compute LDCs, depending on the chosen configuration. For this example, the LDTk option is used, since the specific instrument EFOSC2 is not available in ExoTiC-LD (and therefore the corresponding inputs for that package are not required in this example).

The LDTk Python package is already installed as part of the Tiberius setup installation.

The limb darkening behaviour is described using analytic limb-darkening laws, which can be fitted simultaneously with the transit model.

.. code-block:: text

    ld_law = quadratic  # options: linear, quadratic, square-root, nonlinear


Available limb-darkening laws:

- **Linear**
  $$ I(\mu) = I(0)\,[1 - u_1(1 - \mu)] $$

- **Quadratic**
  $$ I(\mu) = I(0)\,[1 - u_1(1 - \mu) - u_2(1 - \mu)^2] $$

- **Square-root**
  $$ I(\mu) = I(0)\,[1 - u_1(1 - \mu) - u_2(1 - \sqrt{\mu})] $$

- **Non-linear (4-parameter)**
  $$ I(\mu) = I(0)\,\left[1 - \sum_{k=1}^{4} u_k (1 - \mu^{k/2})\right] $$


where $\mu = \cos\theta$ and $I(0)$ is the intensity at the center of the stellar disk (Claret 2000). The coefficients $u_1$ to $u_4$ are wavelength-dependent limb-darkening parameters.

There is also the option to replace negative limb-darkening coefficients (u1, u2) generated during the LDC computation.

.. code-block:: text

    replace_negative_LDCs = 0  # 0 = keep values, 1 = replace negative values


Next, the stellar parameters required for the limb-darkening coefficient calculation must be provided. 

For this example, the parameters of WASP-94A are taken from Teske et al. (2016):

.. code-block:: text

    Teff = 6194            # stellar effective temperature [K]
    Teff_err = 5           # uncertainty in Teff [K]

    logg_star = 4.210      # surface gravity log(g)
    logg_star_err = 0.011  # uncertainty in log(g)

    FeH = 0.320            # stellar metallicity [Fe/H]
    FeH_err = 0.004        # uncertainty in [Fe/H]

The final part of the limb-darkening inputs controls whether the generated LDCs are used in the fit and how they are incorporated (e.g. as fixed values or priors). This is relevant later for the fitting and not yet for the genartaion of the LCDS


.. code-block:: text

    # for fitting the limb darkening
    use_generated_ld = 1
    # use generated LDC values (1 = on, 0 = off)
    # if LDCs are set to 'fixed', these values will be used directly in the fit

    use_generated_ld_as_prior = 0
    # use generated LDCs and their uncertainties as Gaussian priors
    # this overwrites any prior definitions in the prior file

    ld_uncertainty_multiplier = 3
    # factor to inflate LDC uncertainties (e.g. 3 → errors ×3)

    use_kipping_parameterisation = 0
    # use Kipping parameterisation for LDC sampling (1 = on, 0 = off)
    # NOTE: not fully tested

Before running the first light curve fit, the LDCs must be generated. This can already be done at this stage using the edited `fitting_input.txt`, even if some of the fitting parameters that follow have not yet been set, as they are not required for the LDC generation step.

You can therefore generate the LDCs at this point using the current input file (this step can also be performed later, once all fitting parameters have been defined).

To generate the LDCs, save the input file in your project folder and run `generate_LDCs.py` from this location:

.. code-block:: bash

    python /path_to_your_TiberiusFolder/Tiberius/src/fitting_utils/generate_LDCs.py


This will read the required inputs and produce the following output files:

- ``LD_coefficients.txt``
- ``ldtk_model.pickle``


1.1.3 FITTING PARAMETERS
------------------------
.. _fittingParameters_:

In this section, we define the parameters that control the light curve fitting procedure and data preprocessing. 

The `prior_filename` specifies the path to the prior file, which contains all model parameters and their corresponding priors or fixed values. This file must be adapted to the data and is described in the subsection (:ref:`Prior file <PriorFile>`).

.. code-block:: text

    prior_filename = example_prior_def_file.txt



#### Transit window definition

These parameters define the in-transit and out-of-transit regions of the light curve:

.. code-block:: text

    contact1 = 95
    contact4 = 325

- **contact1 / contact4**  
  Frame numbers marking the beginning of ingress (contact 1) and the end of egress (contact 4).  
  These are used to define the out-of-transit baseline, e.g. for GP optimisation or flux normalisation.

These values are determined from the data, for example by plotting the white-light curve and identifying the ingress and egress points.

#### Data clipping and selection

.. code-block:: text

    first_integration = 0
    last_integration = -1

- **first_integration / last_integration**  
  Allow trimming of the light curve by removing data points at the beginning or end (Python indexing).


#### Noise and outlier handling

.. code-block:: text

    common_noise_model =
    clip_outliers = 1
    sigma_cut = 4
    median_clip = 0

- **common_noise_model**  
  Path to a precomputed common-mode noise model (optional).

- **clip_outliers**  
  Enable outlier removal before fitting (1 = on, 0 = off).

- **sigma_cut**  
  Threshold for outlier rejection (e.g. 4 = 4σ clipping).

- **median_clip**  
  Choose clipping method: running median (1) or model-based (0).


#### Flux normalisation

.. code-block:: text

    renorm_flux = 0

- **renorm_flux**  
  Renormalise flux so that the out-of-transit baseline equals unity (1 = on, 0 = off).


1.1.4 Systematics modelling
-----------------------------
This section defines how instrumental systematics and correlated noise in the light curve are modelled.

The input files (e.g. time, background, detector position) provide auxiliary variables that may correlate with non-astrophysical trends in the data. Tiberius uses these inputs to construct a model that separates astrophysical signal (the transit) from instrumental effects.

Two approaches are available:

- **Parametric model**  
  Uses deterministic functions (e.g. polynomials, exponential ramps, step functions) applied to the auxiliary inputs.

- **Gaussian Process (GP) model**  
  Uses kernel-based correlations between data points to model more complex and non-parametric systematics.

If `model_input_files` (for parametric modelling) and `GP_model_input_files` (for GP modelling) are left empty, no systematics modelling is performed and the remaining inputs in this section are ignored.

If systematics are to be included, provide the paths to the corresponding input files, separated by commas. The behaviour of each input is then defined by the chosen configuration (e.g. polynomial orders, exponential ramps, or kernel classes). Multiple components can be used simultaneously by specifying them as comma-separated entries.

In this example, the systematics are modelled using a low-order polynomial combined with the transit model to remove long-term trends in the light curve.

For the white-light curve, a quadratic polynomial in time is used. This choice is implemented via the `polynomial_orders` parameter in the `fitting_input.txt` file:

.. code-block:: text

    model_input_files = input_files/time_norm.pickle
    normalise_inputs = 1

    polynomial_orders = 2

    exponential_ramp = 0
    step_function = 0

In this example, Gaussian Processes are not used. These inputs are therefore left empty.

.. code-block:: text

    GP_model_input_files =
    normalise_GP_inputs = 1

    kernel_classes =

    white_noise_kernel = 1
    typeII_maximum_likelihood = 0
    reset_kernel_priors = 0


1.1.5 Prior File
----------------

.. _PriorFile_:

In the `example_prior_def_file.txt`, the priors for all model parameters are defined. Parameters can either be fixed or treated as free. For free parameters, you can choose between a uniform prior (`prior_type = U`) or a Gaussian (normal) prior (`prior_type = N`). For a uniform prior, `prior_1` and `prior_2` define the lower and upper bounds of the parameter. For a Gaussian prior, `prior_1` corresponds to the mean value and `prior_2` to the standard deviation. These can be ignored for fixed values.

For this dataset, the priors were chosen based on the analysis presented in `Ahrer et al. (2022) <https://doi.org/10.1093/mnras/stab3805>`_, where the WASP-94Ab observations were originally published and analysed.

The example prior file used for the white-light curve fitting can be found in ``example_prior_def_file_fitted_wasp94_WL.txt``. This file provides a complete set of priors consistent with the configuration used in this example.

1.1.6 Sampling setup
----------------------

Tiberius provides two sampling methods for parameter inference: **MCMC (emcee)** and **nested sampling (dynesty)**. The choice is defined via the `sampling_method` parameter.

- **MCMC (emcee)**: ensemble sampler that explores the posterior distribution using Markov chains. 

- **Dynesty (nested sampling)**: performs nested sampling to compute the posterior and evidence simultaneously. 

The selected method determines which set of additional parameters is used below. In this example, we will use 'dynasty'.

.. code-block:: text
  sampling_method = dynesty


The configuration below controls the accuracy and efficiency of the nested sampling run. The number of live points scales with the dimensionality of the model, and the convergence criterion defines when the sampling terminates.

.. code-block:: text

    nlive_points_pdim = 50
    precision_crit = 1e+1

- **nlive_points_pdim**  
  Number of live points per model parameter. The total number of live points is given by  
  `nlive_points_pdim × number of fitted parameters`. Higher values improve accuracy but increase computation time.

- **precision_crit**  
  Convergence criterion for nested sampling. The run stops when the remaining uncertainty in the evidence falls below this threshold. Smaller values increase accuracy but require longer runtimes.

1.1.7 Plotting and output
------------------------

The final options control plotting behaviour and the name of the output directory.

.. code-block:: text

    show_plots = 0
    save_plots = 1
    rebin_data =
    output_foldername = fitting_01

1.3 First WL curve fit
----------------------

Once everything is set up (input files and prior file), you can run the fitting from your project folder. The argument ``0`` tells the code that this is a white-light curve fit.

.. code-block:: bash

    python /path_to_your_TiberiusFolder/Tiberius/src/fitting_utils/light_curve_fit.py 0

During execution, the code will print progress information to the terminal. After completion, the best-fit parameters are displayed in the terminal and written to output files.

The results are stored in the output folder specified in ``fitting_input.txt``. This folder contains:

- Figures showing the fitted light curve and model components  
- Sampling diagnostics and statistics  
- Files with the best-fit parameters  
- Pickle files containing the input data, posterior samples, and fitted models  

In addition, a copy of the prior file and an image of the fitted model are automatically created. These are saved in the project folder (not in the output folder).
