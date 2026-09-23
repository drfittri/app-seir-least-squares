# SEIR Least Squares (Shinylive)

Interactive demo: simulate a basic SEIR model and fit it to observed COVID-19
daily incidence from March 2020 by eye, using the sum of squared errors (SSQ)
as the objective. Runs entirely in the browser via
[Shinylive](https://posit-dev.github.io/r-shinylive/) (WebAssembly R, no server).

**Live:** https://drfittri.github.io/app-seir-least-squares/

## Credit

Original app by **[Spectrum Spark](https://github.com/spectrum-spark)**:
[spectrum-spark/app-seir-least-squares](https://github.com/spectrum-spark/app-seir-least-squares).

This repository is a hosted copy of that work. All app code, design, and the
logo are the original author's. The only changes made here are:

- a footer credit link added to the app UI (`app.R`)
- this README and `NOTICE`

Upstream is the authoritative source. No license file was present in the
original repository, so all rights remain with the original author; this copy
exists for hosting only and will be removed on request.

## Run locally

```r
install.packages("shiny")
shiny::runApp()
```

## Build the static site

```r
# shinylive >= 0.3.0
shinylive::export(".", "_site")
```

Then serve `_site/` over HTTP (`python3 -m http.server`). Opening
`_site/index.html` directly from disk will not work: Shinylive needs HTTP for
its service worker and asset fetches.

Note: export from a staging directory containing only `app.R`, `DESCRIPTION`
and `www/`. Exporting the repo root also copies `test_app/` into the site and
inflates `app.json` to ~129 MB (this is what upstream's published site does).

## Deploy to Posit Connect

`manifest.json` is committed and ready. It declares `appmode: shiny`, pins R
`4.5.1`, and lists 46 CRAN packages. Only two content files are listed
(`app.R`, `www/logo.png`), so the 93 MB `test_app/` directory is never uploaded.

Git-backed deployment: point Connect at this repository's `main` branch. Connect
reads `manifest.json` to decide what to fetch, so the rest of the repo is
ignored.

Push-button deployment:

```r
rsconnect::deployApp(
  appDir = ".",
  appFiles = c("app.R", "www/logo.png"),
  appName = "seir-least-squares"
)
```

### Regenerating the manifest

```r
rsconnect::writeManifest(
  appDir = ".",
  appFiles = c("app.R", "www/logo.png"),
  appMode = "shiny"
)
```

Two traps, both hit while creating the committed manifest:

1. **Do not pass `DESCRIPTION` in `appFiles`.** `DESCRIPTION` declares
   `Package: sirleastsquares`, which makes rsconnect treat the directory as an R
   package and inject a `.Rbuildignore` entry into the manifest. That file does
   not exist, so Connect rejects the deployment. `DESCRIPTION` exists only for
   the Shinylive export path and is not needed by Connect.
2. **`munsell` must be installed locally when regenerating.** `app.R` contains a
   dead `if (FALSE) { library(munsell) }` block, which dependency scanning still
   detects. If the package is absent, renv cannot pin its version and the
   manifest is incomplete. Install it into a scratch library first:
   ```r
   .libPaths(c("/tmp/rlib", .libPaths()))
   install.packages("munsell", lib = "/tmp/rlib")
   ```
   (`shinythemes` is declared in `DESCRIPTION` but never used, so it is
   correctly absent from the manifest.)

If the Connect server does not have R 4.5.1, either install that R version there
or edit `platform` in `manifest.json` to match an available version.

## Deployment (GitHub Pages)

GitHub Pages is served from the `gh-pages` branch
(Settings -> Pages -> Source: `gh-pages` / root). `.nojekyll` is included.

Two ways to deploy:

1. **Manually.** Run the export above, then push the contents of `_site/` to the
   `gh-pages` branch.
2. **Workflow.** `.github/workflows/build_and_deploy_app.yml` rebuilds and
   publishes on every push to `main`. It is the upstream workflow rewritten to
   deploy to the `gh-pages` branch.

   GitHub disables Actions on forks until the owner enables them once: open the
   **Actions** tab of this repository and click "I understand my workflows, go
   ahead and enable them". Until that is done, use route 1.

## First-load time

The first visit downloads roughly 88 MB of WebAssembly assets and then boots R
in the browser before the app appears, so expect tens of seconds. Later visits
are served from the service worker cache. This is inherent to Shinylive, not a
misconfiguration.
