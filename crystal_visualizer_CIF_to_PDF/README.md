# Crystal Structure Visualizer – Colab Edition

An interactive, notebook-based tool for uploading, editing, and visualizing
crystallographic structures, then exporting a publication-ready multi-panel
figure as PDF or SVG. Runs entirely inside Google Colab – no local install,
no desktop app.

This README walks through every part of the tool using a real example:
the 2H-MoS₂ figure below, built end-to-end with the settings shown
alongside it.

*Four independent views of 2H-MoS₂ (a-axis, b-axis, c-axis, and an
isometric view), each with its own legend/cell-dimension/compass
visibility, exported as a single A3 PDF.*

---

## Requirements

The notebook installs everything it needs in its first cell:

```
pip install ase spglib ipywidgets
```

- **ase** – reads/writes crystal structure files (CIF, POSCAR, XYZ, XSF, PDB, …)
- **spglib** – optional; enables automatic symmetry detection. The tool
  still works without it (manual symmetry operations remain available).
- **ipywidgets** – the interface itself.

## Quick start

1. Open `Crystal_Visualizer_Colab.ipynb` in Google Colab.
2. Run **Step 1** (installs dependencies).
3. Run **Step 2** (builds and displays the app). This can take a few
   seconds the first time.
4. Use the **File / Cell** tab's **Upload** button to load a structure.

Every setting updates the live preview immediately; nothing needs a
"refresh" button unless stated otherwise.

---

## Walkthrough: building the MoS₂ example

This reproduces the exported figure above, step by step.

### 1 – Load the structure

**File / Cell** tab → **Upload** → select `MoS2.cif`. The preview appears
immediately.
Still in **File / Cell** go to **Append cells around (display only; fractional)**
and set values of **±a** and **±b** to **1**, it will append cells in-plane direction.

### 2 – Set the bond cutoff so the van der Waals gap stays open

**Display** tab → **Max bond (Å)**: Base setting **Max
bond 2.5** safely captures every real bond while leaving the
interlayer gap open. (For a mixed-element structure where a single global
cutoff isn't enough, add a specific **Elem 1 / Elem 2 / Max Å** pair
instead – pair cutoffs work even if the global **Max bond** is left at 0.)

### 3 – Set colors, legend, and orientation aids (still Display tab)

- Element colors default to standard Jmol colors; double-click a swatch
  to change one.
- **Show legend** adds the color key.
- **Show orientation compass** adds a small a/b/c (or x/y/z) direction
  indicator – very useful once a panel is rotated away from a standard
  axis view.
