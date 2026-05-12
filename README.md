# pymol-skill

Codex/agent skill for generating publication-quality molecular structure
figures with PyMOL.

## Purpose

This skill helps an AI coding agent create reproducible molecular visualization
outputs from structure files or PDB IDs. It guides the agent to write PyMOL
scripts, run PyMOL headlessly, and save figure artifacts.

## Outputs

For each visualization task, the skill aims to produce:

- a `.png` rendered figure;
- a `.pml` PyMOL script for reproducibility;
- a `.pse` PyMOL session for manual editing in the PyMOL GUI.

## Requirements

PyMOL must be available on `PATH`:

```bash
pymol -c -q -e "print('ok')"
```

If it is missing, install the open-source build:

```bash
conda install -c conda-forge pymol-open-source
```

The skill itself does not require the ChatMol Python package. It uses the host
agent's language model and only needs PyMOL for rendering.

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

By default, Codex-adapted outputs are written to `pymol_outputs/` under the
current task directory unless the user requests another path.

