# Reproducing the PETN paper results

This checkout is set up to regenerate the PETN morphology results from
`CGkMC.pdf` / arXiv:2509.25490 using the local fork.

## Repository setup

The fork remote is:

```bash
origin  https://github.com/Satchit4/cgkmc.git
```

The upstream remote is:

```bash
upstream  https://github.com/jwjeffr/cgkmc.git
```

## Local Python environment

Use Python 3.9 through 3.12 from the repository root. The package pins
`numpy ~= 2.0.2` and `scipy ~= 1.13.1`, so Python 3.13+ is not a supported
environment for this checkout.

On this Mac, install Python 3.12 with Homebrew if `python3.12` is not already
available:

```bash
brew install python@3.12
```

```bash
rm -rf .venv
python3.12 -m venv .venv
source .venv/bin/activate
python --version
python -m pip install --upgrade pip
python -m pip install -e ".[dev]"
```

For a quick package check after installation:

```bash
python -m pytest
```

## Quick CLI smoke run

The small cubic example should finish quickly and verifies that the command line
entry point can read JSON input, run a simulation, and write a dump.

```bash
python -m cgkmc \
  --input_file examples/simple_cubic.json \
  --dump_file results/simple_cubic.dump \
  --dump_every 100 \
  --log_file results/simple_cubic.log
```

## Full PETN reproduction run

The paper-scale PETN input is in `examples/petn_paper.json`. It matches the
reported simulation tables:

- PETN cell: tetragonal, `a = b = 9.087 A`, `c = 6.738 A`
- basis: `(0, 0, 0)` and `(1/2, 1/2, 1/2)`
- lattice: `62 x 62 x 140` unit cells
- interaction energies: `-0.294`, `-0.184`, `-0.002 eV`
- cutoffs: `7.0`, `7.5`, `9.5 A`
- beta: `38.68 eV^-1` at `300 K`
- diffusivity: `1.0e11 A^2/s`
- solubility limit: `1.0e-4 A^-3`
- initial radius: `75 A`
- desired final size: `40000`
- KMC iterations: `1000000`

Run it with:

```bash
python -m cgkmc \
  --input_file examples/petn_paper.json \
  --dump_file results/petn.dump \
  --dump_every 1000 \
  --log_file results/petn.log
```

The first full PETN run builds `.kmc_cache/` for a 1,076,320-site lattice. Expect
substantial runtime and memory use. Later runs reuse the cached interaction
matrix as long as the input geometry and interactions are unchanged.

## Regenerating the paper figures

The CLI writes a LAMMPS-style trajectory dump and an energy/occupancy log. These
are the inputs needed for the results figures:

- Figure 2: combine `results/petn.log`, atom counts from `results/petn.dump`,
  and per-frame surface areas exported from OVITO.
- Figure 3: load `results/petn.dump` in OVITO and inspect the final morphology.

OVITO settings used in the paper:

- Modifier: `Construct surface mesh`
- Method: alpha-shape
- Probe radius: `7 A`
- Smoothing level: `100`

For Figure 2, the paper computes:

```text
gamma(t) = (E(t) - E_coh * N(t)) / S(t)
```

with `E_coh = -1.034 eV`. The paper compares the KMC curve against
`gamma_AE ~= 88 mJ/m^2 ~= 5.5 meV/A^2`. The expected final morphology has
dominant `{110}` and `{101}` facets.