- **Dim. style** → *Corner box* prints the cell lengths (a, b, c) as one
  clean block in a panel corner (option **Attached to cell edges** in most cases it's not working that well).

None of these need to be turned on globally before exporting – each
export panel controls its own visibility (see step 5).

### 4 – Go to the Export tab: page and text setup

**Export** tab → **PDF page**:

| Field | Value used | What it does |
|---|---|---|
| Page | A3 | Standard paper size presets, or *Custom* for any W × H |
| Orient | landscape | Swaps W/H for the chosen page size |
| Margin | 0.5 | Outer margin, inches |

**Spacing** → **H-gap** / **V-gap** control the gap between panels (as a
fraction of one panel's size; small negative values are allowed to pack
panels tighter).

**Text size**:

| Field | What it controls |
|---|---|
| Title size | The one overall figure title (typed into **Title**, near the bottom) |
| Caption size | Each panel's own caption, above that panel |
| Tick text scale | Multiplies the automatic axis/tick-label sizing |

### 5 – Panel layout: four independent views

**Panel layout** → **Rows** = 2, **Cols** = 2 creates four panel rows
(P1–P4), each with its own angle, zoom, panel fills, caption, and
legend/cell-dim/compass visibility and position. **Nothing here is
shared** – editing one panel's fields never affects another.

For each panel, set the angle directly, or dial it into the **Display**
tab and click that panel's **Use current** button to copy it in as a
starting point (still fully editable afterward, for that panel alone).
The four preset angle buttons on the Display tab (**a-axis**, **b-axis**,
**c-axis**, **iso**) make this fast:

| Panel | Caption | elev | azim | Changes and features turned on for *this panel only* |
|---|---|---|---|---|
| P1 | a-axis | 0 | 0 | compass |
| P2 | b-axis | 0 | 90 | legend, compass |
| P3 | c-axis | 90 | −90 | cell dimensions, compass |
| P4 | iso | 25 | −60 | fill set to 0.9, compass |

Each row's **Legend / cell-dim / compass (show + position)** section
(collapsed by default – click to expand) is where the checkboxes above
live, alongside a corner dropdown and fine X/Y offset for each, in case
the default corner lands on top of the structure in a particular panel.

### 6 – Title, filename, and format

- **Title**: `MoS2` – the one figure-wide heading, shown centered above
  every panel.
- **Filename**: `structure of MoS2` – no extension needed, it's added
  automatically based on the format chosen. (Path separators and stray
  characters are stripped automatically, so this is safe to type freely.)
- **Format**: `PDF` (or `SVG`, for a scalable vector file instead).

### 7 – Preview, then export

- **Preview layout** renders the exact export figure inline first, so
  nothing is a surprise (I hope so).
- **Export & download** writes the file and downloads it. The status
  line confirms the saved path, e.g. `Exported /content/structure of
  MoS2.pdf`.

---

## Full feature reference

### File / Cell tab
- **Upload** any ASE-readable format: CIF, POSCAR/CONTCAR, XYZ, XSF,
  CUBE, PDB.
- **Download current (.cif)** / **Reset to loaded file**.
- **Unit cell**: edit a, b, c (Å) and α, β, γ (°) directly, with a
  **Keep fractional coords** option so atoms move with the cell
  (or stay at fixed Cartesian positions if unchecked).
- **Supercell expansion**: nx × ny × nz, e.g. 3×3×1.
- **Append cells around**: shows extra fractional copies of the cell
  along ±a, ±b, ±c *for display only* (handy for seeing neighboring
  cells without committing to a full supercell). **Refresh list/sync
  appended** on the Atoms tab bakes these into real, editable atoms.

### Display tab
- **Max bond (Å)** + **pair-specific bond cutoffs** (per element pair;
  work independently of the global cutoff, including when it's 0).
- **Scene**: legend, unit-cell edges, axes/ticks, cell dimensions
  (with a corner-box or classic 3-D-attached style), perspective vs.
  orthographic projection, depth shading, proportional axes (for very
  elongated cells), and an auto-hide for tick labels on any axis the
  camera is looking straight down (prevents unreadable overlap in
  top-down/edge-on views).
- **Compass**: a/b/c or x/y/z direction indicator, with its own corner
  and fine-position controls, same pattern as the legend and cell-dim
  box.
- **Atom size** / **Panel fill** sliders, and the **View rotation**
  Elevation / Azimuth / Zoom sliders with axis-preset buttons.
- **Per-element colours**: click any swatch to recolor.

### Atoms tab
- Full atom list (position + bond count per atom; 0-bond atoms flagged).
- **Select unconnected (0 bonds)** – one click to find stray/edge atoms.
- **Delete selected** / **Delete all of element**.
- **Refresh list / sync appended** – turns display-only appended cells
  (from the File tab) into real, deletable atoms.

### Symmetry tab
- **Detect symmetry (spglib)** – reports the space group, if spglib is
  installed.
- **Manual symmetry operations** – Jones-faithful syntax (`x,y,z`,
  `-x,-y,-z`, `-x+1/2,y,-z+1/2`, …), one per line, applied to generate
  a symmetric structure from a reduced set of atoms.

### Export tab
Covered in the walkthrough above: page size/orientation/margin,
inter-panel spacing, independent title/caption/tick text sizing, a
fully independent panel grid (angle, zoom, panel fill, and
legend/cell-dim/compass visibility + position *per panel*), a
**Use current** shortcut (per panel, or for every panel at once) that
copies the Display tab's current view in as an editable starting
point, and PDF/SVG export with a custom filename.

---

## Tips

- This script is far from being perfect, especially the option to
  generate multiple panels on one page might not give results
  that you expect; some workaround can be the option to export
  each view as a separate image and then combine them with use
  of some other software.

## Files in this folder

| File | Purpose |
|---|---|
| `Crystal_Visualizer_Colab.ipynb` | The notebook – open this in Colab |
| `crystal_visualizer_colab.py` | Same app as a plain `.py` module, if you'd rather `%run` or import it |
| `MoS2.cif` | Example structure used throughout this README (2H-MoS₂, a = b = 3.162 Å, c = 12.325 Å; Wyckoff positions from Fang *et al.*, arXiv:1205.3794) |
| `MoS2_example_output.pdf` | The rendered result of the walkthrough above |
