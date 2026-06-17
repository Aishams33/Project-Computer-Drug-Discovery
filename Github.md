# Ligand Preparation
This small molecules database originates from Enamine and is known as the Hit Locator Library (catalogue number HLL‑460-0-Z-10V).
https://enamine.net/login-page?return=aHR0cHM6Ly9lbmFtaW5lLm5ldC9jb21wb25lbnQvZG93bmxvYWQvP3Rhc2s9ZmlsZS5kb3dubG9hZCZmPTEwNDQ=
## Filtering
```bash
# Move the folder containing documents from the database to a new folder
scp -r Downloads/Enamine_Hit_Locator_Library_plated/ server@name:folder_filter/ 
 
# Running in the folder Enamine_filter/
~/OpenEye/openeye/bin/filter -filter drug -in ../Enamine_Hit_Locator_Library_plated/Enamine_Hit_Locator_Library_plated_460160cmpds_20260301.sdf -out Enamine_Fdrug.sdf

# Molcount 
python3 molcount.py Enamine_Fdrug.sdf 

# For new Enamine db to obtain IDs and convert them to SMILES
obabel enamine_Fdrug.sdf -O Enamine_Fdrug.smi --append "Catalog_ID"
```
## Flipper
```bash
# Running in Enamine_Flipper/
flipper -in Enamine_Fdrug.smi -out Enamine_FdrugFlip.ism -prefix stereo_Enamine -warts true -progress percent -enumBondSpecifiedStereo true 

# Molcount
python3 molcount.py Enamine_FdrugFlip.ism 
```
## Tautomers
```bash
# Running in Enamine_Taut/
tautomers -in ~/ligandprep/newdb_try3/Enamine_Flipper/Enamine_FdrugFlip.ism -out Enamine_FdrugFlipTaut.ism -warts true -maxgenerated 5 -pkanorm true

# Molcount
python3 molcount.py Enamine_FdrugFlipTaut.ism
```
## Omega
``` bash
# Generating conformers for docking 
omega2 pose -mpi_np 14 -in ~/ligandprep/newdb_try3/Enamine_Taut/Enamine_FdrugFlipTaut.ism -out Enamine_FdrugFlipTautOmega.oeb.gz -prefix Omega -warts true -rangeIncrement 5 -maxConfRange 200,800

# Molcount on the conformormer database for docking
python3 -conf molcount.py Enamine_FdrugFlipTautOmega.oeb.gz

# Generating conformers for ROCS 
omega2 rocs -mpi_np 14 -in Enamine_FdrugFlipTaut.ism -out Enamine_FdrugFlipTautOmerocs.oeb.gz -prefix Omerocs -warts true

# Molcount on the conformormer database for ROCS
python3 -conf molcount.py Enamine_FdrugFlipTautOmerocs.oeb.gz
```
## Chunker
``` bash
# Running the tool chunker so the database is split in to even parts
chunker -in Enamine_FdrugFlipTautOmega.oeb.gz -base en_omega -nchunks 200

# To put all oeb.gz files in a folder
mv *.oeb.gz all_chunks/
```
## Fred 
```bash
# This command can be run for a few chunks; if you have many chunks, it is recommended to use a for loop
fred -receptor receptor.oedu -dbase all_chunks/en_omega0000002.oeb.gz -dock_resolution Standard -num_poses 1 -prefix tdc_fred_par2 -nostructs true -mpi_np 20
```
### For loop for the docking
``` python
import os
import subprocess

input_dir = "all_chunks"
output_dir = "results_docking"
os.makedirs(output_dir, exist_ok=True)

receptor = "Receptor.oedu"

for chunk_file in sorted(os.listdir(input_dir)):
    chunk_path = os.path.join(input_dir, chunk_file)
    prefix = os.path.join(output_dir, chunk_file)

    cmd = [
        "fred",
        "-receptor", receptor,
        "-dbase", chunk_path,
        "-dock_resolution", "Standard",
        "-prefix", prefix,
        "-nostructs", true,
        "-num_poses", "1",
        "-mpi_np", "20"
    ]

    print(f"Docking: {chunk_file}")
    subprocess.run(cmd, check=True)

print("All chunks are docked!")
```

