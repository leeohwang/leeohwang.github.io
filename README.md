# LUMEN Initiative

**Lift Up Minds, Empower Neighborhoods** — a student-run 501(c)(3) nonprofit bringing free, hands-on
workshops in astronomy, physics, classics, computer science, and math to youth, with a focus on
underserved communities.

This repository holds the source of the LUMEN landing page, published at
<https://leeohwang.github.io/>.

## Structure

The site is a single self-contained static page. `index.html` carries its own styles and the
interactive constellation scene (rendered with [Three.js](https://threejs.org/), loaded from a CDN).
`fonts/` holds the self-hosted Carlito and Fraunces web fonts it uses.

There is no build step — open `index.html` in a browser, or serve the directory:

```sh
python3 -m http.server 8000
```

## Deployment

Pushing to `main` triggers `.github/workflows/deploy.yml`, which publishes the repository root to
GitHub Pages.

## Contact

outreach@lumen-initiative.org
