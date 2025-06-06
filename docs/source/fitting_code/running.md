```eval_rst
.. _running_fit:
```
# How to run the code

Here we provide some instructions on how to use the code for the various
running modes and on how to analyse its results.

```eval_rst
.. _runcard:
```
## Runcard specifications
First of all, the basic object required to run the code is the runcard.
In this section we document the settings that need to be specified here. Example runcards are available from the separate
repository [smefit_database](https://github.com/LHCfitNikhef/smefit_database). This repository also contains the theory predictions and experimental data files used in the
latest smefit publications.

Clone the `smefit_database` repository, and run
```yaml
python update_runcards_path.py -d /path/to/runcard/destination/ runcards/A_LHC_NLO_LIN_GLOB.yaml
```
This will create a ``smefit`` runcard in ``/path/to/runcard/destination/`` ready to be used,
pointing to the experimental data and theory tables in the repository `smefit_database`.
The user can change this manually if other datasets are desired.


### Input and output path
The folder where the results will be saved can be set using ``result_path``. The file containing
the posterior of the fitted Wilson coefficient will be saved in ``resulth_path/result_ID``.
If ``result_ID`` is not provided, it will be automatically set to the name of the runcard (and any already existing result will be overwritten).

```yaml
result_ID:
result_path:
data_path:
theory_path:


```
### Theory specifications
The default perturbative order of the theory prediction is set by the key ``default_order``. Orders may also be specified
per datset, see [here](./example.html#datasets-to-consider-and-coefficients-to-fit) for more details.
The order in the EFT expansion should be specified by setting ``use_quad`` to either ``True`` or ``False`` to include quadratic or only linear corrections respectively. The option ``use_t0`` controls the use
of the ``t0`` prescription and ``use_theory_covmat`` specifies whether or not to use the theory covariance matrix
which can be specified in the theory files.

```yaml
default_order: LO
use_quad: False
use_t0: False
use_theory_covmat: True
cutoff_scale: 1000
```
Here ``cutoff_scale`` specifies the scale (in GeV) above which all datapoints will be excluded from the fit.

### Minimizer specifications
The different parameters controlling the minimizer used in the analysis are specified here.
If ``single_parameter_fits`` is set to ``True``, the Wilson coefficient specified in the run-card
will be fit one at time, setting all the others to 0. See [here](./example.html#single-parameter-fits) for more details.
If ``pairwise_fits`` is set to ``True``, the minimizer carries out an automated series of pair-wise fits to all possible pairs of
Wilson coefficients that  are specified in the run-card.
Pairwise fits are supported only with |NS|.

```yaml
pairwise_fits: False
single_parameter_fits: False
bounds: Null

# NS settings
nlive: 400 # number of live points used during sampling
lepsilon: 0.05 #  Terminate when live point likelihoods are all the same, within Lepsilon tolerance.
target_evidence_unc: 0.5 # target evidence uncertanty
target_post_unc: 0.5 # target posterior uncertanty
frac_remain: 0.01 # Set to a higher number (0.5) if you know the posterior is simple.
store_raw: false # if true, store the raw result and enable resuming the job.
vectorized: false # if true, ultranest samples a vector from the prior (recommended for large scale problems)
float64: false # double precision


#MC settings
mc_minimiser: 'cma' # Allowed options are: 'cma', 'dual_annealing', 'trust-constr'
restarts: 7 # number of restarts (only for cma)
maxiter: 100000 # max number of iteration
chi2_threshold: 3.0 # post fit chi2 threshold

#A settings
n_samples: 1000 # number of the required samples of the posterior distribution
```

### Datasets to consider and coefficients to fit
The datasets and Wilson coefficients to be included in the analysis must be listed under ``datasets``
and ``coefficients`` respectively. The default order for each dataset is taken from  ``default_order``. However, it is
possible to specify specific orders per dataset. To do this, add the key ``order`` to the dataset entry as follows.

```yaml
datasets:

  - name: ATLAS_tt_8TeV_ljets_Mtt
  - name: ATLAS_tt_8TeV_dilep_Mtt
    order: NLO_QCD
  - name: CMS_tt_8TeV_ljets_Ytt
    order: NLO_QCD
  - name: CMS_tt2D_8TeV_dilep_MttYtt
    order: NLO_QCD
  - name: CMS_tt_13TeV_ljets_2015_Mtt
    order: NLO_QCD
  - name: CMS_tt_13TeV_dilep_2015_Mtt
    order: NLO_QCD
  - name: CMS_tt_13TeV_ljets_2016_Mtt
    order: NLO_QCD
  - name: CMS_tt_13TeV_dilep_2016_Mtt
    order: NLO_QCD
  - name: ATLAS_tt_13TeV_ljets_2016_Mtt
    order: NLO_QCD
  - name: ATLAS_CMS_tt_AC_8TeV
    order: NLO_QCD
  - name: ATLAS_tt_AC_13TeV
  ...
  ...

# Coefficients to fit
coefficients:

  OpQM: {'min': -10, 'max': 10}
  O3pQ3: {'min': -2.0, 'max': 2.0}
  Opt: {'min': -25.0, 'max': 15.0}
  OtW: {'min': -1.0, 'max': 1.0}
  ...
  ...

```

As exemplified above, the syntax to specify the Wilson coefficient corresponding to the operator
``O1`` is ``O1 : {'min': , 'max':} `` where ``min`` and ``max`` indicate the bounds within
the sampling is performed.

### Constrains between coefficients
Some Wilson coefficients are not directly fit, but rather constrained to be linear combinations
of the other ones. Taking as example some coefficient of the Higgs sector considered in ``smefit2.0``,
this can be specified in the runcard in the following way

```yaml
  OpWB: {'min': -0.3, 'max': 0.5}
  OpD: {'min': -1.0, 'max': 1.0}
  OpqMi: {'constrain': [{'OpD': 0.9248},{'OpWB': 1.8347}], 'min': -30, 'max': 30}
  O3pq: {'constrain': [{'OpD': -0.8415},{'OpWB': -1.8347}], 'min': -0.5, 'max': 1.0}
```
In this example the coefficients ``CpqMi`` and ``C3pq`` correspondong to the operators
``OpqMi`` and ``O3pq`` will be set to
```yaml
CpqMi = 0.9248*CpD + 1.8347*CpWB
C3pq = -0.8415*CpD -1.8347*CpWB
```

### Fit in different basis
It is possible to run a fit using a different basis.
In this case the coefficients specified in the runcard should be the ones to fit, and
a rotation basis defining the new basis in terms of the Warsaw basis should be given as input. This can be done
using the option below.

```yaml
rot_to_fit_basis: /path/to/rotation/rotation.json

```

In addition, it is possible to perform a fit in a PCA rotated basis. This corresponds to the basis spanned
by the eigenvectors of the Fisher information matrix at the linear level in the EFT expansion. To carry out such a fit,
add the following flag

```bash
smefit NS --rotate_to_pca path/to/the/runcard/runcard.yaml
```

```eval_rst
.. _ns:
```

### Adding custom likelihoods
SMEFiT supports the addition of customised likelihoods. This can be relevant when an external likelihood is already at hand
and one would like to combine it with the one constructed internally in SMEFiT. To make use of this feature, one should add
the following to the runcard:

```yaml
external_chi2:
  'ExternalChi2': /path/to/external/chi2.py
```
Here, ``ExternalChi2`` is the name of the class that must be defined in the referenced python file as follows:

```python
import numpy as np


class ExternalChi2:
    def __init__(self, coefficients):
        """
        Constructor that allows one to set attributes that can be called in the compute_chi2 method
        Parameters
        ----------
        coefficients:  smefit.coefficients.CoefficientManager
            attributes: name, value
        """
        self.example_attribute = coefficients.name

    def compute_chi2(self, coefficient_values):
        """
        Parameters
        ----------
         coefficients_values : numpy.ndarray
            |EFT| coefficients values

        """

        # example
        chi2_value = np.sum(coefficient_values**2)
        return chi2_value
```
One is free to set custom attributes in the constructor. The coefficient values during optimisation
are accesible via ``coefficient_values`` in the ``compute_chi2`` method. In order for the external chi2
to work, it is important one does not change the name of the ``compute_chi2`` method!

### Adding RG evolution
Renormalisation group evolution can be turned on in the fit by adding the following to the runcard.

```yaml
rge:
  init_scale: 5000.0
  obs_scale: dynamic # float or "dynamic"
  smeft_accuracy: integrate # options: integrate, leadinglog
  yukawa: top # options: top, full or none
  adm_QCD: False # if true, the EW couplings are set to zero
  rg_matrix: <path/to/rge_matrix.pkl>
```

## Running a fit with NS
To run a fiy using Nested Sampling use the command
```bash
smefit NS path/to/the/runcard/runcard.yaml
```

This will generate a file named ``posterior.json`` in the result folder,
containing the posterior distribution of the coefficients specified in the runcard.

```eval_rst
.. _mc:
```
## Running a fit with MC

**Disclaimer**: the MC mode is only supported for linear fits.

The basic command to run a fit using Monte Carlo is

```bash
    smefit MC path/to/the/runcard/runcard.yaml -n replica_number
```

This will produce a file called ``replica_<replica_number>/coefficients_rep_<replica_number>.json``
in the result folder, containing the values of the Wilson coefficients for the replica.
Once an high enough number of replicas have been produced, the results can be merged into the final posterior
running PostFit

```bash
    smefit POSTFIT path/to/the/result/ -n number_of_replicas
```
where ``<number_of_replicas>`` specifies the number of replicas to be used to build the posterior.
Replicas not satisfying the PostFit criteria will be discarded. If the final number of good replicas is lower than
``<number_of_replicas>`` the code will output an error message asking to produce more replicas first.
The final output is the file ``posterior.json`` containing the full posterior of the Wilson coefficients.

## Solving the linear problem
In case only linear cocrrections are used, one
can find the analytic solution to the linear problem by
```bash
    smefit A path/to/the/runcard/runcard.yaml
```
This will also sample the posterior distribution according to the runcard.


## Single parameter fits
Given a runcard with a number of Wilson coefficients specified, it is possible to fit each of them in turn,
keeping all the other ones fix to 0.
To do this add to the runcard
```yaml
single_parameter_fits: True
```
and proceed as documented above for a normal fit.
For both `NS` and `A` the final output will be the file ``fit_results.json``
containing the independent posterior of the fitted Wilson coefficients, obtained by a series of independent single parameter fits.

It is possible to perform single paramters in the presence of multiple constraints. For example

```yaml

single_parameter_fits: True
uv_couplings: true

coefficients:
  # Free params
  OWWW: {'min': -3, 'max': 3}
  Opq1: {'min': -3, 'max': 3}
  Opq3: {'min': -3, 'max': 3}

  ## To fix ##
  OpqMi: {'constrain': [{'Opq1': 1}, {'Opq3': -1}], 'min': -2, 'max': 2 }
  O3pq: {'constrain': [{'Opq3': 1}], 'min': -1, 'max': 1 }

```
performs a single parameter fit for ``OWWW``, ``Opq1`` and ``Opq3``. Note that this requires `uv_couplings` to be set to
True (this allows you to fit parameters that do not appear in the theory tables). The naming `uv_couplings` stems from fitting with
constraints among Wilson coefficients and UV parameters.



## Individual parameter scan
The code can also be used to produce 1-dimensional scans of the chi2 function.
The command

```bash
    smefit SCAN /path/to/the/runcard/runcard.yaml
```
will produce in the results folder a series of pdf files containing plots for
1-dimensional scans of the chi2 with respect to each parameter in the runcard.

This is in general done for Wilson coefficients, but if one is interested in a specific UV model and in particular to perform a mass scan, SMEFiT allows for this specific case.
The way it works is that in the runcard, one specifies

```yaml
uv_couplings: true

coefficients:
  ## To fix ##
  OpqMi: {'constrain': [{'m': [0.24, -2]}, {'m': [0.5, -4]}], 'min': -2, 'max': 2 }

  # Mass parameter
  m: {is_mass: True, 'min': 0.2, 'max': 100.0}
```
and when performing the scan, it will recognise that this is a mass scan and only the mass will be scanned. Note that the mass values are assumed to be specified in TeV.
