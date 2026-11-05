# DECaLS Pipeline

Package prepared from the uploaded `decals_pipeline_class.py`. The unchanged source is retained in `original/`.

## Install
```bash
pip install -e .
```
Restart the notebook kernel. Remove or replace an older local `decals_pipeline_class.py` that shadows the installed module.

## Existing-data workflow
```python
from decals_pipeline_class import DecalsPipeline
pipe = DecalsPipeline()
pipe.run(galaxies_new, size=3000, skip_download=True, do_plot=False,
         do_psf_split=False, force_download=False, do_save_split=True)
```
Define `galaxies_new` as your galaxy-name list. This exact call is supported.
It saves g/r/z images and inverse-variance files, without plots or PSF splitting.
Original skip semantics are preserved: downloads are skipped only when the image file exists. Missing images trigger downloading. Existing images with missing inverse variance raise an error during splitting. `force_download=True` always requests downloading.
`size` is forwarded to the external downloader and does not resize existing images. Its units and service limits cannot be confirmed without that downloader.

## Required input layout
For a name with spaces removed, e.g. NGC6902:
- `decals_data/downloads/fits/NGC6902.fits`: primary cube `(3, ny, nx)`, g/r/z order.
- `decals_data/downloads/invvar/NGC6902_invvar.fits`: g/r/z inverse variance in HDUs 2/4/6.
- Optional `decals_data/downloads/psf/NGC6902_psf.fits`: three 2-D image HDUs in g/r/z order, required only for plotting or PSF splitting.

These layouts come from the uploaded class; they have not been checked against actual FITS files. Ambiguous PSF cubes are rejected rather than assigned a guessed band axis.

## Outputs
- `decals_data/outputs/images/<name>_<band>.fits`
- `decals_data/outputs/invvar/<name>_<band>_invvar.fits`
- Optional `decals_data/plots/<name>.png`
- Optional `decals_data/downloads/psf/<name>_PSF_<band>.fits`

Existing outputs are overwritten. Splitting does not resample or recalibrate pixel values. Celestial WCS is reduced to two dimensions; inverse-variance headers are taken from each band's own HDU. Plotting sigma is computed as `1/sqrt(invvar)` for finite positive weights; invalid weights give NaN, not zero uncertainty. Sigma maps are not exported.

## Download dependency still needed
The original code imports `DECaLSQuery` from `decals_query.py`. That file was not supplied and was not found in the GitHub search. This package imports it only when downloading, so existing-data operations work independently. Supply the original module on the Python path to enable downloads. The default layer remains `ls-dr9`. No substitute archive API or undocumented FITS layout has been invented.

## Source corrections
- Optional downloader import; existing-data use no longer fails at import time.
- Numeric and legacy sexagesimal SIMBAD coordinate columns supported.
- FITS handles closed after loading.
- Correct band-specific inverse-variance headers and two-dimensional image WCS.
- Consistent three-HDU PSF handling; unsupported layouts raise clear errors.
- Invalid inverse variance does not imply zero uncertainty.
- Stage failures propagate to the notebook.

Source reviewed only. No tests, downloads, or pipeline executions performed. Actual image units, band ordering, astrometric alignment and scientific outputs still need verification with the input FITS products. No license has been selected.
