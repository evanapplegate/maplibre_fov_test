# MapLibre FOV Test

Interactive FOV (field of view) slider on a MapLibre GL JS map, displaying a habitat restoration area shapefile along the Salt River in Phoenix, AZ.

**Live demo:** https://evanapplegate.github.io/maplibre_fov_test/

## What this does

- Loads a shapefile as GeoJSON on a MapLibre GL JS web map
- Provides a slider to adjust the camera FOV in real time (1°–60°)

## How it was built

### 1. Shapefile → GeoJSON

The source shapefile uses **NAD83 HARN State Plane Arizona Central (feet)**. It was reprojected to WGS84 and converted to GeoJSON using geopandas:

```python
import geopandas as gpd

gdf = gpd.read_file("LimitsofHabitatRestorationArea.shp")
gdf = gdf.to_crs(epsg=4326)
gdf.to_file("data.geojson", driver="GeoJSON")
```

### 2. MapLibre GL JS map

A single `index.html` loads the GeoJSON via `fetch()`, adds it as a fill+line layer, and auto-fits the view to the data bounds. Basemap is CARTO Positron.

### 3. FOV control

MapLibre has **no public API for FOV**. The only way to change it is through the internal transform object:

```js
map.transform.fov = 45;   // degrees — setter clamps to [0.01, 60]
map.triggerRepaint();      // must manually trigger redraw
```

This setter lives in `src/geo/transform.ts` in the [maplibre-gl-js repo](https://github.com/maplibre/maplibre-gl-js). The default FOV is ~36.87° (`2 * atan(3/4)`).

**How to find this yourself:**
1. Open devtools on any MapLibre map
2. Type `map.transform` and expand it — all camera internals (`_fov`, `_pitch`, etc.) are there
3. Test: `map.transform.fov = 10; map.triggerRepaint();`
4. Or search the source repo for `set fov`

## Files

| File | Description |
|---|---|
| `index.html` | Map + FOV slider, fully self-contained |
| `data.geojson` | Habitat restoration boundary (WGS84) |
| `LimitsofHabitatRestorationArea.*` | Original shapefile bundle (NAD83 HARN) |

## Run locally

```bash
python3 -m http.server 8767
# open http://localhost:8767
```
