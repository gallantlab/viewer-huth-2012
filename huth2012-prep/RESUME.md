# viewer-huth-2012 → current pycortex — RESUME HERE

**Status (2026-07-02):** Path A chosen. Phase 0 investigation + all non-subject prep is **done**.
Blocked on ONE thing only: **subject S3's pycortex database.** Everything else is understood and staged.

---

## ⛔ WHAT I NEED FROM YOU (the only blocker)

**Subject S3's pycortex db** — the entry the modern engine needs to regenerate geometry + the
vertex↔voxel mapping. Easiest hand-off, in order of preference:

1. **Copy `db/S3` into the local filestore** so `cortex.db.get_surf('S3', ...)` just works:
   drop it next to the existing S1 at `pycortex-src/filestore/db/S3/`
   (needs: `surfaces/` fiducial+inflated+flat for lh & rh, `transforms/`, `overlays.svg`, `anatomicals/`).
2. Or a `tar`/path to the S3 db dir anywhere I can read, + tell me the exact **subject name string**
   the db uses (the old viewer hard-codes `"S3"` — confirm the db subject matches, or give the real name).

**Two things to confirm when you bring it:**
- **Subject name** the db uses (old viewer assumes `S3`).
- **Vertex count**: S3's fiducial lh+rh total. The per-vertex `.bin` maps are **134,880 floats**.
  If S3's nverts ≠ 134,880, the 2012 data is on a different mesh (e.g. fsaverage / a decimated
  display mesh) and we'll need the mapping. Resolve this FIRST when data arrives.

---

## ✅ Already verified / done (no S3 needed)

- **Shell** (`index.html`) is already modernized (Gallant Lab nav bar + citation) — leave it.
- **Engine** (`viewer.html`) = legacy: 1.88 MB, three.js **r50dev** + old `glab`/`MRIview`
  (`new MRIview`) — PRE-`make_static`. `reengine.py` CANNOT be used (proved: it fails with
  `no dataset.fromJSON( found` because these descriptors don't exist in this generation).
- **Geometry** `surface.ctm` = standard **OpenCTM v5 / MG2** (first blob 65,460 v / 130,916 t;
  `surface.json` offsets `[0,1172611]` → base + "superinflated" morph concatenated).
- **Data** = raw Float32 `.bin`: `First PC.bin`/`Semantic Space.bin`/`ModelPerformance.bin`
  (per-vertex, 134,880 floats); `catmapdata/S3-{i}.bin` = per-category vertex maps (1,705);
  `voxeldata/S3-{voxind}.bin` = per-voxel semantic vectors (33,612 files, 1,708 floats each).
- **Working clone:** `/tmp/vh2012-work-89504` (full, 1.7 GB). Never clone into `~/CLAUDE/pycortex`.

### Bespoke word-cloud picker — fully reverse-engineered & preserved in this folder
The picker is an interactive **SVG category graph** synced to the brain (`Graph` prototype, old
`viewer.html` ~L11382–11510). Extracted assets (index-aligned to the `.bin` category dimension):
- `nodenames.json` — 1,705 WordNet synsets, **index order = catmapdata/voxeldata dimension order**.
- `wndefs.json` — synset → definition (hover tooltips).
- `colordata.json` — synset → `[hexColor, size]` (the "Semantic Space" RGB word-cloud layout).
- `wngraph.svg` — the laid-out node graph (1,590 `<circle>` nodes + 39 big labels).

Behaviors to re-implement on the modern engine:
- `picker.callback = showvoxel(voxind, vertind)`: click voxel → load `voxeldata/S3-{voxind}.bin` →
  color each graph node red/blue sized by `2*sqrt(|val|)`; also show `ModelPerformance` r for the voxel.
- `showcategory(catname)`: index i in nodenames → load `catmapdata/S3-{i}.bin` → add brain overlay
  (`cmap RdBu_r, min -3, max 3`).
- `showrgb()`: paint the Semantic-Space RGB layout from `colordata`.
- Old API to map to modern mriview: `viewer.addData/rmData`, `viewer.datasets.X.textures[0][vertind]`,
  `viewer.getVert(vert).pos`, `viewer.picker.callback`. Old data loader = `NParray.fromURL` (Float32 XHR).

---

## Build plan once S3 arrives (native path)

1. **Reconcile vertices**: confirm S3 lh+rh fiducial nverts == 134,880 (else find the mesh/mapping).
2. **Load data**: read each `.bin` → numpy → `cortex.Vertex(values, 'S3')` for First PC / Semantic
   Space / ModelPerformance.
3. **Bake modern viewer**: `cortex.webgl.make_static(outdir, {...3 datasets...})` with S3 → native
   r69/mriview geometry + data mosaics + engine. Confirm ROIs come from S3's `overlays.svg`.
4. **Re-author picker** as a custom layer over the baked viewer (new `convert_huth2012.py`, sibling to
   `convert_huth.py`): inject `wngraph.svg` + nodenames/wndefs/colordata, wire modern picker →
   voxeldata/catmapdata using the `.bin` conventions above. Voxel index from a picked vertex comes from
   S3's mapper (the [32,100,100] mask maps cortical voxels ↔ vertices).
5. **Carry over huth-2016 gotchas**: viewopts `brightness`/`contrast`/`smoothness` backfill (or dat.gui
   crashes to "Loading brain…"); Firefox iframe keyboard focus; `.dg.main` control positioning; verify
   brain position (2012 has NO external scalp mesh, so the centering regression likely won't bite).
6. **Verify in real Firefox** vs. the current live viewer (rotate/flatten, 3 maps, voxel-click word
   cloud, category-click overlay). Back up `master` → private `viewer-huth-2012-backup` BEFORE pushing.
   Ship to `master` (GitHub Pages; `.nojekyll` already present).

Tooling home = `pycortex-roidraw` (sibling to `convert_huth.py`/`reengine.py`). pycortex-src =
`v1.3.2-11-g3ee0bbd3` (r69/mriview). Note: native path needs a FULL pycortex install (scipy/nibabel)
— heavier than huth-2016, which only used the webgl templates. No `cortex` is importable locally yet.
