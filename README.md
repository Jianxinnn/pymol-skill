# pymol-skill

Codex/agent skill for generating publication-quality molecular structure
figures with PyMOL.

## Purpose

This skill helps an AI coding agent create reproducible molecular visualization
outputs from structure files or PDB IDs. It guides the agent to write PyMOL
scripts, run PyMOL headlessly, and save figure artifacts.

It also supports iterative figure refinement through PyMOL session files: the
agent can load the previous `.pse` or reuse the previous `.pml`, apply a focused
visual edit, and save a new version.

## Outputs

For each visualization task, the skill aims to produce:

- a `.png` rendered figure;
- a `.pml` PyMOL script for reproducibility;
- a `.pse` PyMOL session for manual editing in the PyMOL GUI.

Outputs are versioned instead of overwritten:

```text
pymol_outputs/<task_slug>/
  scene_v01.pml
  scene_v01.pse
  scene_v01.png
  scene_v02.pml
  scene_v02.pse
  scene_v02.png
```

## Requirements

PyMOL must be available on `PATH`:

```bash
pymol -c -q -d "quit"
```

If it is missing, install the open-source build:

```bash
conda install -c conda-forge pymol-open-source
```

If PyMOL is installed in a conda environment instead of the active shell, run it
through that environment, for example:

```bash
conda run -n proteinhunter pymol -c -q -d "quit"
```

The skill itself does not require the ChatMol Python package. It uses the host
agent's language model and only needs PyMOL for rendering.

Live PyMOL GUI/RPC control is optional. The default workflow is headless and
file-based because it is reproducible and works well in Codex.

## Install For Codex

Copy or symlink this directory into your Codex skills directory:

```bash
mkdir -p ~/.codex/skills
ln -s /path/to/pymol-visualization ~/.codex/skills/pymol-visualization
```

Restart Codex after installation.

## Typical Inputs

- PDB IDs such as `1UBQ`, `6M0J`, or `3WZM`
- local `.pdb`, `.cif`, `.mmcif`, or `.pse` files
- predicted structures from AlphaFold, Boltz, Rosetta, or related tools
- previous `pymol_outputs/` directories for iterative revisions

By default, Codex-adapted outputs are written to `pymol_outputs/` under the
current task directory unless the user requests another path.

## Typical Revision Prompts

- "Rotate the previous figure slightly and zoom into the ligand."
- "Keep the same view, but color chain A blue and chain B gray."
- "Load the last session and add labels for residues 45, 67, and 121."
- "Make a cleaner v2 for a paper figure without changing the structural focus."
