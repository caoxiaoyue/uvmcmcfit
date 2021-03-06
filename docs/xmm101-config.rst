Configuring config.yaml
***********************

config.yaml contains the instructions needed by ``uvmcmcfit`` to initiate the
model fitting process.

Required keywords
-----------------

A few house-keeping parameters::

    # Name of the target
    ObjectName: XMM101
 
    # Name of the fits image; the pixel scale in this image sets the pixel
    # scale in the model image
    ImageName: XMM101.concat.statwt.cont.mfs.fits

    # Name of the uvfits visibility data; the weights should be scaled such
    # that Sum(weights * real) ~ N_vis [see uvutil.statwt()]
    UVData: XMM101.concat.statwt.cont.uvfits
    
    # Number of walkers
    Nwalkers: 24

.. caution:: The number of walkers used by emcee must be more than double the number of parameters).  In this case, there are only 6 parameters, so the minimum number of walkers is 12.  I selected 24 to be on the safe side.

Now for parameters that describe the geometry of the system.  You must define
at least one region.  The first region should be named ``Region0``, the second
``Region1``, etc.  Pay attention to the indentation; the remaining keywords
must be indented to indicate they are sub-components of ``Region0``.  For each
region, you must define a RA and Dec center, an angular radial extent that
contains the emission which you are attempting to model, and at least one
source.

The first source should be named ``Source0``, the second source should be named
``Source1``, etc.  Sources are elliptical Gaussians.  Each source must have the
following parameters: the total intrinsic flux density (IntrinsicFlux [mJy]),
the effective radius defined as sqrt(a*b) (EffectiveRadius [arcsec]), the
offset in RA and Dec from RACentroid and DecCentroid (DeltaRA and DeltaDec
[arcsec]), the axial ratio (AxialRatio), and the position angle in degrees east
of north (PositionAngle [degrees]).

For each source parameter, you must specify the lower and upper limits as well
as how to initialize the walkers for that parameter.  This is done using the
following syntax: ``Limits: [lower limit, lower initialization, upper
initialization, upper limit]``. So, for example, in the code snippet below for
XMM101, ``Source0`` is permitted to have a total intrinsic flux density ranging
from 1 to 25 mJy, but is initialized with a uniform probability density
distribution between 5 and 10 mJy.

::

    # First region
    Region0:

        # Right Ascension and Declination center of the model image (degrees)::
        RACentroid: 36.449395
        DecCentroid: -4.2974618

        # Angular radial extent of the model image (arcsec)
        RadialExtent: 1.5

        # Source0
        Source0:

            # total intrinsic flux density
            IntrinsicFlux:
                Limits: [1.0, 5.0, 10.0, 25.0]

            # effective radius of elliptical Gaussian [sqrt(a*b)] (arcsec)
            EffectiveRadius:
                Limits: [0.01, 0.01, 1.2, 1.2]

            # Offset in RA and Dec from RACentroid and DecCentroid (arcseconds)
            DeltaRA:
                Limits: [-0.4, -0.2, 0.2, 0.4]
            DeltaDec:
                Limits: [-0.4, -0.2, 0.2, 0.4]

            # axial ratio = semi-minor axis / semi-major axis
            AxialRatio:
                Limits: [0.2, 0.3, 1.0, 1.0]

            # position angle (degrees east of north)
            PositionAngle:
                Limits: [0.0, 0.0, 180.0, 180.0]
      

Optional keywords
-----------------

By default, the maximum likelihood estimate is used to measure the goodness of
fit.  Alternatively, you may use the chi-squared value as the goodness of fit
criterion via::

    # Goodness of fit measurement
    LogLike: chi2

By default, parallel processing is not used.  To use parallel processing on a
single machine, set the Nthreads variable to a number greater than 1.  For
example, ::

    # Number of threads for multi-processing on a single computer
    Nthreads: 2

If you have access to a computer cluster with many compute cores, you can use
Message Passing Interface to greatly speed up the modeling process::

    # Use Message Passing Interface
    MPI: True
    Nthreads: 1

.. caution:: Nthreads must be equal to 1 if using MPI!

If you want to compare the model results with an image obtained at another
wavelength (e.g., an *HST* image), you must specify the location of the
alternative image as well as the telescope and filter used to obtain the
image::

    # Alternative image name (used only for comparing with best-fit model)
    OpticalImage: XMM101_F110W.fits

    # Telescope and filter of alternative image    
    OpticalTag: HST F110W

