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

## Deployment

GitHub Pages is served from the `gh-pages` branch of this repository
(Settings -> Pages -> Source: `gh-pages` / root). Rebuild with the command above
and push the contents of `_site/` to that branch.
