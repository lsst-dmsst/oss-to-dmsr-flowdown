# Updating the LSST Data Management System Requirements (DMSR, LSE-61) with flow-down from the LSST Observatory System Specifications (OSS, LSE-30) of OSS requirements
Michael Wood-Vasey
Jim Bosch, Eric Bellm

## Prologue
Notes/Reminders of relevant documents:

SRD LPM-17
OSS LSE-30
DMSR LSE-61

(In beta)
https://www.lsst.io
LPM are LSST Project Management
LSE are LSST Systems Engineering

## Motivation

The Data Management System Requirements do not necessarily completely represent a flow-down from their higher-level document, the LSST Observatory System Specifications (OSS, LSE-30).

Michael Wood-Vasey (DM Science Validation Scientist)
Jim Bosch (DM Data Release Pipeline)
Eric Bellm (DM Alert Production)

undertook a careful read-through of the OSS requirements to identify ones not covered in the Data Management System Requirements (DMSR, LSE-61).  The goal was to first ensure that all _functional_ requirements were included -- i.e., the DMS shall calculate a particular quantity.  When straight-forward from the OSS, _performance_ goals have also been included.  However, for cases with unclear thresholds or potentially unmeetable goals (in the judgement of MWV, Bosch, and Bellm) we have deferred adding a performance goal and have merely specified a requirement that the quantity be measured.

The notes from our original read-through of the OSS and DMSR at the DMLT+SST meeting at the University of Washington in May 21-23 of 2018 were collected in a Google Spreadsheet:
https://docs.google.com/document/d/11Pvo1BgkqiI4NfXzg1HPI_712MmsI7bSwfa4XdKsNbg

## Recommendations

### Additions

The following requirements should be added to the DMSR

1. Contribution to astrometric errors in Prompt Processing

Specification: Data processing shall contribute no more than an RMS error of *dmL1AstroErr* to point-source astrometric errors in Prompt Processing data products.  These requirements apply to the relative errors in repeat observations and are separate from the absolute calibration requirements.

| Description | Value | Unit | Name |
|-------------|-------|------|------|
| Maximum contribution from DM to Prompt Processing point source astrometric errors | 0.1 | arcsecond | *dmL1AstroErr* |

Discussion: This requirement can be tested with simulation, and in commissioning using repeated observations of one or more fields.  

Characterizing precursor data with the same analysis tools will be informative even if the analysis of precursor data is done without an explicit performance requirement.

Determining the contribution of the data processing versus other contributions can be difficult.  Certainly, if the overall systematic error can be demonstrated to be less than the requirement, then the requirement will have been passed.  However, there will be an additional irreducible uncertainty floor set by the atmosphere.

For simulations, this effect can be precisely modeled and known and the contribution from the data processing pipeline estimated.

For astrometry from real on-sky data, the atmosphere will set an estimated fundamental floor at $\sigma \approx 10$~mas.

The astrometric uncertainty can be modeled as
$\sigma^2 = (C\theta/{\rm SNR})^2 + \sigma_{\rm atm}^2 + \sigma_{\rm sys}^2$
where $\theta$ is the realized FWHM of the PSF.  All remaining contributions to the uncertainty can be attributed to $*dmL1AstroErr*=\sigma_{\rm sys}^2$.

Derived from Requirements:
OSS-REQ-0149


2. Contribution to astrometric errors in Data Release Processing

Specification: Data processing shall contribute no more than an RMS error of dmL2AstroErr to point-source astrometric errors in DRP Products.

| Description | Value | Unit | Name |
|-------------|-------|------|------|
| Maximum contribution from DM to Level 2 point source astrometric | 0.05 | arcsecond | dmL2AstroErr |

Discussion: This requirement can be tested by comparison with simulated inputs.

[The same discussion applies for this dmL2AstroErr measurement as for the dmL1AstroErr requirements.]


Derived from Requirements:
OSS-REQ-0162

Note: This specification is essentially directly from the OSS.


3. World Coordinate System Accuracy

Specification: The DM system shall be able to measure how much WCS inaccuracies, including any refinements due to astrometric calibration, in processed images contribute to the PSF FWHM of template and deep detection coadds.

Discussion: Testing can be done by create a co-additions of postage stamps of bright, isolated stars centered on the per-source centroids.  Compare these postage-stamp coadditions to the widths of the same stars in the full-image coadds.  The latter will be using the WCS to connect all of the pixels between images.  Comparing the width of the PSF from the per-source coadd to the PSF from the full image coadd will be a reasonable measure of this 

These tests can be done on precursor data, ideally with similar depth and multiple epochs as LSST.  Stars with detectable proper motion over the course of the survey should be excluded from this measurement.

Derived from Requirements:
OSS-REQ-0153

Notes:
We are only adding a _functional_ requirement to measure this effect.  We do not specify a _performance_ requirement at this time.
The OSS says 0.1 milliarsec.  We are concerned that this is (a) not measurable to this level and (b) not achievable.

The outlined method to do the measurement is very reasonable.  It's possible that some development will help inform the reasonable requirement to set for this.

