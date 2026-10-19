# A repository for Aussie MWA EoR srclists 

Current suggested sky model to use is:

```
srclist_pumav3_EoR0LoBES_EoR1pietro_CenA-GP_2023-11-07.fits
```

For EoR1 high-band (167-197 MHz) Phase II/III compact data, use the catalogue with the fitted
Fornax A model instead ([16/10/2026](#16102026)):

```
srclist_pumav3_EoR0LoBES_EoR1pietro_CenA-GP_2023-11-07_no-shapelets_FnxA-v2-20260905.yaml
```

## DP3 / hyperdrive compatible sky models

Sky models in this section contain only point and Gaussian components (no shapelets), so they work in
DP3 as well as hyperdrive. Each is provided as a hyperdrive `.yaml` and a DP3 `.skymodel`.

### 16/10/2026
`srclist_pumav3_EoR0LoBES_EoR1pietro_CenA-GP_2023-11-07_no-shapelets_FnxA-v2-20260905.yaml` \
`srclist_pumav3_EoR0LoBES_EoR1pietro_CenA-GP_2023-11-07_no-shapelets_FnxA-v2-20260905.skymodel` \
`FornaxA_v2-20260905.yaml` / `FornaxA_v2-20260905.skymodel` (the Fornax A model alone)

The 13/10/2026 no-shapelets catalogue with **Fornax A replaced by a Gaussian model fitted to MWA
visibilities at all baselines**. It supersedes the 14/10/2026 self-cal model, which leaves a striped
residual across Fornax A (see below). 106 Gaussian components, 426.3 Jy at the 200 MHz reference
frequency (458.9 Jy at 183 MHz), flux-weighted spectral index -0.82 (at 200 MHz). Unlike the 14/10/2026 file, the 44
catalogue sources within 1.3 deg of Fornax A are kept: the new model only describes Fornax A itself.
Everything else is unchanged from the no-shapelets catalogue.

Why the self-cal model needed replacing: about 15 of its 75 point components add up in phase at
~310 wavelengths (position angle 60-68 deg), a part of the uv plane the Phase III compact array samples
only with two baselines (Tile087-Tile097 and Tile087-Tile098). There the model predicts 21-30 Jy where
the data show 5-11 Jy, which images as an ~11 arcmin fringe pattern across Fornax A after subtraction.
Refitting the fluxes of its components does not remove it; the component layout is the problem.

How it was made:

 - Data: 21 EoR1-high observations of 2026-09-05 at gridpoints 0, 2 and 4 (training set), 167-197 MHz,
   Phase III compact, calibrated with the 07/11/2023 catalogue with Fornax A x0.80. Every other
   source was peeled (hyperdrive, 1000 sources with ionospheric offsets) and Fornax A added back from
   a simulation of the x0.80 model with the peel's own gain and offsets.
 - Model: 173 Gaussians of 220 arcsec FWHM on a 180 arcsec grid over the self-cal model's footprint,
   plus its 21 large (747 arcsec) Gaussians. Positions fixed; fluxes (>= 0) and spectral indices
   ([-2.5, 1], weak prior toward -0.8) fitted in visibility space with each tile's FEE beam and
   Fornax A's peel ionospheric offsets, using all baselines from 20 wavelengths up (~490 at most)
   with equal total weight per |uv| bin. 106 of the 194 components are non-zero. Model choice by
   leave-one-pointing-out cross-validation within the training set.
 - Flux scale: data/model 1.002 on held-out baselines, flat to 0.5% across the band and 1.00-1.01 from
   10 to 200 wavelengths; the night-median peel gain on Fornax A is 0.99 (x0.80 gives 1.03). The total
   is 5% below the x0.80 model, all of it below 10 wavelengths, which no stage constrains.

Validation on held-out observations (gridpoints 6, 15 and 31 of the same night, never used in the fit):

 - Fornax A residual after the full pipeline peel (3 observations combined, dirty image rms in a
   4 deg box): 0.048 Jy/beam, against 0.085 for the
   x0.80 model, 0.094 for the 07/11/2023 catalogue and 0.250 for the 14/10/2026 self-cal model. No
   stripes.
 - Power spectrum (36 observations, CHIPS, after hyperdrive DI calibration and 1000-source ionospheric
   peel, each normalised by its own calibrated power so the flux-scale change cancels): foreground-wedge
   power 0.46 (XX) / 0.47 (YY) of the 07/11/2023 catalogue's, against 0.98 / 0.96 for x0.80 and
   0.48 / 0.49 for the self-cal model; the self-cal model's excess at k_perp 0.15-0.3 h/Mpc is gone.
   EoR-window power is unchanged within the ~5% scatter.
 - A signal-injection test of the same calibration and peel chain finds no loss of a 21-cm-like
   signal in the EoR window for the 07/11/2023 and self-cal catalogues (transfer 0.97-1.03 once the
   flux scale is divided out).

Limitations: no structure above ~490 wavelengths or below 10 wavelengths, so for Phase II compact /
Phase III compact data, not Phase II extended; spectral indices fitted over 167-197 MHz only, do not
extrapolate to the low band without checking; Stokes I only; fitted to one night.

Used by the MWAEoR-Pipeline EoR1-high configuration (`eor1high-ph3.conf`, `dical_name = "30l_src8k_300it_fnxav2"`)
from 17/10/2026. Full validation report, with power spectra and residual images:
https://claude.ai/artifact/DuyC5BM5W5kMFJzMYRXdNF

The `.skymodel` files follow the 14/10/2026 conventions (full catalogue in one patch named `sky`,
Fornax A file with its own `FornaxA` patch, power laws referenced to 200 MHz, Gaussian axes to 9
significant figures). Both `.yaml` files convert to FITS with `hyperdrive srclist-convert -o fits`.

### 14/10/2026
`srclist_pumav3_EoR0LoBES_EoR1pietro_CenA-GP_2023-11-07_no-shapelets_FnxA-selfcal-20260905x087.yaml` \
`srclist_pumav3_EoR0LoBES_EoR1pietro_CenA-GP_2023-11-07_no-shapelets_FnxA-selfcal-20260905x087.skymodel` \
`FornaxA_selfcal-20260905x087.yaml` / `FornaxA_selfcal-20260905x087.skymodel` (the Fornax A model alone)

The 13/10/2026 no-shapelets catalogue with **Fornax A replaced by a model self-calibrated from MWA
data** (16 zenith-pointed EoR1-high observations of 2026-09-05, 1472677088 to 1472678888, 167-197 MHz,
Phase III 128-tile compact layout, 741 m longest baseline). 96 components (75 points, 21 Gaussians,
415.7 Jy at the 200 MHz reference frequency, 448.9 Jy at 183 MHz, flux-weighted spectral index -0.88) covering 1.3 deg around Fornax A; the 44
catalogue sources inside that radius were removed because the self-cal model already contains them.
Everything else is unchanged from the no-shapelets catalogue.

How it was made:

 - Input: the pipeline-calibrated 8 s / 80 kHz uvfits of the 16 obs (Pawsey `high1.uvfits`,
   `hyp_<obs>_ssins_30l_src8k_300it_8s_80kHz.uvfits`, hyperdrive DI calibration against the
   07/11/2023 shapelet catalogue), converted with `hyperdrive vis-convert` and concatenated in
   time with casacore `msconcat` (materialised with DP3 `msout.uvwcompression=false`).
 - Self-calibration with [rapthor](https://git.astron.nl/RD/rapthor) in the MWA build
   `ghcr.io/d3v-null/rapthor-mwa:sha-48f973a-pass` (DP3 6.6 + EveryBeam MWA beam + WSClean 3.6;
   [usage guide](https://gist.github.com/d3v-null/63eb2255d8823783866da9e77debfce0)): one
   direction, phase-only DDECal (56 s / 1 MHz solutions, 3 MHz smoothness, baselines over 30
   wavelengths) against the catalogue (top 1000 sources by apparent flux, Gaussians under 0.3 Jy
   dropped), then WSClean multiscale imaging of a 28 deg field at 110 arcsec cells (Briggs 0,
   baselines over 15 wavelengths, 4 MHz output channels with a 3-term spectral fit) and PyBDSF
   source extraction. A second cycle seeded with the first cycle's model reproduces it to three
   decimals, so the model is the converged one.
 - Flux scale. (1) WSClean's `-apply-primary-beam` for MWA data was found to have the right beam
   shape but a normalisation of ~0.023 at zenith, so rapthor's "true sky" fluxes were 40x too
   high; the true-sky model was rebuilt as apparent flux divided by the zenith-normalised FEE
   beam (`mwa_hyperbeam`, delays of the observation, block mid-time, Stokes-I power). (2) A
   model self-calibrated from data that were calibrated with the shapelet catalogue inherits that
   catalogue's Fornax A flux scale (Fornax A measured at 516 Jy, with the rest of the field 12-17%
   above the catalogue). The published model is therefore the self-cal model times **0.87** =
   0.97 / 1.12, where 0.97 is the least-squares ratio of the model to Fornax A in independent data
   (obs 1452344160, 2026-01-13) and 1.12 the ratio of the field sources to the catalogue in the
   same data, i.e. Fornax A is put on the field catalogue's flux scale (the same correction the
   empirical "Fornax A x0.80" shapelet model applies).
 - Validation on obs 1452344160 (a different night, same array): data/model least-squares scale
   0.94-0.97 and flat from 15 to 450 wavelengths before the 0.87 scaling (the shapelet model gives
   0.81 rising to 0.90, the MS-CLEAN model 1.00 falling to 0.78); dirty-image residual after
   subtracting Fornax A 0.07 Jy/beam rms (shapelet 0.26, MS-CLEAN 0.22). Calibrating and
   subtracting the whole catalogue with this model, the CHIPS foreground-wedge power
   (0.01 < k_perp < 0.1 h/Mpc, k_par < 0.1) is 0.39 (XX) / 0.43 (YY) of the original catalogue's,
   against 0.43 / 0.47 for the Fornax A x0.80 shapelet model and 0.56 / 0.59 for the MS-CLEAN
   model, at equal window power. Products:
   `https://projects.pawsey.org.au/high1.uvfits/fornaxafix_1452344160_{G2_calibrated,H2_residual}_selfcalfnxa087.uvfits`.
 - Limitations: the model only contains structure the 741 m array can see (nothing above 450
   wavelengths, 7.5 arcmin resolution), so it is for Phase II compact / Phase III data, not for
   Phase II extended. The per-component spectral indices come from a 3-term fit over 167-197 MHz
   and should not be extrapolated far outside that band. The 0.87 factor is a single-night
   measurement (the 14-night scatter of the equivalent shapelet correction is +/-0.04).
 - The `.skymodel` files are DP3 / makesourcedb text written by
   [`yaml2dp3.py`](https://github.com/MWATelescope/mwa-demo/blob/557a06e/shapelet2gauss/yaml2dp3.py)
   from the `.yaml` (full catalogue with `--patch single`, Fornax A file with its own `FornaxA`
   patch), trailing commas stripped so that lsmtool also reads them. Ra/Dec, spectral and
   Gaussian conventions are as described for the 13/10/2026 file below. The Fornax A components are
   power laws referenced to 200 MHz like the rest of the catalogue (re-referenced exactly from the
   183.02 MHz fit), so that `hyperdrive srclist-convert` can write the FITS format (see below).

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

### Converting these files to other formats

Everything in this section is a hyperdrive `.yaml` (the master copy) plus a DP3 `.skymodel` made from it.
`hyperdrive srclist-convert` (0.8.0, `-i`/`-o` types `hyperdrive, fits, rts, woden, ao, dp3`) is the
tool for the other formats; the output type is taken from the extension, or given with `-o`. What was
tested on these files:

| target | command | what you get |
|---|---|---|
| FITS (the format of the 07/11/2023 `.fits`) | `hyperdrive srclist-convert <file>.yaml <file>.fits` | lossless: all 354167 components, curved power laws kept (7 s, 42 MB). The writer requires power-law flux densities to be given at the reference frequency, which these files satisfy; a model whose power laws are referenced elsewhere makes it panic with "Power-law flux densities must be at the reference frequency". |
| RTS | `hyperdrive srclist-convert -o rts <file>.yaml <file>_rts.txt` | **curved power laws are dropped** (RTS has no such flux model): only 69277 of 338753 sources survive. Fine for `FornaxA_selfcal-20260905x087.yaml` (all power laws, written as flux lists). `--collapse-into-single-source` makes an RTS patch list. |
| WODEN | `hyperdrive srclist-convert -o woden <file>.yaml <file>_woden.txt` | same loss of curved power laws as RTS; the Fornax A model converts completely. |
| AO (Andre Offringa / `aoflagger`-style text) | `hyperdrive srclist-convert -o ao <file>.yaml <file>_ao.txt` | same loss of curved power laws; the Fornax A model converts completely. |
| DP3 `.skymodel` | not writable by hyperdrive 0.8.0 ("Writing dp3-style source lists is not supported"); use [`yaml2dp3.py`](https://github.com/MWATelescope/mwa-demo/blob/557a06e/shapelet2gauss/yaml2dp3.py), which keeps the curved power laws (`SpectralIndex=[a, q ln 10]`, exact). | the `.skymodel` files here. |
| `.skymodel` back to hyperdrive | `hyperdrive srclist-convert -i dp3 <file>.skymodel <file>.yaml` | works (round trip of the Fornax A file reproduces all 96 components and the same flux at 183 MHz). |

For the hyperdrive tools themselves (`di-calibrate`, `vis-simulate`, `peel`, `srclist-by-beam`) the `.yaml`
or the `.fits` can be used directly; for DP3 use the `.skymodel` with `predict.usebeammodel=true`,
`beammode=full` and `usefastpredict=false` (see the 13/10/2026 notes and the rapthor-mwa guide).

## Hyperdrive / WODEN sky models

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
