# Myanmar Earthquake Sequence Animation (2025)

Animated visualization of earthquake activity in central Myanmar following the devastating **M7.7 earthquake on 28 March 2025** near Mandalay. This project downloads seismicity data from the USGS Earthquake Catalog, overlays events on an OpenStreetMap basemap with tectonic lineaments, and produces a time-lapse MP4 video showing how the earthquake sequence evolved over time.

---

## Background

On 28 March 2025, a magnitude 7.7 earthquake struck central Myanmar, causing widespread destruction across the Mandalay and Sagaing regions. This project visualizes the mainshock and its subsequent aftershock sequence by animating daily seismicity on a map that includes:

- **OpenStreetMap** tile basemap for geographic context
- **Myanmar Tectonic Map (2011)** lineament overlay showing major fault systems
- **International borders** for regional reference
- **Depth-coded earthquake markers** with a 7-day fade-out window to highlight temporal clustering

The study area covers the central Myanmar seismic zone along the Sagaing Fault system, bounded by:

| Parameter      | Value  |
|----------------|--------|
| Min Longitude  | 95.0°E |
| Max Longitude  | 97.0°E |
| Min Latitude   | 19.0°N |
| Max Latitude   | 24.5°N |

## Data Sources

| Source | Description | Format |
|--------|-------------|--------|
| [USGS Earthquake Catalog (FDSNWS)](https://earthquake.usgs.gov/fdsnws/event/1/) | Authoritative global earthquake catalog maintained by the U.S. Geological Survey. Queried via the FDSN Event Web Service API. | GeoJSON |
| [Myanmar Tectonic Map 2011](https://raw.githubusercontent.com/drtinkooo/myanmar-earthquake/main/Myanmar_Tectonic_Map_2011.geojson) | Digitized tectonic lineaments of Myanmar showing major fault traces including the Sagaing Fault. | GeoJSON |
| [OpenStreetMap](https://www.openstreetmap.org/) | Open-source map tiles used as the basemap layer via Cartopy's tile service. | Raster tiles |

### USGS API Query Parameters

```
https://earthquake.usgs.gov/fdsnws/event/1/query
  ?format=geojson
  &starttime=2025-03-28
  &endtime=<current_date>
  &minlatitude=19.0
  &maxlatitude=24.5
  &minlongitude=95.0
  &maxlongitude=97.0
```

## Methodology

### Data Processing Pipeline

1. **Data Acquisition** -- Raw GeoJSON is downloaded from the USGS FDSNWS API with spatial and temporal filters applied at the query level.

2. **Format Conversion** -- The GeoJSON is parsed to extract four fields per event:
   - `longitude` (degrees East)
   - `latitude` (degrees North)
   - `depth` (km below surface)
   - `time` (UTC, formatted as `YYYY-MM-DDTHH:MM:SS`)

   These are written to a space-delimited `.xyzt` text file compatible with GMT and other geospatial tools.

3. **Map Composition** -- A Cartopy map is constructed with three layers:
   - **Layer 1 (bottom):** OpenStreetMap raster tiles at zoom level 8
   - **Layer 2 (middle):** Tectonic fault lineaments rendered in orange-red
   - **Layer 3 (top):** International borders (Natural Earth 10m resolution)

4. **Animation Rendering** -- For each calendar day in the dataset:
   - Events from the preceding 7 days are selected
   - Each event's opacity decays linearly with age (newest = fully opaque, 7+ days = invisible)
   - Events are color-coded by depth using the scheme below
   - The frame is saved as a PNG at 150 DPI

5. **Video Assembly** -- All frames are compiled into an MP4 video at 10 frames per second using `imageio` with FFmpeg.

### Depth Color Scheme

| Depth Range (km) | Color  |
|-------------------|--------|
| 0 -- 25           | Red    |
| 25 -- 50          | Orange |
| 50 -- 100         | Yellow |
| 100 -- 200        | Green  |
| 200 -- 400        | Blue   |
| 400 -- 800        | Purple |

## Repository Structure

```
myanmar-earthquake/
|-- animation_MDYquakes.ipynb   # Main Jupyter notebook (data processing + animation)
|-- query.json                  # Raw USGS earthquake data (GeoJSON)
|-- Quakes_filtered.xyzt        # Processed earthquake data (lon lat depth time)
|-- frames/                     # Individual animation frames (PNG)
|   |-- frame_000.png
|   |-- frame_001.png
|   |-- ...
|-- Myanmar_Animation_2025_Final.mp4  # Output video
|-- requirements.txt            # Python dependencies
|-- .gitignore
|-- README.md
```

## Requirements

- Python 3.10+
- An internet connection (for downloading USGS data, OSM tiles, and the tectonic map GeoJSON)

### Python Dependencies

```
pandas
geopandas
matplotlib
cartopy
imageio
imageio-ffmpeg
tqdm
```

## Quickstart

### 1. Clone the repository

```bash
git clone https://github.com/<your-username>/myanmar-earthquake.git
cd myanmar-earthquake
```

### 2. Install dependencies

```bash
pip install -r requirements.txt
```

### 3. Download earthquake data

Download the latest data from the USGS Earthquake Catalog by visiting the API URL or using `curl`:

```bash
curl -o query.json "https://earthquake.usgs.gov/fdsnws/event/1/query?format=geojson&starttime=2025-03-28&endtime=$(date +%Y-%m-%d)&minlatitude=19.0&maxlatitude=24.5&minlongitude=95.0&maxlongitude=97.0"
```

> **Note:** Replace the `endtime` parameter with your desired end date if you are not on a Unix-like system, or manually construct the URL.

### 4. Run the notebook

Open and execute all cells in `animation_MDYquakes.ipynb`:

```bash
jupyter notebook animation_MDYquakes.ipynb
```

Or run headlessly from the command line:

```bash
jupyter nbconvert --to notebook --execute animation_MDYquakes.ipynb --ExecutePreprocessor.timeout=600
```

### 5. View the output

The animation video will be saved as `Myanmar_Animation_2025_Final.mp4` in the project root.

## Customization

| Parameter | Location | Description |
|-----------|----------|-------------|
| Bounding box | USGS API URL / notebook | Change `minlatitude`, `maxlatitude`, `minlongitude`, `maxlongitude` |
| Date range | USGS API URL | Modify `starttime` and `endtime` |
| Fade window | Cell 2, animation loop | Change `pd.Timedelta(days=6)` to adjust the trailing visibility window |
| Frame rate | Cell 2, video assembly | Change `fps=10` in `imageio.get_writer()` |
| Map zoom | Cell 2, `add_image` | Adjust the zoom level integer (default: 8) |
| Output DPI | Cell 2, `plt.savefig` | Change `dpi=150` for higher/lower resolution frames |

## License

This project is provided for educational and research purposes. Earthquake data is sourced from the [USGS Earthquake Hazards Program](https://earthquake.usgs.gov/) and is in the public domain. OpenStreetMap data is licensed under the [Open Data Commons Open Database License (ODbL)](https://opendatacommons.org/licenses/odbl/).

## Acknowledgments

- **U.S. Geological Survey (USGS)** -- Earthquake catalog data via the FDSNWS Event Web Service
- **OpenStreetMap contributors** -- Basemap tile data
- **Myanmar Geosciences Society** -- Tectonic map digitization reference
