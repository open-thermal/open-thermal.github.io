# OpenThermal

**An open ecosystem for outdoor thermal research: projects, tools, code and papers on human thermal exposure and mean radiant temperature.**

Website: <https://open-thermal.github.io> · Source: <https://github.com/open-thermal/open-thermal.github.io>

OpenThermal is meant to be shared by many labs. Anyone can add a project, tool, dataset or resource through a pull request, and use issues for
questions and suggestions; see [CONTRIBUTING.md](CONTRIBUTING.md). The first projects here come from the SHaDE Lab at Arizona State University
([shadelab.asu.edu](https://shadelab.asu.edu)); more are welcome.

Mean radiant temperature (MRT) is the radiation a person absorbs from sun, sky and surrounding surfaces. It often drives outdoor
heat stress more than air temperature does. The projects below estimate it from different kinds of data.

---|---|---|---|---|
| **Cool Routes** | Walking routes on the ASU campus that trade a little distance for much less radiant heat, from hourly SOLWEIG MRT forecasts | [coolroutes.sensableheatscapes.com](https://coolroutes.sensableheatscapes.com/) | Buo *et al.*, *Building and Environment* (in press), [doi:10.1016/j.buildenv.2026.114622](https://doi.org/10.1016/j.buildenv.2026.114622) | [`codebases/cool-routes`](codebases/cool-routes) |
| **VisionMRT** | Physics-based MRT, UTCI and PET from a 360° image: segmentation, depth, shadow casting, surface energy balance | [shadelab.asu.edu/VisionMRT](https://shadelab.asu.edu/VisionMRT) | Middel *et al.*, *Sustainable Cities and Society* 151:108016 (2026), [doi:10.1016/j.scs.2026.108016](https://doi.org/10.1016/j.scs.2026.108016) | [`codebases/visionmrt`](codebases/visionmrt) |
| **WebMRT** | Machine-learning prediction of summertime MRT from meteorology and the surroundings (LightGBM, trained on MaRTy field data) | [shadelab.asu.edu/WebMRT](https://shadelab.asu.edu/WebMRT) | AlKhaled *et al.*, *Sustainable Cities and Society* 115:105861 (2024), [doi:10.1016/j.scs.2024.105861](https://doi.org/10.1016/j.scs.2024.105861) | [`codebases/webmrt`](codebases/webmrt) |
| **PanoMRT** | MRT, UTCI and PET from a thermal 360° panorama and weather data: surface temperatures measured per pixel, six-direction radiation budget (no web tool) | none | Middel *et al.*, *Science of The Total Environment* 859:160301 (2023), [doi:10.1016/j.scitotenv.2022.160301](https://doi.org/10.1016/j.scitotenv.2022.160301) | [`codebases/panomrt`](codebases/panomrt) |

The website also lists the related image-based modeling paper, Shaeri, AlKhaled & Middel (2025), *A Multimodal Physics-Informed Neural
Network Approach for Mean Radiant Temperature Modeling*, ICCV Workshops, [IEEE Xplore](https://ieeexplore.ieee.org/document/11375311).

## Tools

The two instruments behind the SHaDE Lab's field data, each with an interactive 3D model you can rotate and take apart:

| Tool | What it is | Paper | Code |
|---|---|---|---|
| **MaRTy** | Mobile human-biometeorological station: six-directional radiation (twelve sensors), air temperature, humidity, wind and GPS on a cart | Middel & Krayenhoff, *Science of The Total Environment* 687:137–151 (2019), [doi:10.1016/j.scitotenv.2019.06.085](https://doi.org/10.1016/j.scitotenv.2019.06.085) | [`codebases/MaRTy`](codebases/MaRTy) (CERN-OHL-P-2.0) |
| **MaRTiny** | Low-cost IoT device with a globe thermometer and embedded computer vision that counts people in sun and in shade | Kulkarni *et al.*, *Frontiers in Environmental Science* 10:866240 (2022), [doi:10.3389/fenvs.2022.866240](https://doi.org/10.3389/fenvs.2022.866240) | [`codebases/MaRTiny`](codebases/MaRTiny) (CERN-OHL-P-2.0) |

The 3D models are drawn in code (three.js) from photographs of the devices and the sensor tables in the papers. They are illustrative:
proportions are approximate and some mounting details are simplified.

## What is in this repository

```
index.html            the website (static, no build step)
assets/               styles, scripts and the single place to edit links (PORTAL block in assets/site.js)
demos/                interactive demos that run in the browser
  coolroutes/           real ASU campus data and a JavaScript router with the same cost function as the Python code
  visionmrt/            precomputed all-day outputs of the released code on two sample sites
  webmrt/               the exported LightGBM sample model running in the browser
  _3d/ marty/ martiny/  the shared three.js viewer and the MaRTy and MaRTiny 3D models
codebases/            curated, tested example code for each project (each folder has its own README)
  cool-routes/  visionmrt/  webmrt/  panomrt/  MaRTy/  MaRTiny/   (MaRTy: logger program and field guide; MaRTiny: Arduino and ESP32-CAM firmware, image server, MongoDB tools and model notes)
patches/              proposed change to the Cool Routes frontend (URL parameters for start and destination)
LICENSE  NOTICE       Apache-2.0 and third-party notices
```

### What "example code" means

Each codebase is a curated release of the **published method** on **small sample data**, with tests. It is not the full web application
and it does not reproduce each paper's complete validation. Each README lists what is included, what is deliberately left out, the
provenance of the sample data and the known limitations.

## Run the website locally

The demos load their data with `fetch`, so serve the folder over HTTP (opening `index.html` from disk will not work):

```bash
python -m http.server 8080
# open http://localhost:8080
```

The page opens in dark mode by default; the ◐ button switches themes and the browser remembers the choice.

The 3D viewers load three.js from a pinned jsDelivr URL (see the import map in `index.html`), so they need an internet connection and a
browser with WebGL.

## Run the code and tests

Each codebase is self-contained (Python 3.10+).

```bash
cd codebases/cool-routes && pip install -r requirements.txt && python -m pytest     # 21 tests, about 15 s
cd codebases/webmrt      && pip install -r requirements.txt && python -m pytest     # 17 tests, about 5 s
cd codebases/visionmrt   && pip install -r requirements.txt && python -m pytest     # 41 tests, about 4 min
cd codebases/panomrt     && pip install -r requirements.txt && python -m pytest     # 51 tests, about 3 min (-m "not slow": 15 s)
```

The in-browser demos are checked against the Python results (needs Node 18+):

```bash
node demos/coolroutes/parity.test.mjs    # JS router vs. Python golden routes
node demos/webmrt/parity.test.mjs        # JS tree evaluator and solar position vs. Python / pvlib
```

Each codebase has a `run_demo.py` that regenerates its demo assets under `demos/`.

## Editing the site

All project links and release status live in one block at the top of [`assets/site.js`](assets/site.js) (`window.PORTAL`):
the live-tool URL, paper URL, code URL and whether the live tool can be embedded.

* **Embedding the live tools.** Cool Routes is embedded in the page. `shadelab.asu.edu` currently sends `X-Frame-Options: DENY`, so
  WebMRT and VisionMRT show a launch card. If the host allows this site (`Content-Security-Policy: frame-ancestors`), set their
  `embed` flag to `true`.
* **Start and destination in the embedded Cool Routes.** [`patches/`](patches) holds a small proposed change to the Cool Routes frontend
  that reads `?start=COOR&end=CRTVC`. It is a proposal only and has not been applied to any lab repository.
* **Hosting.** The site is plain static files (`.nojekyll` is included) and is served with GitHub Pages at <https://open-thermal.github.io>.

## Data and third-party notices

Sample data are included for demonstration and documented in each codebase README. In short: the Cool Routes walking network derives
from OpenStreetMap (© OpenStreetMap contributors, ODbL); the VisionMRT sample weather is from Open-Meteo (CC BY 4.0) and its sites ship
as segmentation and depth maps only (no source photographs); the WebMRT sample is a seeded subset of the lab's MaRTy measurements.
The SOLWEIG model code used to produce the Cool Routes MRT rasters derives from UMEP (GPL-3.0) and is **not** included. See
[NOTICE](NOTICE).

## Citing

If you use a tool or its code, please cite the paper for that tool (BibTeX is on the website under *Publications*, and each codebase has
a `CITATION.cff`).

## License

Code is released under the [Apache License 2.0](LICENSE), except [`codebases/MaRTy`](codebases/MaRTy) and [`codebases/MaRTiny`](codebases/MaRTiny), which are released under the
[CERN Open Hardware Licence v2, Permissive (CERN-OHL-P-2.0)](codebases/MaRTy/LICENSE) because they describe hardware instruments.
Each contributed project keeps the license stated in its folder; third-party material keeps its own license (see [NOTICE](NOTICE)).

## Contact

[The SHaDE Lab](https://shadelab.asu.edu/contact/), Arizona State University.
