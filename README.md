# Project-Computer-Drug-Discovery
A workflow created using tools from the OpenEye Software. This workflow focused on finding an inhibitor for the enzyme Tyrosine Decarboxylase.

# Virtual Screening Workflow using OpenEye

## Project Description

In this project, a virtual screening was performed using OpenEye software. The workflow consisted of:
1. Molecular filtration
2. Flipper (generating stereochemistry)
3. Tautomer generation
4. Conformation generation with OMEGA
5. Shape-based screening with ROCS

# Input and output formats used
| Tool | Input | Output |
|------|--------|--------|
| Filter | input.sdf | filtered.sdf |
| Flipper | filtered.smi | flipper.ism |
| Tautomers | flipper.ism | tautomers.ism |
| OMEGA | tautomers.ism | omega.oeb.gz |
| ROCS | omega_rocs.oeb.gz | rocs.oeb.gz |

Obabel is used to convert the filtered.sdf to filtered.smi
