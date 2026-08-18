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
| `Buffer.gpkg` | Contains the individual buffers for roads, railway, power lines, water, forest, oxbow arms and settlements, together with intermediate analysis layers. | Used to check and trace the creation of the exclusion areas. |
| `Total_Exclusion_Area.gpkg` | The combined exclusion layer after Merge, Fix Geometries, Dissolve and Clip to the `Frame` boundary. | Main input for the final suitability analysis. |
| `Person3.gpkg` | GeoPackage containing the layers created during Part 3. | Includes the `Ackerland` and `Gruenland` selection, Clip, Dissolve, Difference, Multipart to Singleparts and the final result. |
| `Suitable_Windfarm_Areas` | Final result layer stored inside `Person3.gpkg`. | Contains 4 suitable areas of at least 18 ha, with a total area of approximately 90.93 ha. |
| `Map_2_Suitable_Areas.pdf` | Exported final map showing the suitable areas for wind turbine development. | Map 2 of the assignment. |


## 4. A note on the shapefiles (.shp)

Each shapefile is really a **set of files sharing one name** — e.g. `Straßennetz.shp`, `.shx`, `.dbf`, `.prj` (and sometimes `.sbn`, `.sbx`, `.xml`). They must stay together. If you ever move or copy a layer, move **all** of its same-named files, not just the `.shp`.

---

## Person 2 – Buffer Analysis and Exclusion Areas


**Study area:** ~182 km² west of Hanover | **CRS:** EPSG:31467 (DHDN / Gauss-Krüger zone 3)

### 1. What this stage of the project was about

Person 1 handed over the QGIS project with the seven original shapefiles, the georeferenced aerial photo, and the digitized oxbow arms of the Leine river. Our job was to turn all of that into one thing: a single layer showing every area where a wind turbine legally cannot be built, based on the minimum distances given in the assignment. This means building eleven separate safety buffers (roads, railway, power lines, water, forest, oxbow arms, and settlements) and then merging them into one clean exclusion area that Person 3 can subtract from the usable farmland.

### 2. The eleven buffers

Each buffer distance and the exact selection used is listed below. `Straßennetz.shp` holds several road categories in one field (`typ`); we only buffered the three categories the assignment actually asks for and left the rest (Gemeindestraße, Weg, Straße sonstige) untouched, since applying a distance to them isn't part of the criteria.

| Feature | Buffer | Selection used |
| --- | --- | --- |
| District roads (Kreisstraße) | 15 m | `typ='Kreisstraße'` |
| Federal / state roads | 20 m | `typ IN('Bundestraße','Landesstraße, Staatsstraße')` |
| Motorways (Bundesautobahn) | 40 m | `typ='Bundesautobahn'` |
| Railway lines | 40 m | all features |
| Overhead power lines | 195 m | all features (3 × 65 m rotor diameter) |
| Water bodies | 25 m | all features |
| Oxbow arms (Altwasser) | 200 m | all features (protected biotope) |
| Forest | 200 m | `typ='Wald; Forst'` |
| Settlement / industrial / commercial / special-function areas | 650 m | Siedlungsflaeche intersecting Ortslagen, or `typ IN('Indus.-Gew.-Fl.','Fl. fkt. Praeg.')` |
| Individual houses / settlement splinters | 450 m | Siedlungsflaeche NOT intersecting Ortslagen |

All buffers were run with the same settings for consistency: 16 segments per quarter-circle, round caps and joins, and 'dissolve result' switched on so overlapping parts of the same class merge automatically.

### 3. The settlement problem, and how we solved it

This was the one part that wasn't straightforward. The assignment separates 'settlement and residential areas' (650 m) from 'individual houses and settlement splinters' (450 m), and defines the splinters as settlement areas 'not assigned to a local location'. The trouble is that `Landnutzung.shp` has no field that marks this difference — every settlement patch just carries the same value, `Siedlungsflaeche`.

We solved it with a spatial comparison instead of an attribute filter: any `Siedlungsflaeche` polygon that touches an `Ortslagen` polygon (the layer of named villages) counts as a settlement assigned to a local location, and gets the 650 m buffer along with industrial, commercial and special-function areas. Anything that doesn't touch Ortslagen — isolated farms, hamlets, scattered houses — gets the 450 m buffer instead. Out of 1,771 `Siedlungsflaeche` polygons, 1,640 (about 3,231 ha) are attached to a named village, and 131 (about 118 ha) sit on their own. This matches the assignment's own wording almost exactly, so we're confident it's the right interpretation, not just a convenient one.

### 4. Putting the eleven buffers together

Once all eleven buffers existed, we combined them into one exclusion layer in four steps:

- **Merge** – all 11 buffer layers combined into a single layer.
- **Fix geometries** – cleaned up before dissolving, not after. Merging that many overlapping polygons can leave small invalid slivers, and dissolving on top of those can create gaps or broken multiparts that a later 'fix' won't undo. Repairing first keeps the topology clean going into the dissolve.
- **Dissolve** – no grouping field, so every buffer that touches another one melts into a single shape with no leftover internal lines.
- **Clip** – cut to the Frame boundary, so buffers that stuck out past the edge of the study area don't count.

The result, `Total_Exclusion_Area`, is exactly one polygon feature.

### 5. How much area is actually excluded

This is where it's worth being precise rather than using the assignment's rounded '~200 km²'. The Frame boundary measures **181.78 km² (18,177.8 ha)** — that's the real, measured figure, not an estimate. The exclusion area comes out to **17,519.48 ha**.

That means about **96.4%** of the study area is off-limits once every buffer is applied, leaving roughly **658 ha (6.6 km²)** as scattered, unconstrained patches. That's a small remainder, but it's not a mistake — the 650 m settlement buffers are large and this is a fairly densely settled part of Lower Saxony, so they overlap heavily and swallow most of the map. You can see this visually too: the exclusion layer looks almost solid, with only small isolated white 'holes' left over, and those holes are exactly the candidate zones Person 3 needs to filter further.

### 6. A few things that went wrong along the way (and how we caught them)

Worth mentioning briefly, since it explains some of the choices above:

- Early attempts at extracting `Wald;Forst` using `IN('Wald','Forst')` matched nothing, because the field actually stores the combined string `'Wald; Forst'` — the semicolon is part of the value, not a separator.
- We tested an extra safety filter (`$area < 10,000,000`) to screen out potential oversized outlier polygons before buffering forest and settlement classes. It turned out to do nothing: the largest forest polygon is 261.8 ha and the largest `Siedlungsflaeche` polygon is 22.0 ha, both far below that threshold. We removed it from the final expressions rather than keep a condition that has no real effect.
- A couple of intermediate results were briefly left as temporary, in-memory layers before being properly saved to file — fixed by exporting each one explicitly before moving to the next step.

### 7. What we're handing to Person 3

- `Total_Exclusion_Area.gpkg` — the final exclusion layer (main deliverable).
- `Buffer.gpkg` — all eleven individual buffers plus the merged and dissolved intermediate layers, kept for traceability.
- `Altwasser.gpkg` and `Luftbild_georef.tif` — carried over from Person 1.

**Next step for Person 3:** take the land-use classes Ackerland and Grünland, subtract `Total_Exclusion_Area` using a Difference operation, convert the result from multipart to singlepart, calculate area in hectares, and keep only the patches of at least 18 ha — enough room for three wind turbines.
