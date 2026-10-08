# python-osm-guest-lecture

Author: Zhanchao Yang <br>
Oct 8, 2026

Guest lecture materials for **MUSA-5500 · Geospatial Data Science in Python, Week 8**: OpenStreetMap, street networks, and network accessibility with Pandana.

The class has two parts:

1. **OpenStreetMap and OSMnx.** What OSM is (nodes, ways, relations, tags), then pulling a city boundary, nearby features, and a walking network with OSMnx, ending with a first shortest-path route in University City.
2. **From routing to accessibility with Pandana.** Why "one route" doesn't scale to "every street corner", what Pandana is, and a four-step walkthrough: get amenities → build a network → attach POIs → query and map distance to the nth nearest amenity across Center City. It closes with transit accessibility using [r5r](https://ipeagit.github.io/r5r/).

## Setup

Tested on Python 3.12.

```bash
python -m venv .venv
```

```bash
source .venv/bin/activate
```

```bash
pip install -r requirements-pandana.txt osmnet altair
```

`osmnet` provides `pandana.loaders.osm`, which the tutorial uses, and `altair` is used for one chart. If Pandana fails to build with pip, conda-forge is the most reliable route:

```bash
conda install -c conda-forge pandana osmnet osmnx geopandas altair
```

Start Jupyter from the **repository root** so the `data/...` and `cache/` paths resolve. The `backup/` notebooks read those paths, so either launch from the root or change them to `../data/...`.

## Running the lecture

1. **First half:** `backup/01_OSM_Step_by_Step.ipynb`, §1–§6, following slides 11–22.
2. **During the break:** in `week-8A-street-network.ipynb`, run the imports cell and the cells under *"Streets within a specific polygon"* up to `center_city_outline = ...`. Part 2 depends on that variable.
3. **Second half:** `week-8A-street-network.ipynb` → *Part 2: Pandana*, Steps 1–4, following slides 30–35.


## Data and license

Map data © [OpenStreetMap contributors](https://www.openstreetmap.org/copyright), available under the Open Database License (ODbL 1.0). Snapshot details are in [`data/README.md`](data/README.md).

Code and teaching materials are released under the [Apache License 2.0](LICENSE).

## References

- OSMnx: https://osmnx.readthedocs.io/
- Pandana: https://udst.github.io/pandana/
- r5r: https://ipeagit.github.io/r5r/ · r5py: https://r5py.readthedocs.io/
- Course tutorial: https://xiaojianggis.github.io/MUSA-5500-Geospatial-Data-Science-Python/labs/week-8-network-analysis/week-8A-street-network.html
