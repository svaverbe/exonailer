# exonailer

The **EXO**planet tra**N**sits and r**A**d**I**al ve**L**ocity fitt**ER** (**EXO**-**NAILER**), is 
an easy-to-use code that allows you to efficiently fit exoplanet transit lightcurves, radial velocities 
or both. 

Author: Néstor Espinoza (nespino@astro.puc.cl)

*If you make use of this code, please cite Espinoza et al., 2016, ApJ, in press. (http://arxiv.org/abs/1601.07608)*

DEPENDENCIES
------------

This code makes use of six important libraries:

- **Numpy**.
- **Scipy**.
- **The Bad-Ass Transit Model cAlculatioN (batman) for transit modelling** (http://astro.uchicago.edu/~kreidberg/batman/).
- **emcee for MCMC sampling** (http://dan.iel.fm/emcee/current/).
- **Astropy for time conversions** (http://www.astropy.org).
- **The GNU Scientific Library** (https://www.gnu.org/software/gsl/)

All of them are open source and can be easily installed in any machine. Be 
sure to install them before running the installer (see below), otherwise, it 
will complain. This code also makes use of the `ajplanet` module for 
radial-velocity modelling (https://github.com/andres-jordan/ajplanet) and the 
`flicker-noise` module (https://github.com/nespinoza/flicker-noise), for modelling 
1/f noise. Copies of the source codes of those modules are included in this repository 
and will be installed automatically.

INSTALLATION
------------
To install the code, simply run the `install.py` code by doing:

    python install.py

After this is done, the code will be ready to use!

USAGE
-----

To use the code is very simple. Suppose we have a target that we named 
`my_data`:

    1. Put the photometry under `transit_data/my_data_lc.dat`. Similarly, 
       put the RVs (if you have any) under `rv_data/my_data_rvs.dat`. These 
       are expected to have four columns: times, data, error and name of the 
       instrument (which is a string); however, only the two first are mandatory: 
       the code will recognize that you don't have errors on your variables and if no 
       instrument names are given, it will assume all come from the same instrument. 
       The flux is expected to be normalized to 1. The RVs are expected to be in km/s.

    2. Create a prior file under `priors_data/my_data_priors.dat`. The code 
       expects this file to have three columns: the parameter name, the prior 
       type and the hyperparameters of the prior separated by commas (see below). 
       If you want a parameter to be fixed, put `FIXED` on the Prior Type column 
       and define the value you want to keep it fixed in the hyperparameters column.

As can be seen from the above, the code can handle data taken with different instruments. 
Currently this only modifies the outputs of the RVs where, if more than one instrument is 
detected, a different center-of-mass velocity is fitted for each instrument in order to 
account for offsets between them, and if jitter is included, a different jitter term is 
also fitted for each instrument.

Next, you can modify the options in the exonailer.py code. The options are:

    target:             The name of your target (in this case, `my_data`).

    phot_noise_model:   This parameter defines the noise model used for the photometry. If set 
                        to 'white', it assumes the underlying noise is white-noise. If set to 
                        'flicker', it assumes it is a white + 1/f.

    phot_detrend:       This performs a small detrend on the photometry. If set to 'mfilter' 
                        it will median filter and then smooth this filter with a gaussian filter. 
                        It works pretty well for Kepler data. If you don't want to do any kind 
                        of detrending, set this to None.

    window:             This defines the window of the 'mfilter'. Usually way longer than your 
                        transit event.

    phot_get_outliers:  This automatically sigma-clips any outliers in your data. It relies on 
                        having decent priors on the ephemeris (t0 and P).

    n_omit:             Is an array that lets you ommit transit in the fitting procedure (e.g., 
                        transits with spots). Just put the number of the transits (counted from 
                        the first event in time, with this event counted as 0) that you want 
                        to ommit in the list and the code will do the rest.

    ld_law:             Limb-darkening law to use. For all the laws but the logarithmic the 
                        sampling is done using the transformations defined in Kipping (2013). 
                        The logarithmic law is sampled according to Espinoza & Jordán (2015b).

    mode:               Can be set to three different modes. 'full' performs a full transit + rv 
                        fit, 'transit' performs only a transit fit to the photometry, while 'rvs' 
                        performs a fit to the RVs only.

    rv_jitter:          If set to True, an extra 'jitter' term is added to the RVs error to account 
                        for stellar jitter.

    transit_time_def:   Defines the input and output time scales (the times are assumed to be in the 
                        JD format, i.e., JD, BJD, MDJ, etc.) of the transit times. If input transit times 
                        are, for example, in utc and you want results in tdb, this has to be 'utc->tdb'.

    rv_time_def:        Same as for transit times but for the times in the RVs.

Once you are done with this, just run the code by doing:

    python exonailer.py

GENERATING THE PRIOR FILE
-------------------------

The priors currently supported by the code are:

    Normal:         Expects that the third column in the prior file has the form 

                                       mu,sigma 

                    where mu is the mean value and sigma the standard-deviation.

    Uniform:        Expects that the third column in the prior file has the form

                                         a,b, 

                    where a is the minimum value and b is the maximum value.

    Jeffreys:       Expects that the third column in the prior file has the form

                                        low,up

                    where low is the lower limit of the variable and up is the upper 
                    limit of the variable.

    FIXED:          This assumes you are giving the fixed value of the variable in 
                    the third column.

The mandatory variables that must have some of the above defined priors are:

    Period:         The period of the orbit of the exoplanet. Same units as the time.
    
    t0:             The time of transit center. Same units as the time.

    a:              Semi-major axis in stellar units.

    p:              Planet-to-star radius ratio.

    inc:            Inclination of the orbit in degrees.

    sigma_w:        Standard-deviation of the underlying white noise process giving rise to 
                    the observed noise (in ppm).

    ecc:            Eccentricity of the orbit.

    omega:          Argument of periapsis (in degrees)

Of course, e.g., for a circular fit, you might want to fix `ecc` (to 0) and `omega` (e.g., to 90). 
The optional variables are:

    mu:             Center-of-mass velocity of the RVs.

    K:              Radial-velocity semi-amplitude.
    
    sigma_w_rv:     Jitter term for radial-velocities (see below).

    sigma_r:        Parameter for 1/f noise (see below).

If in the options of the exonailer.py code you set `phot_noise_model` to `'flicker'`, then you 
must also define a `sigma_r` parameter (see Carter & Winn, 2009). If you set `rv_jitter` to 
`True`, you must also set a `sigma_w_rv` parameter for the jitter term. 

In the case in which you have two or more instruments for the RVs, you can also define 
different priors for each value of `mu` and `sigma_w_rv` by adding a subscript and the name 
of the instrument. For example, if you have data from coralie and feros, instead of 
using a common prior for `mu` and `sigma_w_rv` for both of them, you can specify one for 
each by assigning priors to the variables `mu_coralie` and `sigma_w_rv_coralie` and `mu_feros`
and `sigma_w_rv_feros`.

WHISH-LIST
----------

    + Add example datasets with RV + transit.

    + Create a tutorial.


TODO
----

    + Add option to add photometry from different instruments.

    + GPs for detrending and for noise models.

    + Transit and RVs for multi-planet systems.

    + Noise models for RVs.
