# Bio-RTD Guide for Manas

This guide explains how to run the bio-rtd examples on your machine.

## What is Bio-RTD?

Bio-RTD is a Python library for modeling **Residence Time Distributions (RTD)** of integrated continuous biomanufacturing processes. It simulates how biological molecules (like monoclonal antibodies) flow through various unit operations in downstream processing.

---

## Setup

### 1. Activate the Conda Environment

```bash
conda activate manasphd
```

### 2. Navigate to the bio-rtd Directory

```bash
cd /path/to/bio-rtd
```

### 3. Set Python Path (Required for Examples)

```bash
export PYTHONPATH="$PWD"
```

Or combine into one command:
```bash
conda activate manasphd && cd /path/to/bio-rtd && export PYTHONPATH="$PWD"
```

---

## Template Examples

These are configuration templates showing how to set up individual unit operations. They don't produce visual output - they just demonstrate the API.

### Dilution Template
**What it does:** Models adding a dilution buffer to a process stream.
- `dilution_ratio=1.6` means 60% buffer is added
- `c_add_buffer` defines buffer composition

```bash
python examples/templates/fc_uo/dilution_template.py
```

### Concentration Template
**What it does:** Models a concentration step (like ultrafiltration).
- `flow_reduction=8` means output flow is 1/8 of input (8x concentration)
- `non_retained_species` defines what passes through (like salt)
- `relative_losses` defines product loss percentage

```bash
python examples/templates/fc_uo/concentration_template.py
```

### Flow-Through Chromatography Template
**What it does:** Models flow-through chromatography where product passes through while impurities bind.
- `pdf` defines peak dispersion (how concentration spreads)
- `v_void` is column void volume
- `FlowThroughWithSwitching` handles multiple columns with periodic switching

```bash
python examples/templates/fc_uo/flow_through_template.py
```

### CSTR (Surge Tank) Template
**What it does:** Models a Continuous Stirred Tank Reactor - a surge tank providing ideal mixing.
- Smooths out concentration variations
- `v_void` or `rt_target` defines tank size
- Place between semi-continuous and continuous operations

```bash
python examples/templates/surge_tank/cstr_template.py
```

### PCC (Periodic Column Chromatography) Template
**What it does:** Models the most complex unit operation - multi-column chromatography with cycling.
- Defines all cycle phases: equilibration, loading, wash, elution, regeneration
- `load_bt` defines binding/breakthrough behavior
- `peak_shape_pdf` defines elution peak shape

```bash
python examples/templates/sc_uo/pcc_template.py
```

---

## Model Examples (With Visualization)

These examples run simulations and open interactive plots in your browser.

### Single PCC Model
**What it does:** Simulates a complete PCC operation and shows:
- Inlet vs outlet concentration profiles
- Elution peak shape and collection window
- Binding capacity curves
- Cycle-by-cycle analysis

```bash
python examples/models/single_pcc.py
```

**Output:** Opens browser with 5+ interactive Bokeh plots.

---

### Single UF/DF Model
**What it does:** Models Ultrafiltration/Diafiltration using 3 chained operations:
1. Concentration (10x volume reduction)
2. Buffer Exchange (95% buffer replaced)
3. FlowThrough (residence time distribution)

Shows how protein concentrates and salt is removed.

```bash
python examples/models/single_uf_df.py
```

**Output:** Opens browser with 2 plots (protein and salt profiles).

---

### Single Twin CSTR Model
**What it does:** Demonstrates two alternating surge tanks.
- While one fills, the other drains
- Smooths periodic input into continuous output

```bash
python examples/models/single_twin_cstr.py
```

**Output:** Opens browser showing inlet pulses converted to smooth output.

---

### Integrated mAb Processing Train
**What it does:** Simulates a complete monoclonal antibody downstream process:

| Step | Unit Operation | Purpose |
|------|----------------|---------|
| 1 | FlowThroughWithSwitching | Cell removal (depth filtration) |
| 2 | PCC | Protein A capture chromatography |
| 3 | CSTR | Surge tank to buffer elution output |
| 4 | FlowThrough | Virus inactivation (68 min hold) |
| 5 | FlowThroughWithSwitching | AEX polishing |
| 6 | ComboUO | UF/DF (40x concentration + buffer exchange) |

Simulates 24 hours of processing.

```bash
python examples/models/integrated_mab.py
```

**Output:**
- Opens browser with 7 plots (one per step)
- Prints PCC cycle material balance to terminal

---

### Integrated mAb GUI (Interactive)
**What it does:** Interactive web application where you can adjust parameters in real-time:
- Inlet concentration and flow rate
- PCC column size
- CSTR volume
- AEX column size

```bash
bokeh serve --show examples/models/integrated_mab_gui.py
```

**Output:** Opens browser at `http://localhost:5006/integrated_mab_gui` with sliders to adjust parameters.

**To stop:** Press `Ctrl+C` in terminal.

---

## Documentation Tutorials

Step-by-step tutorials for learning bio-rtd:

```bash
python examples/documentation/tutorial_1a.py
python examples/documentation/tutorial_1b.py
python examples/documentation/tutorial_1c.py
python examples/documentation/tutorial_1d.py
python examples/documentation/tutorial_1e.py
```

---

## Quick Reference

### Run Any Example
```bash
conda activate manasphd
cd /path/to/bio-rtd
export PYTHONPATH="$PWD"
python examples/models/single_pcc.py
```

### Run Interactive GUI
```bash
conda activate manasphd
cd /path/to/bio-rtd
export PYTHONPATH="$PWD"
bokeh serve --show examples/models/integrated_mab_gui.py
```

### Run All Tests
```bash
conda activate manasphd
cd /path/to/bio-rtd
python -m pytest bio_rtd_test/ -v
```

---

## Key Concepts

| Term | Meaning |
|------|---------|
| **RTD** | Residence Time Distribution - how long molecules spend in a unit |
| **PCC** | Periodic Column Chromatography - multi-column capture |
| **CSTR** | Continuous Stirred Tank Reactor - ideal mixing tank |
| **UF/DF** | Ultrafiltration/Diafiltration - concentration + buffer exchange |
| **CV** | Column Volume - standard unit for chromatography |
| **PDF** | Probability Distribution Function - peak shape |
| **mAb** | Monoclonal Antibody - the product being processed |

---

## Troubleshooting

### "No module named 'examples'"
You forgot to set PYTHONPATH:
```bash
export PYTHONPATH="$PWD"
```

### "No module named 'bio_rtd'"
The package isn't installed. Run:
```bash
pip install -e .
```

### Plots don't open
Make sure you have a default browser set. Or check terminal for the URL and open manually.

### Bokeh server won't start
Check if port 5006 is in use. Kill any existing bokeh processes:
```bash
pkill -f "bokeh serve"
```

---

## Environment Details

The `manasphd` conda environment includes:
- Python 3.10
- numpy, scipy (core scientific computing)
- bokeh (interactive visualization)
- pandas, openpyxl, xlrd (Excel support)
- pytest (testing)

Created with:
```bash
conda create -n manasphd python=3.10 -y
conda activate manasphd
pip install -e .
pip install bokeh pandas xlrd openpyxl pytest
```
