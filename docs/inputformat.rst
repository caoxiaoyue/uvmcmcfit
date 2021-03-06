Input Data Format
=================

``uvmcmcfit`` uses visibilities obtained from an interferometer as the primary
input data.  The format must be either *uvfits* (preferred) or CASA *ms*.  

Which format should I use?
--------------------------

Use the CASA *ms* format if you are only willing to run ``uvmcmcfit`` inside
the CASA shell.  Note that doing this will mean you can't take advantage of the
parallel processing features included in ``uvmcmcfit``.  For this reason, I
suggest using *uvfits* as the input file format for the visibilities. 


Do I really need to worry about the weights?!?
----------------------------------------------

The weights in the visibility dataset are a critical component of
``uvmcmcfit``.  So yes, you do need to worry about them.  ``uvmcmcfit`` uses
the weights to determine how much to trust each visibility measurement.  In
this way, the relative value of the weights is important (similar to a
deconvolution task like clean).  

Not only that, but the amplitude of the
weights matters as well.  Consider two extreme scenarios: one in which the
weights are very very tiny, and one in which they are very very large.  In the
former scenario, good and models become indistinguishable because the
goodness-of-fit measurement is normalized by the weights, so the difference
between a good model and a bad model becomes very very small.  On the other
hand, if the weights are too large, then two models which are both plausible fits might still show a significant difference in the goodness of fit measurement.  The right value for the amplitude of the weights is somewhere in between.

From my personal experience exploring both ALMA and SMA continuum data, scaling
the weights based on the scatter in the actual visibility datasets does a good
job of setting the amplitude of the weights.

.. Note::

    In future versions of CASA, the plan (as I understand it) is to set both
    the relative and absolute value of the weights rigorously.  For now, only
    the relative value of the weights is meaningful, so the following advice
    might be useful to you.

A CASA routine exists to re-compute the weights based on the scatter in the
visibilities within a spectral window (spw).  This task is called ``statwt``.
I found that this routine worked well within CASA, but when I exported the
results to a uvfits file (using ``exportuvfits``), the normalization of the
weights was off.  So I wrote my own version of CASA's ``statwt``:
:func:`uvutil.statwt`.  Using it properly is more convoluted than I would like
at the moment, but it works.  

First, you need to export a uvfits file that contains visibility data as
function of frequency.  Let's call this file 'defaultweights_frequency.uvfits'.
You then call :func:`uvutil.statwt` as in the following example::

    import uvutil
    uvutil.statwt('defaultweights_frequency.uvfits', 'statwtweights_frequency.uvfits')

where 'defaultweights_frequency.uvfits' is the visibility dataset with the
default weights and 'statwtweights_frequency.uvfits' is a new file that is
created by :func:`uvutil.statwt` that has the weights recomputed according to
the scatter in the visibilities within a given spectral window.

.. Note::

    You can use the ``ExcludeChannels`` option to exclude certain channels from
    being considered when computing the scatter in the visibilities (e.g. so
    that a strong line does not bias the rms measurement too high).  Do this by
    defining a list that contains pairs of start and end channels.  All
    channels between (and including) the start and end channels will be
    excluded when computing the rms scatter in the visibilities.  For example,
    uvutil.statwt('defaultweights_frequency.uvfits',
    'statwtweights_frequency.uvfits', ExcludeChannels=[45,145]) will ensure
    that channels 45 to 145 will not be considered when computing the rms
    scatter in the visibilities.

Next, import the new uvfits file back into CASA using ``importuvfits``.

Next, average over the channels of interest to create the single-channel
visibility dataset of interest.  Do this in CASA using ``split``.

Next export the output from ``split`` to a uvfits file using ``exportuvfits``.
This is the file that you will use as input for ``uvmcmcfit``.

