# Build & deploy

End state: pushing to `main` updates <https://drp1.omicable.com/> within ~30s.

## One-time setup

### 1. Create the publishing repo
On GitHub, create a public repo — suggested name `gama-drp1-dashboard` (owner: whichever account you want associated with the paper link long-term).

### 2. Push these staging files

```bash
cd staging/omicable-drp1
git init -b main
git add .
git commit -m "initial: landing, citation, license, CNAME"
git remote add origin git@github.com:<user>/gama-drp1-dashboard.git
git push -u origin main
```

### 3. Add the dashboard from the Codespace
From inside the `Gama_Vivian_DRP1_bulkRNAseq` Codespace:

```bash
# clone the publishing repo alongside the analysis repo
cd /workspaces
git clone git@github.com:<user>/gama-drp1-dashboard.git

# copy the generated dashboard
cp Gama_Vivian_DRP1_bulkRNAseq/03_Results/02_Analysis/Plots/Trajectory_Flow/interactive_bump_dashboard.html \
   gama-drp1-dashboard/dashboard.html

# pin Plotly locally — the CDN URL is in the generated HTML
cd gama-drp1-dashboard
curl -L https://cdn.plot.ly/plotly-2.27.0.min.js -o plotly-2.27.0.min.js

# confirm the exact CDN string in dashboard.html, then rewrite to a relative path
grep -o 'https://cdn.plot.ly/plotly-[0-9.]*\.min\.js' dashboard.html | head -1
sed -i 's|https://cdn.plot.ly/plotly-2.27.0.min.js|./plotly-2.27.0.min.js|g' dashboard.html

# sanity check: should print nothing
grep -c 'cdn.plot.ly' dashboard.html

git add dashboard.html plotly-2.27.0.min.js
git commit -m "add dashboard + pinned Plotly runtime"
git push
```

If the Plotly version in the generated HTML is not `2.27.0`, change both the `curl` URL and the `sed` pattern to match. The version string is whatever appears between `plotly-` and `.min.js` in the `<script>` tag.

### 4. Enable GitHub Pages
Repo → Settings → Pages:
- **Source**: Deploy from a branch
- **Branch**: `main`, folder `/ (root)`
- Save. Wait ~30 s for the first build.

You'll temporarily see `https://<user>.github.io/gama-drp1-dashboard/` work — that's expected before DNS.

### 5. Point DNS
At omicable.com's registrar, add:

| Type | Name | Value | TTL |
|---|---|---|---|
| CNAME | `drp1` | `<user>.github.io.` | 3600 |

Trailing dot matters on most registrars. No A records needed for a subdomain.

### 6. Bind the custom domain
Back in Settings → Pages:
- **Custom domain**: `drp1.omicable.com` → Save. (Repo already contains the `CNAME` file, so this is just the UI mirror.)
- Wait for the DNS check (green tick — usually 1–10 min, occasionally longer).
- Tick **Enforce HTTPS** once the Let's Encrypt cert provisions (separate wait, up to ~20 min).

When all three are green, <https://drp1.omicable.com/> is live.

## Per-regeneration refresh

When `bump_dashboard` regenerates the figure:

```bash
cp /workspaces/Gama_Vivian_DRP1_bulkRNAseq/03_Results/02_Analysis/Plots/Trajectory_Flow/interactive_bump_dashboard.html \
   /workspaces/gama-drp1-dashboard/dashboard.html

cd /workspaces/gama-drp1-dashboard
sed -i 's|https://cdn.plot.ly/plotly-2.27.0.min.js|./plotly-2.27.0.min.js|g' dashboard.html
git add dashboard.html && git commit -m "refresh dashboard" && git push
```

Consider wiring this into a `make publish-dashboard` target in the analysis repo so a single command regenerates + syncs + pushes.

## On paper acceptance

When the manuscript is accepted, three things to update in this repo:

1. `index.html` — replace the "Resubmitted… Manuscript in review." line with the final citation and journal DOI.
2. `CITATION.cff` — fill in `title`, `volume`, `pages`, and add a second `identifiers:` entry for the journal DOI.
3. `README.md` — same update to the citation block.

Cut a fresh Zenodo release after these changes so the snapshot reflects the published version.

## Health checks

```bash
# DNS resolves to GitHub Pages?
dig +short drp1.omicable.com
# Expect: <user>.github.io.  →  one of 185.199.108.153 / .109.153 / .110.153 / .111.153

# HTTPS works and serves your page?
curl -sI https://drp1.omicable.com/ | head -1
# Expect: HTTP/2 200

# Plotly really pinned?
curl -s https://drp1.omicable.com/dashboard.html | grep -c 'cdn.plot.ly'
# Expect: 0
```