4. Artifact and Transient+Moving Object Rejection in Deep Coadds

Specification: False detections on the deep detection coadds caused instrumental artifacts (e.g., glints) or by moving objects, transient objects shall be no more than *falseDeepDetect* fraction of all detections on those coadds. This specification will be tested using simulations under realistic survey conditions encountered in Commissioning.

| Description | Value | Unit | Name |
|-------------|-------|------|------|
Fraction of all detections on deep detection coadds caused by unremoved artifacts | 0.1 | percent | *falseDeepDetect* |

Discussion: 
Objects that only appear for a small fraction of the LSST survey should not be part of the Object Table.  The scientific rationale for this requirement is to enable  statistical studies of objects in the static sky (galaxies, stars) that are not overly biased by false detections. *falseDeepDetect* is a reasonable number that achieves this. This is also a reasonable upper limit based on experience with existing surveys.

To distinguish transients the observations need to be separated in time.  To distinguish instrumental artifacts the observations need to be separated in boresight pointing and rotational angle.

Note that variable objects should be detected, but the distinction between variable and transient can be a function of the observing not just the underlying physical nature of the object.  A transient object with a 10-year characteristic lightcurve is effectively variable rather than transient during the LSST survey.  As a working definition of transient, one might choose to say that an object that is only detectable during a contiguous 10% of the survey is transient.  For early data releases, this 10% criterion would imply that transients with time-scales of months, e.g., nearby supernovae, will be included in the deep coad detectionsd.

Instrumental artifacts will not otherwise appear in data release catalogs, while transient and moving objects will be identified in the DIA Object and Solar System Object catalogs.

Potential Edge Cases:
* Nearby stars with significant proper motions may be a challenge to properly include and exclude.  A star that moves less than 1"/year can be likely reconstructed and identified in the Object table.  But a star that moves faster than that will likely need special handling to correctly include.  
* AGN from galaxies not visible in the coadd but that are only visible because of increased emission for several years of the survey.

However, such edge cases will be a much smaller fraction of the total detections than *falseDeepDetect*.

This requirement can be tested through multiple approaches:
a) Inject simulated artifacts (moving objects, satellite trails, optical effects) and characterize how many appear in the deep detection coadd catalog.  This approach will be particularly useful in quantifying behavior at both the high and low S/N regimes.
b) For each DIA Object compare per-epoch forced photometry measurements with being a constant source.  Compare the distribution of per-epoch forced measurements to the expectation for a non-transient objects, potentially using the 10% criterion above.
c) Examine forced-photometry lightcurves of Objects.  Quantify the number of Objects that are detected in only 10% of the images.

Derived from Requirements:
OSS-REQ-0157


5. Measurements of Blended Objects

Specification: The Observatory shall be capable of measuring properties of overlapping (blended) objects.  Any degradation of accuracy and precision in measurement of blended objects shall be bounded by the need to satisfy LSR-REQ-0043, LSR-REQ-0044, LSR-REQ-0045, and LSR-REQ-0046.

Discussion: This requirement will give rise to a deblender software component, or an algorithm to perform simultaneous measurements of blended objects. While it is extremely difficult to write quantitative yet all-encompassing and actionable requirements on the deblender, it is understood that it is primarily an enabling component to perform measurements and statistical studies involving those measurements. Therefore, its performance will be evaluated by assessing the efficacy of measurements that employ it, and studies that depend on those measurements (e.g., in the context of weak lensing).

Derived from Requirements:
OSS-REQ-0155



####

Note: The specification parameter Names above are directly from the OSS.  We note that there has been a decision in the project to update our terminology "Level 1"->"Prompt Processing" and "Level 2"->"Data Release Processing".  We of course agree with this decision, but have not change the specification name ("dmL1AstroErr") here under the decision that it's better to use the same names for the same quantities across multiple documents.

### Clarifications

#### Issues
* This one is in concept fine, but the suggested size, shape, and performance numbers need to be updated.
* Could consider adding outlier fraction requirements
* What about blending?
1. Galaxy Shape and Size Measurements for Sersic Profiles
For a galaxy with elliptical isophots and Sersic radial profile with radius *ellipTestRadius* and Sersic index *ellipTestSersicIndex*, the ellipse parameters shall be measured to an accuracy *ellipAccuracy1* for galaxies detected on the coadd at a signal-to-noise *ellipTestSNR1*, and *ellipAccuracy2* for galaxies detected on the coadd at a signal-to-noise *ellipTestSNR2*.

| Description | Value | Unit | Name |
|-------------|-------|------|------|
| Maximum RMS error on ellipse parameters measured for a galaxy of SNR *ellipTestSNR1* | 1 | percent | *ellipAccuracy1* |
| Maximum RMS error on ellipse parameters measured for a galaxy of SNR *ellipTestSNR2* | 0.01 | percent | *ellipAccuracy2* |
| Galaxy radius definition for the ellipse parameter accuracy test | 5 | arcsecond | *ellipTestRadius* |
| Galaxy Sersic index definition for the ellipse parameter accuracy test | 1 | float | *ellipTestSersicIndex* |
| Reference SNR #1 for ellipse parameter accuracy test | 10 | float | *ellipTestSNR1* |
| Reference SNR #2 for ellipse parameter accuracy test | 10000 | float | *ellipTestSNR2* |

