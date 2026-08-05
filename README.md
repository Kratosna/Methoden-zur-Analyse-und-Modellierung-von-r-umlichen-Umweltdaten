# Project Folder — File Guide

Data preparation, georeferencing, digitizing

## How to open the project

- Open `final.qgz` in QGIS. All layers load automatically.
- Keep every file in this one folder. The project uses **relative paths**, so it works on any machine as long as the folder is kept together.
- If a layer shows as "unavailable" when you open it, a file was moved out of this folder. Put it back — don't re-link by hand.
- Coordinate system throughout: **EPSG:31467** (DHDN / Gauss-Krüger zone 3, metres). Do not reproject.

## 1. Project file

| File | What it is | Notes |
|------|-----------|-------|
| `final.qgz` | The QGIS project file — the whole map, all layers, styling and both print layouts. | **Open this first.** Everything else loads through it. |

## 2. Original input layers (do not modify)

The seven shapefiles supplied with the assignment.

| File | What it is | Key field |
|------|-----------|-----------|
| `Frame.shp` | Study-area boundary (≈ 182 km² west of Hanover). | — |
| `Straßennetz.shp` | Road network. | `typ` (Kreisstraße, Bundestraße, Landesstraße/Staatsstraße, Bundesautobahn, …) |
| `Schienennetz.shp` | Railway lines. | `typ` |
| `Freileitungen.shp` | Overhead power lines. | — (no class field) |
| `Gewässer.shp` | Water bodies. | `typ` |
| `Landnutzung.shp` | Land use — holds settlement, forest, arable and grassland classes. | `typ` |
| `Ortslagen.shp` | Named places. | `Name` only (no settlement-type attribute) |

## 3. Produced files

| File | What it is | Notes |
|------|-----------|-------|
| `Luftbild_georef.tif` | The georeferenced aerial (EPSG:31467). | Use this one — it aligns with the vector data. |
| `Luftbild.points` | The georeferencing control points exported from QGIS. | Backs the accuracy figure in the report (5.1 px). Documentation only. |
| `Altwasser.gpkg` | The digitized oxbow arms — 7 polygons (field `typ = 'Altwasser'`). | 200 m protection buffer (biotope). Single-file GeoPackage. |
| `DLM.pdf` | Exported draft of Map 1 (the Digital Landscape Model). | Draft — cartography to be finalized. |

## 4. A note on the shapefiles (.shp)

Each shapefile is really a **set of files sharing one name** — e.g. `Straßennetz.shp`, `.shx`, `.dbf`, `.prj` (and sometimes `.sbn`, `.sbx`, `.xml`). They must stay together. If you ever move or copy a layer, move **all** of its same-named files, not just the `.shp`.
