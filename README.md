# drp1.omicable.com

Interactive trajectory dashboard — companion to:

> Baum T., Bodnya C., Costanzo J., Boulton D., Mogilenko D., Caino M.C., Zhelonkin A.*, and Gama V.* (2025). Resubmitted to the *Journal of Neurodevelopmental Disorders*. Manuscript in review.

Archived snapshot: [doi.org/10.5281/zenodo.20213748](https://doi.org/10.5281/zenodo.20213748)

Live URL: <https://drp1.omicable.com/>

## Layout

| File | Purpose |
|---|---|
| `index.html` | Minimal landing page (citation, link to dashboard). |
| `dashboard.html` | The interactive Plotly figure. Generated upstream, refreshed manually. |
| `plotly-2.27.0.min.js` | Pinned Plotly runtime — page works without the CDN. |
| `CNAME` | GitHub Pages custom-domain marker (`drp1.omicable.com`). |
| `CITATION.cff` | Machine-readable citation metadata. |
| `LICENSE` | CC-BY-4.0 for the figure; MIT for bundled Plotly. |
| `BUILD.md` | One-time setup + per-regeneration refresh instructions. |

## Regeneration

The dashboard is produced by `01_Scripts/Python/bump_dashboard/` in the analysis repo (`Gama_Vivian_DRP1_bulkRNAseq`). After regeneration, copy `interactive_bump_dashboard.html` into this repo as `dashboard.html`, swap the Plotly CDN URL for the local pin, commit, push. See [`BUILD.md`](./BUILD.md).
