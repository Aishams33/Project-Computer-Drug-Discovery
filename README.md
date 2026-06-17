# Project-Computer-Drug-Discovery
A workflow created using tools from the OpenEye Software. This workflow focused on finding an inhibitor for the enzyme Tyrosine Decarboxylase of *E.faecalis*.

# Requirements
- OpenEye license
- Linux, macOS of Windows
- Command line 

## Virtual screening workflow using OpenEye

In this project, a virtual screening was performed using OpenEye software. The workflow was split in to the ligand- and receptor preparation. The ligand preparation consisted of:
1. Filtration
2. Flipper (generating stereochemistry)
3. Tautomer generation
4. Conformation generation with OMEGA
5. Molecular docking with FRED
6. Docking poses visualized with VIDA
7. Shape-based screening with ROCS

The receptor preparation consisted of:
1. building loops and adding hydrogen atoms with SPRUCE 
2. MakeReceptor (to visualize the receptor)

## File Formats Used for Input and Output
| Tool | Input | Output |
|------|--------|--------|
| Filter | input.sdf | filtered.sdf |
| Flipper | filtered.smi | flipper.ism |
| Tautomers | flipper.ism | tautomers.ism |
| OMEGA | tautomers.ism | omega.oeb.gz |
| ROCS | omega_rocs.oeb.gz | rocs.oeb.gz |
| SPRUCE | proetin.db | receptor.oedu |

Open Babel (OBabel) was used to convert the filtered output file into SMILES format. 

## Authors
Ayça Korkut and Aisha Salad - Avans University of Applied Sciences, School of Life Sciences and Technology
