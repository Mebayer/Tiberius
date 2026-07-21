.. _quickstart:

Quickstart Fitting
===================

This quickstart guide is meant to give a very brief introduction to the most basic usage of Tiberius. It is not meant to be a comprehensive tutorial, but rather a starting point for users who want to get up and running quickly. For more detailed information and examples, please refer to the rest of the documentation.

.. _quickstart_Installation:

1. Installation 
---------------

If you haven't already, please follow the installation instructions :ref:`installation` to install Tiberius and its dependencies.

2. Setup your project folder
-----------------------------

To get started quickly and test your installation, you can use the example data provided in the repository: `Tiberius/examples/wasp94`.

To perform a fit, you will need to set up a project folder. This can have whatever structure you like, but for the sake of this quickstart guide, we will assume a simple structure.

Copy the entire `/wasp-94` directory so that it sits alongside the Tiberius source directory. This preserves the relative paths used throughout the example.

For example::
    
    parent_directory/
    ├── Tiberius/
    └── wasp-94/

.. code-block:: bash

    cd parent_directory
    cp -r Tiberius/examples/wasp-94 .   




3. Example Fitting data
-----------------------

This example uses observations of a transit of WASP-94b obtained with NTT/EFOSC2. The data are provided in the `input_files.zip` as a compressed archive.

First, extract the archive using:

.. code-block:: bash

    cd wasp-94/EFOSC2_20170814
    unzip input_files.zip 

This will contain the raw data files, which are in the form of arrays. For this quickstart, not all of these are needed, but if you want to later test and play around with different options, you can use all of the files. The extracted folder structure should remain unchanged so that you do not have to modify any of the paths in the `input_files.txt` file.

4. Run Tiberius
------------------

Now you should be ready to run Tiberius! The first step is to generate the limb darkening coefficients (LDCs) for the white light curve. This is done using the ``generate_LDCS.py`` script, which takes as input the stellar parameters and the instrument throughput. For this example, you can run:

.. code-block:: bash

    python /path_to_your_TiberiusFolder/src/fitting_utils/generate_LDCS.py
    
This will generate the LDCs for the white light curve and create the following two files as output in the directory:

- ``LD_coefficients.txt``
- ``ldtk_model.pickle``

Having created the LDCs, you can now perform the fit to the white light curve. This script takes as input the fitting parameters and options specified in the ``fitting_input.txt`` file, which is located in the same directory as the input files. You can run the fit using:

.. code-block:: bash

    python /path_to_your_TiberiusFolder/src/fitting_utils/light_curve_fit.py 0


5. Output
------------------

This simple example will produce the following output files:

- `fitted_model_wb0001.png`: an image file showing the fitted model.
- `example_prior_def_file_fitted_wb0001.txt`: a text file containing the now fitted values.
- output directory `/fitting_example`(a name defined in the `fitting_input.txt`)
    


