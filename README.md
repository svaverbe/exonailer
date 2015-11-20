# exo-nailer

The *EXO*planet tra*N*sits and r*A*d*I*al ve*L*ocity fitt*ER* (*EXO*-*NAILER*), is an easy-to-use code 
that allows you to efficiently fit exoplanet transit lightcurves, radial velocities 
or both. It makes use of two important pieces of code:

    + The *B*ad-*A*ss *T*ransit *M*odel c*A*lculatio*N* (batman) for transit modelling.
      -> http://astro.uchicago.edu/~kreidberg/batman/
    + emcee for MCMC sampling.
      -> http://dan.iel.fm/emcee/current/

DEPENDENCIES
------------

This code makes use of three important libraries:

        + Numpy.
        + Scipy.

All of them are open source and can be easily installed in any machine.

WHISH-LIST
----------
    + Allow the code to handle priors from the outside (i.e., easier 
      than going inside the utilities code).

    + Add example datasets.

    + Create a tutorial.