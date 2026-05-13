---
name: pymol-visualization
description: >
  Generate publication-quality molecular visualization images with PyMOL.
  Use for PDB/CIF/PSE structure figures, protein cartoons, ligand binding
  sites, protein-protein interfaces, mutation highlights, surfaces, structure
  comparisons, Goodsell-style renders, ray-traced molecular graphics, and
  iterative revisions of existing PyMOL figures or sessions, including concise
  post-render assembly of comparison panels when the story depends on multiple
  PyMOL views.
---

# PyMOL Visualization Skill

Generate reproducible molecular structure figures with PyMOL: a `.pml` script,
a rendered `.png`, and an editable `.pse` session.

## Prerequisites

PyMOL must be installed. Check with:
```bash
pymol -c -q -d "quit" >/dev/null && echo "PyMOL available" || echo "PyMOL not found"
```
If missing: `conda install -c conda-forge pymol-open-source`.

The skill can be triggered without PyMOL, but rendering requires a working
PyMOL command. If PyMOL is only installed in a conda environment, run through
that environment, for example `conda run -n proteinhunter pymol -c -q script.pml`.

## Operating Modes

Use the smallest mode that fits:

1. **New figure**: write a fresh `.pml` from a PDB ID or structure file.
2. **File-session iteration**: revise the latest `.pml` or load the latest
   `.pse`, then save a new version. This is the default for multi-turn edits.
3. **Live PyMOL session**: only when the user explicitly says PyMOL is running
   with RPC/GUI and asks to control it. Still save `.pml`, `.png`, and `.pse`.

## Workflow

### 1. Ask the User

Ask only for missing information that blocks the figure:
- **Structure**: PDB ID, local file, existing `.pse`, or predicted model path.
- **Goal**: overview, binding site, interface, active site, mutation, surface,
  alignment, or another clear visual target.
- **Style**: paper, presentation, clean technical, or artistic.

If the user asks for a revision of an existing output, first locate the newest
`.pml`, `.pse`, and `.png` in the task's output directory and infer the intended
starting point from file names and modification times.

### 2. Read the Right Reference

Read `references/recipes.md` before writing scene-specific commands. It contains
the reusable patterns for binding sites, interfaces, active sites, mutations,
surfaces, alignments, labels, distances, and Goodsell-style renders.

### 3. Write and Run a Versioned Script

Use a task-local output directory and never overwrite prior versions:

```bash
mkdir -p pymol_outputs/<task_slug>
pymol -c -q pymol_outputs/<task_slug>/scene_v01.pml
```

If `pymol` is not on `PATH`, replace it with the environment-qualified command,
for example `conda run -n proteinhunter pymol -c -q ...`.

For revisions, create `scene_v02.*`, `scene_v03.*`, etc. Prefer editing the
previous `.pml`; if only a `.pse` exists, start with `load previous.pse`.
Make one focused visual change per revision unless the user asks for a broader
restyle.

### 4. Deliver Output

Always deliver:
1. **PNG image**: rendered figure
2. **PML script**: reproducible commands
3. **PSE session**: editable PyMOL session

Before claiming success, verify all three files exist and the PNG/PSE are
non-empty. Return absolute paths.

### 5. Assemble Comparison Panels When Needed

Use this only when the user asks for a story figure, comparison panel, gallery,
report image, or slide-ready molecular visual. Keep it small; PyMOL still owns
the molecular render, while the assembler only arranges real rendered PNGs and
labels.

1. Inspect the rendered PNGs before composing. Do not infer image content from
   filenames alone.
2. Write one sentence for the figure's claim before layout, such as a
   conformational shift, binding-site change, mutant effect, ligand pose,
   interface difference, or model-confidence comparison.
3. Use a small versioned assembler in the same output directory, such as
   `assemble_v01.py` or `assemble_v01.mjs`. Save the composite as
   `panel_v01.png` and keep source renders unchanged.
4. Attach every label and metric to the exact panel, state, chain, residue,
   pose, mutant, time point, or model it describes. Never mix metrics from
   different panels into one undifferentiated block.
5. Make the comparison rule explicit for the project:
   - fit/compatibility panels: favorable evidence should be visually and
     numerically separated from unfavorable evidence;
   - contrast/control panels: explain whether the expected signal is gain,
     loss, displacement, exposure, burial, contact change, clash, distance, or
     confidence change;
   - proxy metrics such as contacts, clashes, PAE, pLDDT, distances, and buried
     area must be named as proxies unless independently validated.
6. Prefer one question per composite, 2-4 panels, shared camera/color rules,
   and minimal labels. Remove decorative layout that does not clarify the claim.
7. Write a tiny manifest or caption with source image paths, structure/session
   paths, metric definitions, and the interpretation caveat.
8. Before delivery, view the composite at final size and check that text is
   readable, labels point to the right state, no elements overlap, and all
   molecular panels are real PyMOL renders.

If the user asks for a full deck or PDF, finish these assembled figures first,
then pass them to the presentation workflow rather than expanding this skill
into a slide-generation system.

## Script Template

New figure:

```pml
reinitialize

# Load
fetch 4HHB, async=0
# or: load /path/to/structure.pdb, myprotein

# Clean
remove solvent
remove elem H
set valence, 0

# Base look
bg_color white
space cmyk
set ray_shadow, 0
set ray_trace_mode, 1
set antialias, 3
set ambient, 0.5
set spec_count, 5
set shininess, 50
set specular, 1
set reflect, 0.1
set orthoscopic, on
set opaque_background, off
set cartoon_oval_length, 1
set cartoon_rect_length, 1
set cartoon_discrete_colors, on
dss

# Representation - adapt from references/recipes.md
hide everything
show cartoon

# Color
util.color_chains("(all) and elem C", _self=cmd)
util.cnc("all", _self=cmd)

# Camera
orient

save pymol_outputs/example/scene_v01.pse
ray 2400, 1800
png pymol_outputs/example/scene_v01.png, dpi=150
quit
```

For a revision from an existing session:

```pml
reinitialize
load pymol_outputs/example/scene_v01.pse

# Requested edit only: rotate view, recolor, relabel, adjust zoom, etc.

save pymol_outputs/example/scene_v02.pse
ray 2400, 1800
png pymol_outputs/example/scene_v02.png, dpi=150
quit
```

Optional live PyMOL/RPC helper, only after the user confirms `pymol -R` is
running:

```python
from xmlrpc import client

pymol = client.ServerProxy("http://localhost:9123/RPC2")
pymol.do("set cartoon_transparency, 0.3")
pymol.do("save pymol_outputs/example/live_edit_v01.pse")
pymol.do("png pymol_outputs/example/live_edit_v01.png, width=1800, height=1350, ray=1")
```

## Key Rules

1. Use `space cmyk` for print-oriented colors.
2. Remove solvent and hydrogens unless requested.
3. Save `.pse` before ray tracing.
4. Use `async=0` with `fetch`.
5. End batch scripts with `quit`.
6. Render at 1200x900 or larger.
7. Version outputs instead of overwriting.
8. Use live PyMOL/RPC only when requested.
