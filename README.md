# A repository for Aussie MWA EoR srclists 

Current suggested sky model to use is:

```
srclist_pumav3_EoR0LoBES_EoR1pietro_CenA-GP_2023-11-07.fits
```

## Hyperdrive / WODEN sky models

### 13/10/2026
`srclist_pumav3_EoR0LoBES_EoR1pietro_CenA-GP_2023-11-07_no-shapelets.yaml`

A version of the 07/11/2023 hyperdrive sky model with **no shapelet components**, for
predictors that only support point and Gaussian components (DP3 in particular, whose
`sky_model/SkyModelFunctions.cc` throws "Only point sources and Gaussian sources are supported"
for anything else). Everything that was not a shapelet (337k points, 15k Gaussians) is
byte-for-byte unchanged.

 - **Fornax A**: the 2 shapelet components (3828 coefficients each, 559 Jy at 200 MHz once the
   shapelet zero-spacing factor of 0.993 is applied) and 5 small Gaussians (2.2 Jy) were replaced
   by the WODEN multiscale-CLEAN model
   [`examples/FornaxA/srclist_msclean_fornaxA_phase1+2.txt`](https://github.com/JLBLine/WODEN/blob/master/examples/FornaxA/srclist_msclean_fornaxA_phase1%2B2.txt)
   converted with `hyperdrive srclist-convert -i woden`: 4544 points + 1736 Gaussians, reference
   frequency 188.79 MHz, spectral index -0.8 (hyperdrive's default; the WODEN file has one
   frequency per component). Caveat: this model integrates to 468 Jy at 200 MHz, about 17% less
   than the shapelet model.
 - **The other 59 shapelet components** (55 EoR1 sources, 135 Jy in total, mostly 45
   coefficients / order 8; brightest nEXT035140-2743 at 21.6 Jy) were each fitted with a set of
   Gaussians using
   [mwa-demo/shapelet2gauss](https://github.com/MWATelescope/mwa-demo/tree/c6ce36a9b7b2eff944ed233af47605dd91fccf04/shapelet2gauss)
   ([PR](https://github.com/MWATelescope/mwa-demo/pull/3)): 6114 Gaussians in total, median 107
   per component. The fit is a sign-constrained matching pursuit over a dictionary of 8 Gaussian
   shapes, solved in the uv plane with hyperdrive-exact shapelet and Gaussian envelopes
   (validated to 1e-5 against `hyperdrive vis-simulate`), weighted by the real baseline
   distribution of MWA obs 1443210064 (Phase II compact, max 741 m) plus a uniform fill out to
   2000 wavelengths, with the zero spacing carrying 5% of the weight. Sign constraints are
   what make this work: a free nonlinear Gaussian fit finds cancelling +/-7000 Jy pairs for a
   2 Jy source, which break under the phase-centre uv projection that hyperdrive and DP3 use
   for Gaussians.
 - Fit quality: on baselines under 600 wavelengths the rms error is 0.06% (median) to 0.14%
   (worst) of the peak visibility, zero-spacing flux error under 0.05%. Simulating 7 of the
   sources (including the 3 brightest) with hyperdrive from the shapelet and the Gaussian
   versions gives 0.04-0.17% rms and under 0.9% maximum visibility differences at the
   1443210064 phase centre. Per-component numbers are in the tool's
   `fit_report_2023-11-07_srclist.txt`.
 - Limitations: the shapelet models ring strongly beyond ~1500 wavelengths (their images,
   band-limited at 5000 wavelengths, have negative lobes larger than the peak); that structure
   is only reproduced approximately (0.5-1% rms out to 2000 wavelengths). For Phase II extended
   data, re-run the tool with that array's baseline lengths and a larger `--uvmax`.
 - The `.fits` version of the 07/11/2023 model was not converted.

`srclist_pumav3_EoR0LoBES_EoR1pietro_CenA-GP_2023-11-07_no-shapelets.skymodel`

The same catalogue as a DP3 / BBS text sky model (the `makesourcedb` format, readable directly by
DP3 with `predict.sourcedb=<file>.skymodel`, no `makesourcedb` step). How it was made:

 - `hyperdrive srclist-convert -o ao` cannot be used: hyperdrive's AO writer skips every source
   with a curved power law, which is 77% of the components here (271898 of 354340). It was
   written instead by
   [`yaml2dp3.py`](https://github.com/MWATelescope/mwa-demo/blob/557a06e/shapelet2gauss/yaml2dp3.py)
   from `mwa-demo/shapelet2gauss` (same tool directory as the shapelet fitter):
   ```
   python yaml2dp3.py srclist_pumav3_EoR0LoBES_EoR1pietro_CenA-GP_2023-11-07_no-shapelets.yaml \
          srclist_pumav3_EoR0LoBES_EoR1pietro_CenA-GP_2023-11-07_no-shapelets.skymodel --patch single
   ```
 - Header: `Format = Name, Type, Patch, Ra, Dec, I, Q, U, V, ReferenceFrequency, SpectralIndex,
   LogarithmicSI, MajorAxis, MinorAxis, Orientation`. Ra is written as `hh:mm:ss.sssss`, Dec as
   `+dd.mm.ss.sssss`. Component names are `<hyperdrive source name>_<index>`.
 - Spectral models, all with `LogarithmicSI=true` and the component's own `ReferenceFrequency`
   (200 MHz for the catalogue, 188.79 MHz for the Fornax A model):
   hyperdrive `power_law` S = S0 (ν/ν0)^a becomes `SpectralIndex=[a]`; hyperdrive
   `curved_power_law` S = S0 (ν/ν0)^a exp(q ln²(ν/ν0)) becomes `SpectralIndex=[a, q·ln(10)]`,
   because DP3 evaluates S0 (ν/ν0)^(c0 + c1 log10(ν/ν0)) (`base/PointSource.cc`). Both are
   exact, no sampling.
 - Gaussians: `MajorAxis`/`MinorAxis` are the FWHM in arcsec and `Orientation` the position
   angle in degrees, unchanged from the YAML. `OrientationIsAbsolute` is left at DP3's default
   (false), which matches hyperdrive's phase-centre-frame Gaussian evaluation.
 - Patches: this file puts everything in one patch named `sky` (`--patch single`). The tool's
   default makes one patch per hyperdrive source (the natural direction-dependent unit, with the
   patch centre on the brightest component), but DP3's sky-model loader does not scale to that:
   with 338797 patches DP3 needs 24 minutes just to read the file, against 49 s for the single
   patch. Make per-source patches on a beam-selected cut (a few hundred to a few thousand
   sources load in seconds), e.g. `hyperdrive srclist-by-beam -m <metafits> -n 1000 <yaml>`
   followed by `yaml2dp3.py` (cut first, then convert).
 - DP3 applies no horizon cut: with the all-sky catalogue 97828 of the 360395 components are
   behind the tangent plane of an EoR1 phase centre and DP3 still predicts them. Cut the model
   to the field before use.
 - Validation: DP3 `predict` (`usebeammodel=false`) from this file on an MS of obs 1443210064
   was compared with an independent direct Fourier sum of the file's own contents using the MS
   phase centre and UVW column (DP3's spectral formula, signed n term): they agree to 0.00012%
   rms, maximum 0.1 Jy on 8.1 kJy visibilities, over all 360395 components. For the 5000
   brightest sources the same sum reproduces DP3 to 0.0000% (max 1 mJy), while
   `hyperdrive vis-simulate` of the YAML deviates from it by 0.2-0.8% rms because hyperdrive's
   visibilities are not exactly in the frame of the UVWs it writes to the MS (hyperdrive PR 92
   adds `--uvw-frame casacore` for this).

### 07/11/2023
`srclist_pumav3_EoR0LoBES_EoR1pietro_CenA-GP_2023-11-07.fits` \
`srclist_pumav3_EoR0LoBES_EoR1pietro_CenA-GP_2023-11-07.yaml`
 - LoBES (Lynch et al 2021) in EoR0 field
 - Procopio et al 2017 with a few added shapelet models in EoR1 field (including Fornax A model from Line et 2020)
 - Cook et all 2022 for Cen A and Galactic Plane SNR
 - Everything is either a power-law or curved power-law SED

## RTS sky models (deprecated)

### 28/06/2019
`srclist_pumav3_EoR0aegean_EoR1pietro+ForA_phase1+2.txt`
 - Added a much improved Fornax A model using both phase 1 and phase 2 extended data

`srclist_pumav3_EoR0aegean_EoR1pietro+ForA_phase1+2_TGSSgalactic.txt`
 - New ForA model plus attempt at filling galactic plane with TGSS sources

### 12/07/2018
`srclist_pumav3_EoR0aegean_EoR1pietro+ForA_TGSSgalactic.txt`

 - Attempt to fill in the galactic plane with TGSS sources

### 16/02/2017
`srclist_pumav3_EoR0aegean_EoR1pietro+ForA.txt`

 The main features of this srclist are:
   + Based off of the GLEAM catalogue
   + Matched to TGSS, VLSSr, SUMMS, and NVSS
   + Has PUMA based extended models up to 20 deg away from EoR0
   + Has Pietro's extended models up to 10 deg away from EoR1
   + Outside of those areas, it is the PUMA matched GLEAM catalogue
     if a GLEAM source failed the PUMA tests, it has been included
     but just as a GLEAM source - it has not been eyeballed
   + Also has Jenny's shapelet model for Fornax A

### 02/02/2015 - J Line
`srclist_puma-v2_complete.txt`

 - Added the most up to date and complete PUMA srclist,
   srclist_puma-v2_complete.txt. This srclist covers the
   full southern sky, using either mwacs or mrc as a base.
   It is complete with point source extended models between
   22h < RA < 6h, -57deg < Dec < +3deg (2 hours about EoR0
   and EoR1). Elsewhere, only the sources accepted by PUMA
   are currently included.
   
 - In ./puma_v2, the separate srclists used to create
   srclist_puma-v2_complete.txt are stored and explained
   
 - In ./extended, information and plots of all the extended
   sources in srclist_puma-v2_complete.txt are included
   
 - srclist_by_beam.py will take a srclist, and return a
   given number of sources in a megapatch srclist. These 
   will be brightest sources once convolved with the beam, 
   as defined by the pointing given by a designated metafits
   file. The sources are also confined to a user specified
   distance of the pointing centre. To use the plotting option 
   (-p), wscaxes module is needed.
