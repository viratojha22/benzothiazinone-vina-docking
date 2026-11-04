# Benzothiazinone Docking against PDB 6HEZ with AutoDock Vina

Reproducible Google Colab workflow that retrieves 1,3-benzothiazin-4-one derivatives from ChEMBL, selects 100 diverse ligands, and docks them with AutoDock Vina 1.2.5 into the BTZ043-binding site of PDB 6HEZ, followed by scoring and ranking.

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/viratojha22/benzothiazinone-vina-docking/blob/main/Docking.ipynb)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

## Overview

The 1,3-benzothiazin-4-one scaffold is the core of the benzothiazinone (BTZ) class of compounds, which includes BTZ043. This project docks a diverse set of ChEMBL-derived benzothiazinones into the BTZ043 site of PDB 6HEZ and ranks them by predicted binding affinity. The whole pipeline (retrieval, filtering, ligand preparation, docking and ranking) runs end to end in a single Colab notebook.

## Target and ligands

| Item | Details |
|---|---|
| Target structure | PDB ID [6HEZ](https://www.rcsb.org/structure/6HEZ) |
| Receptor used | Chain A only, with the FAD cofactor retained |
| Reference ligand | BTZ043 (PDB ligand code 0SK), used to define the binding site |
| Ligand source | ChEMBL, substructure search for the 1,3-benzothiazin-4-one core |
| Ligands docked | 100 |

### Ligand selection

| Step | Compounds remaining |
|---|---|
| ChEMBL substructure search (core SMILES `O=C1N=CSc2ccccc12`) | 708 |
| Largest-fragment selection, core re-check (SMARTS), duplicate removal | 706 |
| Nitro group present and molecular weight 400-520 | 394 |
| Diversity selection (MaxMin picking on Morgan fingerprints, radius 2, 2048 bits, seed 42) | 100 |

## Workflow

1. Retrieve benzothiazinone derivatives from ChEMBL by substructure search
2. Standardize structures and calculate properties (MW, logP, HBD, HBA, rotatable bonds) with RDKit
3. Filter and select 100 diverse ligands
4. Prepare ligands: add hydrogens, generate 3D coordinates (RDKit), MMFF optimization, convert to PDBQT (Meeko). All 100 ligands prepared successfully
5. Prepare the receptor: extract chain A with FAD, convert to PDBQT with Open Babel (pH 7.4)
6. Define the grid box from the co-crystallized BTZ043 in 6HEZ and run AutoDock Vina in batch
7. Rank ligands by best predicted affinity and export the top 20

## Docking parameters

| Parameter | Value |
|---|---|
| Software | AutoDock Vina 1.2.5 |
| Receptor | Rigid, chain A with FAD |
| Grid center (x, y, z) | 13.87, -20.62, 37.13 |
| Grid size (x, y, z) | 20.1, 20.0, 20.0 Å |
| Exhaustiveness | 8 |
| Number of poses per ligand | 5 |
| Runtime | About 134 minutes for 100 ligands on a Colab CPU runtime |

## Validation

- **Binding-site check:** the grid is centered on the co-crystallized BTZ043 (ligand code 0SK) in chain A. 25 of its 28 ligand atoms lie within 4.5 Å of the prepared receptor.
- **Redocking:** the co-crystallized ligand (nitroso form, as found in 6HEZ) was redocked into the same grid. With the screening settings (exhaustiveness 8, 5 poses), the top-ranked pose had an RMSD of 9.43 Å and the closest pose was 2.37 Å. With more thorough sampling (exhaustiveness 32, 20 poses, seed 42), the top-ranked pose (-9.95 kcal/mol) had an RMSD of 9.45 Å, while several lower-ranked poses were close to the crystal pose, the closest being pose 12 (-8.52 kcal/mol, RMSD 1.60 Å).
- **Interpretation:** Vina can sample the crystallographic binding mode but does not rank it first, so the protocol does not meet the usual criterion of a top-ranked pose within 2 Å of the crystal pose. A likely contributor is that the crystal ligand is covalently bound to Cys387, which Vina cannot model. Binding affinities in this repository should therefore be read as exploratory rankings, not validated predictions.

## Results

Top five ligands by best predicted binding affinity. The full ranking is in `results/all_scores.csv` and the top 20 in `results/top20_scores.csv`.

| Rank | Ligand (ChEMBL ID) | Best affinity (kcal/mol) |
|---|---|---|
| 1 | CHEMBL3394191 | -11.355 |
| 2 | CHEMBL5193230 | -10.997 |
| 3 | CHEMBL5563705 | -10.938 |
| 4 | CHEMBL5181669 | -10.703 |
| 5 | CHEMBL4466845 | -10.679 |

Docked poses for the top 20 ligands are in `results/top_poses/`.

## Repository structure

```
├── Docking.ipynb              Colab notebook (full workflow)
├── data/
│   └── receptor/              Prepared receptor structure (PDB)
├── results/
│   ├── all_scores.csv         Ranked scores for all 100 ligands
│   ├── top20_scores.csv       Top 20 ligands with SMILES, MW and logP
│   └── top_poses/             Docked poses of the top-ranked ligands (PDBQT)
├── LICENSE
└── README.md
```

## How to run

1. Click the **Open in Colab** badge above.
2. Run the cells in order. The notebook downloads AutoDock Vina 1.2.5 and installs RDKit, Meeko, Open Babel and the ChEMBL client.
3. When the notebook asks for the receptor, upload the prepared structure from `data/receptor/` in this repository.
4. The notebook fetches 6HEZ from the RCSB to locate the BTZ043 site, retrieves ligands from ChEMBL, docks them and writes scores to `/content/docking_project/results/`.
5. Download the results from the Colab Files panel.

## Limitations

- Vina scores are computational estimates of binding affinity and do not replace experimental measurements.
- The receptor is treated as rigid, and only chain A with its cofactor is used.
- Vina does not model covalent bond formation, so any covalent binding mode of the reference ligand is not captured.
- The redocking used the nitroso form found in the crystal structure, whereas the screened ligands are nitro compounds docked non-covalently. Nitro-benzothiazinones act as prodrugs that are activated in the bacterium, and this screen does not model that activation.
- The ligand set is restricted to nitro-containing benzothiazinones with MW 400-520, so conclusions apply to this subset only.
- No comparison with experimental activity data has been done.
- Redocking of the co-crystallized ligand did not recover the crystal pose as the top-ranked result (see Validation), and the 100-ligand screen used a lower exhaustiveness (8) than the validation run.
  
## Planned additions

- 2D interaction diagrams for the top hits
- Protein-ligand interaction analysis (hydrogen bonds and contacts)

## Citations

- Trott O, Olson AJ. AutoDock Vina: improving the speed and accuracy of docking with a new scoring function, efficient optimization, and multithreading. *J Comput Chem.* 2010;31:455-461.
- Eberhardt J, Santos-Martins D, Tillack AF, Forli S. AutoDock Vina 1.2.0: new docking methods, expanded force field, and Python bindings. *J Chem Inf Model.* 2021;61:3891-3898.
- Zdrazil B, et al. The ChEMBL Database in 2023: a drug discovery platform spanning multiple bioactivity data types and time periods. *Nucleic Acids Res.* 2024;52:D1180-D1192.
- O'Boyle NM, et al. Open Babel: an open chemical toolbox. *J Cheminform.* 2011;3:33.
- RDKit: open-source cheminformatics, https://www.rdkit.org
- Meeko: https://github.com/forlilab/Meeko
- Protein structure: PDB ID 6HEZ, [authors and publication].

## Acknowledgments

The code in this repository was developed with assistance from Claude (Anthropic). All methods, parameters and results were reviewed and validated by the author.

## License

Released under the [MIT License](LICENSE). ChEMBL-derived data remain under their original CC BY-SA license.

## Author

Virat Ojha, M.Pharm (Medicinal Chemistry)