``` bash
# Merging all docking score files into a single file
(echo -e "Title\tFRED Chemgauss4 score"; tail -q -n +2 *.txt | sort -k2 -n) > all_scores_sorted.txt
```
## ROCS 
```bash
# Prepping the query
omega2 rocs -mpi_np 20 -in query.ism -out query.oeb.gz -prefix Omerocs -warts true

# Molcount
python3 molcount.py query_Omerocs.oeb.gz

# Running ROCS with mcquery
rocs -mpi_np 20 -dbase Enamine_FdrugFlipTautOmerocs.oeb.gz -query query_Omerocs.oeb.gz -mcquery true -qconflabel title -nostructs true -prefix rocs_db -rankby TanimotoCombo
```
## Redoing the entire workflow for the top 10 hits
``` bash 
grep -E "Hit1|Hit2|Hit3|Hit4|Hit5|Hit6|Hit7|Hit8|Hit9|Hit10" Enamine_Fdrug.smi > top_10_molecules.smi
```
### Flipper
```bash
# Running in Enamine_Flipper/
flipper -in top_10_molecules.smi -out top_10_molecules_FLip.ism -prefix stereo_top_10_flip -warts true -progress percent -enumBondSpecifiedStereo true

# Molcount
python3 molcount.py top_10_molecules_FLip.ism
```
### Tautomers
```bash
# Running in Enamine_Taut/
tautomers -in top_10_molecules_FLip.ism -out top_10_molecules_FLipTaut.ism -warts true -maxgenerated 5 -pkanorm true

# Molcount
python3 molcount.py top_10_molecules_FLipTaut.ism
```
### Omega
``` bash
# Generating conformers for docking 
omega2 pose -mpi_np 22 -in top_10_molecules_FLipTaut.ism -out top_10_molecules_FLipTautOmega.oeb.gz -prefix Omega -warts true -rangeIncrement 5 -maxConfRange 200,800

# Molcount
python3 molcount.py -conf top_10_molecules_FLipTautOmega.oeb.gz 
```
### Fred
``` bash
fred -receptor Receptor.oedu -dbase top_10_molecules_FLipTautOmega.oeb.gz -dock_resolution Standard -num_poses 1 -prefix top_10_fred_docking -nostructs false -mpi_np 22
```
>Several positive controls were used in this study. The positive controls were run exactly the same as the ligand preparation.
# Receptor preparation

## Superposition
``` bash
# Folders with the structures
input_folder= "structures_2" 
output_folder= "outputfolder" 
mkdir -p "$output_folder" 

# Loop over all PDB files
for file in "$input_folder"/*.pdb; do 
for file1 in "$input_folder"/*.pdb; do 

# Filenames without path and extension
base1=$(basename "$file" .pdb) 
base2=$(basename "$file1" .pdb) 

# Doing the superposition 
superposition -i "$file" -r "$file1" -o "$output_folder/${base1}_vs_${base2}.pdb" 

echo "Superposition done: $base1 vs $base2" 

done 
done
```

``` bash
# To merge all output_logs into a large file. It loops through all log files and extracts the first line from each file after a 3-line superposition match.
for f in *.log; do echo $(basename $f); cat $f | grep Superposition -A3 done | head -n1
```
### Spruce
``` bash 
spruce -in protein.pdb -site_residue "residue_of_choice" -build_loops true -loop_db_filename spruce_db.oeb.gz
```
### Makereceptor
Due to problems opening MakeReceptor via the terminal, it was eventually downloaded locally. The software was downloaded locally via:
https://www.eyesopen.com/customer-software-download. After logging into the website, the following was selected: OpenEye Applications -> OpenEye-applications-2025.2.1-macOS-12.7-x64 -> OEDOCKING 4.3.4.1
