# Plant map widget for Grist

A custom [Grist](https://www.getgrist.com/) widget that plots plants on your own aerial photo.

Upload a drone shot or a screenshot of an orthophoto, tell the widget where three or four
recognisable features are in the real world, and it works out the mapping between image
pixels and latitude/longitude. After that you can click to add a plant, drag to correct it,
and click a pin to jump to that row in Grist.

<img width="1301" height="895" alt="image" src="https://github.com/user-attachments/assets/8fe1acbb-83a3-463e-b09b-b70244aeaecd" />

---

## Features
 
**Mapping**
 
- Any image as the base map: drone photo, orthophoto screenshot, a scanned site plan.
- Georeferencing from control points. Three or more points fit an affine transform by
  least squares, which handles rotation, scale and shear, so the photo doesn't have to be
  north-up or perfectly square to the plot.
- Live accuracy readout: RMS fit error in metres and image scale in cm per pixel, plus a
  per-point error column so you can see which control point is the bad one.
- Pixel-only mode for when you don't need real coordinates, storing positions as image
  pixels instead.
**Working with plants**
 
- **Select** – click a pin to move Grist's cursor to that row. Any widget with its
  **SELECT BY** set to the map follows, so a card beside it can show species, notes,
  planting date or whatever else you keep.
- **Place** – click the photo to create a new row with coordinates filled in.
- **Move** – drag a pin to correct its position; the coordinates are written on release.
- **Unplaced strip** – plants with no coordinates, or coordinates far outside the current
  image, appear as hollow pins in a labelled strip beside the photo. Drag one onto the
  photo to place it; nothing is written until you do.
**Configuration lives in the document, not in the code**
 
- Upload the map image from inside the widget. Large files are downscaled in the browser
  first, so a 20 MP drone frame becomes about 1–2 MB.
- Calibration is saved to a `Map_images` table in your document, so it survives reloads
  and is shared by everyone using the document.
- The table and its first row are created on your first save or upload, not when the
  widget is added to a page.
- Any number of plots in one document: one map image and calibration per plants table,
  all stored as rows in the same `Map_images` table.
- Replacing the image invalidates the calibration and prompts you to redo it.
- Explicit **Save** and **Discard**, with an unsaved-changes indicator. A calibration
  worse than 1 m asks for confirmation before saving.
**Interface**
 
- Light and dark themes using Grist's own palette, following your operating system by
  default with a per-widget override.
- Layout adapts: the calibration panel is a sidebar when the widget is wide, and a bottom
  drawer when it's narrow.
- Errors are shown in the widget: missing columns, refused writes, rejected uploads and
  unreadable calibration data each explain what to fix.
---
 
## Requirements
 
- A Grist document (hosted or self-hosted) where you can grant a widget **Full document
  access**.
- For in-widget image upload: a reasonably recent Grist, and a reverse proxy that allows
  request bodies larger than 1 MB. See [Troubleshooting](#troubleshooting).
- No build step, no dependencies to install. The widget is one HTML file.
---
 
## Getting started
 
### 1. Prepare the plants table
 
Create (or pick) the table that will hold your plants. It needs three columns:
 
| Column ID | Type    | Purpose                        |
|-----------|---------|--------------------------------|
| `Lat`     | Numeric | Latitude, or image x in pixel mode |
| `Lng`     | Numeric | Longitude, or image y in pixel mode |
| `Name`    | Text    | Label shown on the pin         |
 
Add any other columns you like: species, planted date, supplier, notes, photos.
 
> **Check the column ID, not the label.** Grist derives the ID from the label when the
> column is created and then leaves it alone. A column labelled `Lat` that was originally
> called something else still has the old ID, and the widget looks up IDs. The ID is shown
> in the right-hand panel under the column name.
>
> If the names don't match, the widget will tell you which ones are missing on load.
 
### 2. Add the widget
 
Add a widget to your page, choose **Custom**, then pick **Custom URL**.
 
Now choose one of the two options below.
 
<details open>
<summary><strong>Option A — use the hosted build (quickest)</strong></summary>
Paste this URL:
 
```
https://mathias-g.github.io/grist-plant-map-widget/grist-plant-map-widget.html
```
 
That's it. The widget is a static page; your images and coordinates are stored in your own
Grist document, not on the host.
 
If a later version doesn't appear after an update, add a version marker to bust the cache:
`...grist-plant-map-widget.html?v=2`, then `?v=3` next time.
 
</details>
<details>
<summary><strong>Option B — host your own copy (fork it)</strong></summary>
Worth doing if you want to change the defaults, pin a version, or keep everything under
your own control.
 
1. Fork this repository, or download `grist-plant-map-widget.html` and `plot.jpg` into a
   new **public** repository. (GitHub Pages needs a paid plan for private repos.)
2. In the repository, go to **Settings → Pages**. Under *Build and deployment* set
   **Source** to *Deploy from a branch*, **Branch** to `main`, folder `/ (root)`, and save.
3. Wait a minute, then refresh. Your URL appears at the top of the page:
```
   https://<your-username>.github.io/<repo-name>/grist-plant-map-widget.html
```
 
   HTTPS is automatic on `github.io`, which is what Grist requires.
4. Open that URL in a normal browser tab first. You should see the placeholder image and
   the toolbar. It will say it isn't calibrated, which is expected outside Grist.
5. Paste the URL into Grist.
 
Rename the file to `index.html` if you'd rather have a shorter URL ending in a slash.
 
Keep `plot.jpg` alongside the HTML. It's the placeholder shown before you upload a real
image, and it's what the demo control points refer to.
 
</details>

### 3. Grant access
 
In the widget's right-hand panel, set **Access level** to **Full document access**.
 
This is not optional. The widget reads and writes rows, creates the `Map_images` table, and
requests the short-lived token used to upload and download the map image. With a lower
access level, requests hang with no error, which looks like nothing happening at all.
 
### 4. Upload your map image
 
Switch to the **Calibrate** tab, choose a file, and press **Upload image**.
 
This creates the `Map_images` table and its first row, and stores the photo as an
attachment in your document.
 
Images are downscaled to 4000 px on the long edge before upload, which is enough to
resolve individual plants on a garden-sized plot. A photo taken as close to straight down
as possible will calibrate better; see [How it works](#control-points-and-the-transform)
for why.
 
### 5. Calibrate
 
Pick three or four features you can identify both on your photo and on an aerial map.
Corners of buildings, fence posts, gateposts and path junctions are good. **Tree canopies
are not** — they lean away from the centre of the frame and will skew the fit.
 
For each one: click it on the photo, then paste its real coordinates as
`latitude, longitude`.
 
Where to get the coordinates: right-clicking a spot in Google Maps puts them at the top of
the menu, and OpenStreetMap, QGIS or any tool that reports WGS84 will do. Many national
mapping agencies publish free high-resolution aerial imagery, which is usually sharper than
a screenshot from a consumer map viewer.
 
Watch the fit error in the status bar. Under 0.5 m is good. If one point shows a much
larger error than the others, it's usually misidentified rather than mismeasured.
 
Spread the points towards the corners of the plot rather than clustering them in the
middle — spread is what constrains rotation and scale.
 
Press **Save calibration** when you're happy.
 
> Existing plants keep whatever coordinates they were given. Recalibrating does not move
> them. Get the calibration right before you start placing plants.

<img width="1301" height="895" alt="image" src="https://github.com/user-attachments/assets/809fba1b-f16a-49f3-abab-ea1adc4df47f" />

 
### 6. Place plants
 
Switch to **Place** and click the photo to add a plant. Switch to **Move** and drag a pin to
correct one. Press `f` at any time to re-fit the view to the image.
 
### 7. Link the map to another widget
 
Clicking a pin in **Select** mode moves the map's cursor, but nothing else on the page
reacts to it until you say so. This is a one-time setting in Grist; the widget can't do it
for you, and there's no error when it's missing — clicking a pin simply appears to do
nothing.
 
In almost all cases the widget you want is your **plants table**: it should follow the map,
so that clicking a pin selects that plant's row.
 
Click the plants table widget. Open the right panel using the green bar at the top right of
the page, go to the **Data** tab, and set its **SELECT BY** to the map.
 
Two widgets can't select each other, so a pair gives you one direction at a time. The first
row below is the usual setup:
 
| Set this                            | And this happens                                   |
|-------------------------------------|----------------------------------------------------|
| Plants table **selects by** the map | Clicking a pin selects that plant's row in the table |
| Map **selects by** the plants table | Clicking a row in the table highlights its pin     |

<img width="1331" height="1022" alt="image" src="https://github.com/user-attachments/assets/41daeea5-6c20-433f-91e1-ac9df35b1154" />

 
---
 
## How it works
 
### Control points and the transform
 
Each control point pairs a pixel position in the image with a real-world coordinate. With
three or more, the widget solves for an affine transform:
 
```
lat = a·x + b·y + c
lng = d·x + e·y + f
```
 
Six unknowns, so three points determine it exactly and more are fitted by least squares.
Affine covers translation, rotation, scale and shear, which is what you need for a
near-nadir aerial photo over flat ground. It does not correct lens distortion or terrain
relief, so a photo taken close to straight down over flat ground calibrates best.
 
The reported error is the RMS distance in metres between where each control point actually
is and where the fitted transform puts it.
 
### Stored data
 
Your plants table holds `Lat`, `Lng` and `Name`. Everything else the widget needs lives in
a `Map_images` table it creates:
 
| Column               | Type        | Holds                                              |
|----------------------|-------------|----------------------------------------------------|
| `Name`               | Text        | Label for this map, shown when picking between maps |
| `Image`              | Attachments | The photo                                           |
| `Control_points`     | Text        | JSON list of `{px, py, lat, lng}`                   |
| `Image_width`        | Numeric     | Dimensions the calibration was made against         |
| `Image_height`       | Numeric     |                                                     |
| `Calibrated_against` | Numeric     | Attachment id the control points refer to           |
| `Coordinate_mode`    | Text        | `geo` or `pixel`                                    |
| `Plants_table`       | Text        | Table id this map belongs to                        |
 
Pairing the image with its calibration in one row is deliberate: they can't drift apart. If
the image is replaced, `Calibrated_against` no longer matches and the widget discards the
old points and says so, rather than confidently applying the wrong transform.
 
You can edit this table by hand. Bad JSON in `Control_points` produces a visible error
rather than a blank map.
 
### Multiple plots in one document
 
There's no limit on how many maps a document can hold. To add another plot:
 
1. Create a second plants table with its own `Lat`, `Lng` and `Name` columns.
2. Add a custom widget bound to that table, pointing at the same URL.
3. Upload an image and calibrate it.
A second row appears in `Map_images` on that first upload, holding the new photo, its
calibration, and the id of the plants table it belongs to. You never add rows to
`Map_images` yourself, and there's only ever one such table no matter how many maps you
have.
 
Each widget looks up its own row by the table it's bound to, so the maps are fully
independent: different photos, different calibrations, and one can be in pixel mode while
another uses real coordinates.
 
Two consequences worth knowing. A plants table can only have one map, so pointing two
widgets at the same table gives them both the same image. And deleting a row from
`Map_images` makes its widget fall back to the placeholder image, as though it had just
been added.
 
### Unplaced plants
 
A plant appears in the unplaced strip when it has no numeric coordinates, or when its
coordinates fall more than one image width or height beyond the edge of the photo. Anything
closer than that is drawn where it actually is, clipped into the margin, on the assumption
that it's genuinely just outside the frame rather than wrong.
 
Unplaced pins are draggable in every mode. Dropping one back into the strip writes nothing.
 
---
 
## Configuration
 
Everything you'd normally need is in the document. These are in the HTML file, near the top
of the `<script>` block, for anyone hosting their own copy.
 
| Setting             | Default                        | Effect |
|---------------------|--------------------------------|--------|
| `CONFIG.cols`       | `{lat:'Lat', lng:'Lng', label:'Name'}` | Column IDs on the plants table |
| `CONFIG.placeholderUrl` | `'plot.jpg'`               | Demo image shown before anything is uploaded |
| `CONFIG.placeholderPoints` | three demo points       | Demo calibration for the placeholder image |
| `CONFIG.newRecordDefaults` | `{}`                    | Extra fields set on every plant created by clicking |
| `CONFIG.maxRmsMetres` | `1.0`                        | Fit error above which saving asks for confirmation |
| `MAX_EDGE_PX`       | `4000`                         | Longest edge after downscaling on upload |
| `JPEG_QUALITY`      | `0.85`                         | Re-encode quality on upload |
| `MAP_TABLE`         | `'Map_images'`                 | Name of the table holding maps and calibration |
| `TRAY_GAP`, `TRAY_W` | `26`, `96`                    | Position and width of the unplaced strip |
 
---
 
## Troubleshooting
 
**Nothing happens when I click Place.**
Access level is not set to Full. Requests queue behind the permission gate rather than
failing, so there's no error to see. Set it in the widget's right-hand panel.
 
**Upload fails with a CORS or network error.**
Usually a size limit rather than a permissions problem. A reverse proxy in front of Grist
rejects the request with `413 Payload Too Large`, and that rejection carries no CORS
headers, so the browser reports a missing `Access-Control-Allow-Origin` instead of the real
status. On nginx the default is 1 MB:
 
```nginx
client_max_body_size 64M;
```
 
then `nginx -t && systemctl reload nginx`. On Caddy it's `request_body max_size`, on
Traefik the buffering middleware's `maxRequestBodyBytes`.
 
If you can't change the server, downscaling already keeps most images under the limit; drop
`MAX_EDGE_PX` further if needed.
 
**Changes to my forked copy don't show up.**
GitHub Pages caches HTML for about ten minutes and Grist's iframe caches on top of that.
Hard reload with `Ctrl+Shift+R` first. If that doesn't do it, bump the version marker on
the URL in Grist: `?v=2`, then `?v=3`.
 
**Clicking a pin doesn't select the row.**
Nothing is listening. A widget only follows the map if its **SELECT BY** is set to the map,
in the Data section of the right-hand panel.
 
**The widget says a column is missing but I can see it.**
The label and the ID differ. Check the ID in the right-hand panel under the column name.
 
**Everything shows as unplaced after I recalibrated.**
Expected if the new calibration puts the old coordinates outside the photo. Drag them back
onto the map, or clear their coordinates and place them again.
 
## Limitations
 
- Affine only. No lens distortion correction, no terrain correction. Fine for a small flat
  plot from a near-nadir photo; not a substitute for photogrammetry over hilly ground or
  multiple stitched frames.
- Recalibrating does not move existing plants. Their stored coordinates are treated as the
  truth, so a bad initial calibration is not something you can undo later.
- Grist's own theme isn't readable from inside a widget iframe, so light/dark follows your
  operating system unless you override it in the widget.
- The `Map_images` table isn't the widget's selected table, so there's no change
  subscription for it. Editing calibration directly in the table shows up on next reload.
- Attachment images are re-fetched on every widget load, since the download URL carries a
  short-lived token and can't be cached.
- Uploading attachments from a widget is known to have problems on documents with access
  rules applied, when the user isn't the document owner.
---
 
## Built with
 
[Leaflet](https://leafletjs.com/) in `L.CRS.Simple` mode, and the
[Grist plugin API](https://support.getgrist.com/code/). One HTML file, no build step.
 