Discussion: While the detection will be performed on full-depth coadds, the measurement may use techniques, such as multifit, that perform measurements using individual epoch data.

This requirement can be verified using simulated inputs.  Insert fake galaxies with known parameters into either simulated or precursor datasets, and test how well the input parameters are recovered.  While the requirement is defined at a specific size and Sersic index, the injected galaxies should span use a realistic distribution of shape and size distributions.



### Substantial concerns with OSS

One is tempted to write the following DMSR requirements following OSS-REQ-0275 to cover requirements on photometric processing from the DM processing.  However, it is very difficult to measure the specific contribution of the processing pipeline and we are not sure how to do so.  It's unclear what to even calculate (i.e., the _functional_ requirement).
Even pixel-level simulations may not be sufficient to verify a _performance_ requirement.

There is a basic information-theory level question underlying this uncertainty.  We don't have a completely predictive model for how well we _should_ be able to do given a certain set of information.  It is thus difficult to ascribe a given fraction of the error to the data processing pipelines themselves.

It is possible that if a more complete model for the different limiting uncertainties were available, then different portions of the remaining uncertainty could be assigned to different error sources, including the DM processing.  However, if these additional error sources are merely constant
additional uncertainties added in quadrature, then it will be hard to justify assigning to one versus the other.  Only if different sources of uncertainty in the model have different dependencies on parameters we will be able to separate out the contributions.  It is certainly even likely that the uncertainty introduced by DM processing will not be a constant error to be added in quadrature -- it could, for example, be a function of PSF, airmass,  particular turbulence patterns, particular atmospheric absorption that's difficult to correct.  These requirements then become on the effective average over the distribution of observations of bright, isolated stars over these varying dependent parameters.

#### Why is photometry hard and astrometry easier?
These concerns are particularly acute for the photometric requirements.  The effects of astrometry are simpler and better understood and lend themselves better to writing requirements about the uncertainty being added by the data processing.  These astrometric requirement functional and performance requirements are not strictly clearly exactly right, but they are close enough that we believe these are reasonable.  We thus have more confidence in the requirements we are recommending to add relating to astrometry.  However, the additional complexity of photometric performance pushes things beyond the level of reasonable estimation.

1. Contribution to photometric errors in Prompt Processing

Specification: Data processing shall contribute no more than *dmL1PhotoErr* to point-source photometric errors in Prompt Processing data products.  These requirements apply to the relative errors in repeat observations and are separate from the absolute calibration requirements.

Discussion: This requirement can be tested with simulation, and in commissioning using repeated observations of one or more fields.   

Determining the contribution of the data processing versus other contributions can be difficult.  Certainly, if the overall systematic error can be demonstrated to be less than the requirement, then the requirement will have been passed.  However, there will be an additional irreducible uncertainty floor set by the atmosphere.

For simulations, this effect can be precisely modeled and known and the contribution from the data processing pipeline estimated.

For photometry, the full uncertainty expectation is more detailed, but the same principle of identifying a systematics floor may hold.  The objects to measure for this requirement should be stars with well-understood SEDs.

| Description | Value | Unit | Name |
|-------------|-------|------|------|
| Maximum contribution from DM to Level 1 point source photometric errors | 6 | millimagnitude | *dmL1PhotoErr* |

Derived from Requirements: OSS-REQ-0149


2. Contribution to photometric errors in Data Release Processing

Specification: Data processing shall photometrically calibrate raw image data such that the data processing contributes no more than the allocations in the table procCalAlloc for repeatability (procCalRep), uniformity (procCalUniformity), and relative accuracy (procCalZP).

Discussion: These allocations include effects from calibration algorithms, errors and noise in producing the necessary calibration data products, as well as errors and uncertainties in any reference catalogs used in the calibration process.

These will be very challenging goals to verify.

| Description | Value | Unit | Name |
|-------------|-------|------|------|
| The maximum allowed RSS contribution to the overall photometric repeatability of bright isolated point sources caused by errors introduced in the data processing pipelines. | 0.003 | AB magnitude | procCalRep |
| The maximum allowed RSS contribution to the overall uniformity across the sky of the photometric zero-point of bright isolated point sources introduced in the data processing pipelines. | 0.007 | AB magnitude | procCalUniformity | 
| The maximum allowed RSS contribution to the overall accuracy of the color zero points of bright isolated point sources introduced by the data processing pipelines. | 0.003 | AB magnitudes | procColorZP |

Derived from Requirements: OSS-REQ-0275

------
More generally, the following OSS Requirements are reasonable but not independently verifiable:
OSS-REQ-0275,
OSS-REQ-0336,
OSS-REQ-0276,
OSS-REQ-0277,
OSS-REQ-0334,
OSS-REQ-0282
