---
title: "Structural Bioinformatics with Biopython 1.86"
teaching: 35
exercises: 25
questions:
- "How can I work with protein structures using Biopython 1.86?"
- "What tools does Biopython provide for PDB file manipulation?"
- "How can I analyze protein structures and calculate basic properties?"
- "What methods are available for structural analysis in current Biopython versions?"

objectives:
- "Parse and manipulate PDB files with Biopython 1.86"
- "Extract structural information like sequences and basic properties"
- "Calculate distances, angles, and basic structural metrics"
- "Work with chains, residues, and atoms in protein structures"

keypoints:
- "Biopython's `Bio.PDB` module provides tools for structural bioinformatics"
- "PDB files can be parsed to extract atoms, residues, chains, and models"
- "Basic structural properties like distances and angles can be calculated"
- "Biopython 1.86 has specific syntax for structure manipulation"
- "Always check Biopython version compatibility for structural analysis code"
---

## Introduction to Structural Bioinformatics with Biopython

Biopython's `Bio.PDB` module provides powerful tools for working with macromolecular structures. This lesson focuses on compatible code for Biopython version 1.86, ensuring all examples work with this specific version.

First, let's check the Biopython version and import necessary modules:

```python
import Bio
print(f"Biopython version: {Bio.__version__}")

from Bio.PDB import PDBParser, PPBuilder, Selection
from Bio.PDB.PDBExceptions import PDBConstructionWarning
import warnings
import numpy as np

# Suppress some common warnings for cleaner output
warnings.filterwarnings('ignore', category=PDBConstructionWarning)
```

## Working with PDB Files

### Basic PDB File Parsing

```python
def create_sample_pdb_data():
    """Create sample PDB data for demonstration"""
    sample_pdb = """\
ATOM      1  N   MET A   1      27.360  39.840   5.260  1.00 30.00           N
ATOM      2  CA  MET A   1      26.660  38.650   5.720  1.00 30.00           C
ATOM      3  C   MET A   1      25.160  38.830   5.580  1.00 30.00           C
ATOM      4  O   MET A   1      24.530  39.760   6.100  1.00 30.00           O
ATOM      5  CB  MET A   1      27.280  37.390   5.100  1.00 30.00           C
ATOM      6  N   VAL A   2      24.560  37.940   4.800  1.00 30.00           N
ATOM      7  CA  VAL A   2      23.140  37.980   4.580  1.00 30.00           C
ATOM      8  C   VAL A   2      22.380  36.870   5.240  1.00 30.00           C
ATOM      9  O   VAL A   2      22.910  35.770   5.420  1.00 30.00           O
ATOM     10  CB  VAL A   2      22.700  38.080   3.120  1.00 30.00           C
TER
"""
    # Write to temporary file
    with open('sample.pdb', 'w') as f:
        f.write(sample_pdb)
    return 'sample.pdb'

def parse_pdb_structure(pdb_file):
    """Parse a PDB file and extract basic information"""
    
    # Create parser - QUIET parameter is available in 1.86
    parser = PDBParser(QUIET=True)
    
    # Parse structure
    structure = parser.get_structure('my_structure', pdb_file)
    
    print(f"Structure: {structure}")
    print(f"Number of models: {len(structure)}")
    
    # Iterate through models
    for model in structure:
        print(f"Model {model.id}:")
        print(f"  Number of chains: {len(model)}")
        
        # Iterate through chains
        for chain in model:
            print(f"  Chain {chain.id}:")
            print(f"    Number of residues: {len(chain)}")
            
            # Count atoms properly
            atom_count = 0
            for residue in chain:
                atom_count += len(list(residue.get_atoms()))
            print(f"    Number of atoms: {atom_count}")
            
            # Get residue types
            residue_types = {}
            for residue in chain:
                resname = residue.get_resname()
                residue_types[resname] = residue_types.get(resname, 0) + 1
            
            print(f"    Residue types: {residue_types}")
    
    return structure

# Parse sample structure
sample_file = create_sample_pdb_data()
structure = parse_pdb_structure(sample_file)
```

> ## Challenge 1: Extract Sequence from Structure
>
> Create a function that extracts the amino acid sequence from a PDB structure using Biopython 1.86 compatible methods.
>
>> ## Solution
>>
>> ```python
>> def extract_sequence_from_structure(structure):
>>     """Extract amino acid sequence from PDB structure using Biopython 1.86"""
>>     from Bio.PDB import PPBuilder
>>     from Bio.Seq import Seq
>>     
>>     ppb = PPBuilder()
>>     sequences = []
>>     
>>     for model in structure:
>>         for chain in model:
>>             # Get polypeptides from chain - build_peptides is available in 1.86
>>             for pp in ppb.build_peptides(chain):
>>                 seq = pp.get_sequence()
>>                 sequences.append({
>>                     'chain_id': chain.id,
>>                     'sequence': seq,
>>                     'length': len(seq),
>>                     'sequence_string': str(seq)
>>                 })
>>     
>>     return sequences
>>
>> def analyze_structure_sequence(structure):
>>     """Analyze sequences in a protein structure"""
>>     sequences = extract_sequence_from_structure(structure)
>>     
>>     print("Sequences found in structure:")
>>     for seq_info in sequences:
>>         print(f"Chain {seq_info['chain_id']}:")
>>         print(f"  Length: {seq_info['length']} residues")
>>         print(f"  Sequence: {seq_info['sequence_string']}")
>>         
>>         # Calculate amino acid composition
>>         aa_composition = {}
>>         for aa in seq_info['sequence_string']:
>>             aa_composition[aa] = aa_composition.get(aa, 0) + 1
>>         
>>         print("  Amino acid composition:")
>>         for aa, count in sorted(aa_composition.items()):
>>             percentage = (count / seq_info['length']) * 100
>>             print(f"    {aa}: {count} ({percentage:.1f}%)")
>>         print()
>>     
>>     return sequences
>>
>> # Analyze our sample structure
>> sequences = analyze_structure_sequence(structure)
>> ```
>> {: .language-python}
> {: .solution}
{: .challenge}

## Structural Analysis and Properties

### Calculating Basic Structural Properties

```python
def analyze_structural_properties(structure):
    """Calculate various structural properties compatible with Biopython 1.86"""
    
    properties = {}
    
    for model in structure:
        model_props = {}
        
        for chain in model:
            chain_props = {
                'residues': [],
                'atoms': [],
                'centroid': None,
                'extent': None
            }
            
            # Collect all atoms in chain
            atoms = list(chain.get_atoms())
            if not atoms:
                continue
                
            # Calculate centroid using numpy
            coords = np.array([atom.get_coord() for atom in atoms])
            centroid = coords.mean(axis=0)
            chain_props['centroid'] = centroid
            
            # Calculate extent (max distance from centroid)
            distances = np.linalg.norm(coords - centroid, axis=1)
            max_distance = np.max(distances)
            chain_props['extent'] = max_distance
            
            # Analyze each residue
            for residue in chain:
                if residue.id[0] == ' ':  # Skip hetero/water residues
                    res_props = {
                        'residue_name': residue.get_resname(),
                        'residue_id': residue.id[1],
                        'atoms': [],
                        'centroid': None
                    }
                    
                    res_atoms = list(residue.get_atoms())
                    if res_atoms:
                        res_coords = np.array([atom.get_coord() for atom in res_atoms])
                        res_props['centroid'] = res_coords.mean(axis=0)
                        res_props['atoms'] = [(atom.name, atom.get_coord()) for atom in res_atoms]
                    
                    chain_props['residues'].append(res_props)
            
            model_props[chain.id] = chain_props
        
        properties[model.id] = model_props
    
    return properties

def print_structural_analysis(properties):
    """Print formatted structural analysis"""
    
    for model_id, model_data in properties.items():
        print(f"Model {model_id}:")
        
        for chain_id, chain_data in model_data.items():
            print(f"  Chain {chain_id}:")
            print(f"    Extent: {chain_data['extent']:.2f} Å")
            print(f"    Centroid: [{chain_data['centroid'][0]:.1f}, "
                  f"{chain_data['centroid'][1]:.1f}, {chain_data['centroid'][2]:.1f}]")
            print(f"    Number of residues: {len(chain_data['residues'])}")
            
            # Show first few residues
            print("    First 3 residues:")
            for res in chain_data['residues'][:3]:
                centroid = res['centroid']
                print(f"      {res['residue_name']} {res['residue_id']}: "
                      f"centroid at [{centroid[0]:.1f}, {centroid[1]:.1f}, {centroid[2]:.1f}]")

# Analyze structural properties
props = analyze_structural_properties(structure)
print_structural_analysis(props)
```

### Calculating Distances and Angles

```python
def calculate_distance(atom1, atom2):
    """Calculate distance between two atoms"""
    coord1 = atom1.get_coord()
    coord2 = atom2.get_coord()
    return np.linalg.norm(coord1 - coord2)

def calculate_angle(atom1, atom2, atom3):
    """Calculate angle between three atoms"""
    v1 = atom1.get_coord() - atom2.get_coord()
    v2 = atom3.get_coord() - atom2.get_coord()
    
    # Normalize vectors
    v1_u = v1 / np.linalg.norm(v1)
    v2_u = v2 / np.linalg.norm(v2)
    
    # Calculate angle in radians
    angle_rad = np.arccos(np.clip(np.dot(v1_u, v2_u), -1.0, 1.0))
    
    # Convert to degrees
    return np.degrees(angle_rad)

def calculate_structural_parameters(structure):
    """Calculate bond lengths, angles compatible with Biopython 1.86"""
    
    results = {
        'bond_lengths': [],
        'angles': []
    }
    
    for model in structure:
        for chain in model:
            residues = list(chain.get_residues())
            
            for residue in residues:
                # Skip if not a standard amino acid
                if residue.id[0] != ' ':
                    continue
                
                # Get backbone atoms
                try:
                    n = residue['N']
                    ca = residue['CA']
                    c = residue['C']
                    
                    # Calculate bond lengths
                    n_ca_dist = calculate_distance(n, ca)
                    ca_c_dist = calculate_distance(ca, c)
                    
                    results['bond_lengths'].extend([
                        ('N-CA', n_ca_dist, residue),
                        ('CA-C', ca_c_dist, residue)
                    ])
                    
                    # Calculate angles
                    try:
                        # N-CA-C angle
                        n_ca_c_angle = calculate_angle(n, ca, c)
                        results['angles'].append(('N-CA-C', n_ca_c_angle, residue))
                    except (KeyError, AttributeError):
                        pass
                        
                except KeyError:
                    # Missing backbone atoms
                    continue
    
    return results

def print_structural_parameters(parameters):
    """Print calculated structural parameters"""
    
    print("Bond Lengths:")
    bond_types = {}
    for bond_type, length, residue in parameters['bond_lengths']:
        bond_types.setdefault(bond_type, []).append(length)
    
    for bond_type, lengths in bond_types.items():
        avg_length = np.mean(lengths)
        std_length = np.std(lengths)
        print(f"  {bond_type}: {avg_length:.3f} ± {std_length:.3f} Å (n={len(lengths)})")
    
    print("\nAngles:")
    angle_types = {}
    for angle_type, angle, residue in parameters['angles']:
        angle_types.setdefault(angle_type, []).append(angle)
    
    for angle_type, angles in angle_types.items():
        avg_angle = np.mean(angles)
        std_angle = np.std(angles)
        print(f"  {angle_type}: {avg_angle:.1f} ± {std_angle:.1f}° (n={len(angles)})")

# Calculate and print parameters
params = calculate_structural_parameters(structure)
print_structural_parameters(params)
```

> ## Challenge 2: Residue Neighborhood Analysis
>
> Create a function that finds all residues within a certain distance of a target residue.
>
>> ## Solution
>>
>> ```python
>> def find_residue_neighbors(structure, target_chain_id, target_residue_id, distance_cutoff=8.0):
>>     """Find residues within specified distance of target residue"""
>>     
>>     neighbors = []
>>     
>>     for model in structure:
>>         # Find target residue
>>         target_residue = None
>>         for chain in model:
>>             if chain.id == target_chain_id:
>>                 for residue in chain:
>>                     if residue.id[1] == target_residue_id and residue.id[0] == ' ':
>>                         target_residue = residue
>>                         break
>>                 if target_residue:
>>                     break
>>         
>>         if not target_residue:
>>             print(f"Target residue {target_chain_id}:{target_residue_id} not found")
>>             return neighbors
>>         
>>         # Get target residue CA atom
>>         try:
>>             target_ca = target_residue['CA']
>>         except KeyError:
>>             print("Target residue has no CA atom")
>>             return neighbors
>>         
>>         # Check all other residues
>>         for chain in model:
>>             for residue in chain:
>>                 # Skip target residue itself and non-standard residues
>>                 if (residue == target_residue or residue.id[0] != ' '):
>>                     continue
>>                 
>>                 try:
>>                     residue_ca = residue['CA']
>>                     distance = calculate_distance(target_ca, residue_ca)
>>                     
>>                     if distance <= distance_cutoff:
>>                         neighbors.append({
>>                             'chain_id': chain.id,
>>                             'residue_id': residue.id[1],
>>                             'residue_name': residue.get_resname(),
>>                             'distance': distance
>>                         })
>>                 except KeyError:
>>                     # Residue has no CA atom
>>                     continue
>>     
>>     # Sort by distance
>>     neighbors.sort(key=lambda x: x['distance'])
>>     
>>     return neighbors
>>
>> def print_residue_neighbors(neighbors, target_chain, target_residue, cutoff):
>>     """Print formatted neighbor analysis"""
>>     
>>     print(f"Residues within {cutoff} Å of {target_chain}:{target_residue}:")
>>     print("-" * 60)
>>     
>>     if not neighbors:
>>         print("No neighbors found within specified distance")
>>         return
>>     
>>     for neighbor in neighbors:
>>         print(f"  {neighbor['chain_id']}:{neighbor['residue_id']} "
>>               f"({neighbor['residue_name']}): {neighbor['distance']:.2f} Å")
>>     
>>     print(f"\nTotal neighbors: {len(neighbors)}")
>>
>> # Find neighbors of residue 1 in chain A
>> neighbors = find_residue_neighbors(structure, 'A', 1, 10.0)
>> print_residue_neighbors(neighbors, 'A', 1, 10.0)
>> ```
>> {: .language-python}
> {: .solution}
{: .challenge}

## Working with Multiple Chains and Models

```python
def analyze_complex_structure():
    """Create and analyze a multi-chain structure"""
    
    multi_chain_pdb = """\
ATOM      1  N   MET A   1      27.360  39.840   5.260  1.00 30.00           N
ATOM      2  CA  MET A   1      26.660  38.650   5.720  1.00 30.00           C
ATOM      3  C   MET A   1      25.160  38.830   5.580  1.00 30.00           C
ATOM      4  O   MET A   1      24.530  39.760   6.100  1.00 30.00           O
ATOM      5  CB  MET A   1      27.280  37.390   5.100  1.00 30.00           C
ATOM      6  N   VAL A   2      24.560  37.940   4.800  1.00 30.00           N
ATOM      7  CA  VAL A   2      23.140  37.980   4.580  1.00 30.00           C
ATOM      8  C   VAL A   2      22.380  36.870   5.240  1.00 30.00           C
ATOM      9  O   VAL A   2      22.910  35.770   5.420  1.00 30.00           O
ATOM     10  CB  VAL A   2      22.700  38.080   3.120  1.00 30.00           C
ATOM     11  N   ASP B   1      15.360  35.840   8.260  1.00 30.00           N
ATOM     12  CA  ASP B   1      14.660  34.650   8.720  1.00 30.00           C
ATOM     13  C   ASP B   1      13.160  34.830   8.580  1.00 30.00           C
ATOM     14  O   ASP B   1      12.530  35.760   8.100  1.00 30.00           O
ATOM     15  CB  ASP B   1      15.280  33.390   8.100  1.00 30.00           C
TER
"""
    
    with open('multi_chain.pdb', 'w') as f:
        f.write(multi_chain_pdb)
    
    parser = PDBParser(QUIET=True)
    return parser.get_structure('multi_chain', 'multi_chain.pdb')

def analyze_multi_chain_structure(structure):
    """Analyze a structure with multiple chains"""
    
    print("Multi-chain Structure Analysis:")
    print("=" * 50)
    
    inter_chain_distances = []
    
    for model in structure:
        chains = list(model.get_chains())
        
        print(f"Model {model.id}: {len(chains)} chains")
        
        # Calculate inter-chain distances
        for i, chain1 in enumerate(chains):
            for j, chain2 in enumerate(chains):
                if i < j:  # Avoid duplicate pairs
                    # Get CA atoms from each chain
                    ca_atoms1 = []
                    ca_atoms2 = []
                    
                    for residue in chain1:
                        if residue.id[0] == ' ':
                            try:
                                ca_atoms1.append(residue['CA'])
                            except KeyError:
                                pass
                    
                    for residue in chain2:
                        if residue.id[0] == ' ':
                            try:
                                ca_atoms2.append(residue['CA'])
                            except KeyError:
                                pass
                    
                    if ca_atoms1 and ca_atoms2:
                        # Calculate minimum distance between chains
                        min_distance = float('inf')
                        for atom1 in ca_atoms1:
                            for atom2 in ca_atoms2:
                                distance = calculate_distance(atom1, atom2)
                                if distance < min_distance:
                                    min_distance = distance
                        
                        inter_chain_distances.append({
                            'chain1': chain1.id,
                            'chain2': chain2.id,
                            'min_distance': min_distance
                        })
        
        # Print chain information
        for chain in chains:
            residues = list(chain.get_residues())
            standard_residues = [r for r in residues if r.id[0] == ' ']
            
            print(f"  Chain {chain.id}:")
            print(f"    Total residues: {len(residues)}")
            print(f"    Standard residues: {len(standard_residues)}")
            
            # Count atoms
            atom_count = 0
            for residue in chain:
                atom_count += len(list(residue.get_atoms()))
            print(f"    Atoms: {atom_count}")
    
    # Print inter-chain distances
    if inter_chain_distances:
        print("\nInter-chain distances (minimum CA-CA):")
        for dist_info in inter_chain_distances:
            print(f"  {dist_info['chain1']}-{dist_info['chain2']}: "
                  f"{dist_info['min_distance']:.2f} Å")

# Analyze multi-chain structure
multi_structure = analyze_complex_structure()
analyze_multi_chain_structure(multi_structure)
```

## Basic Structural Alignment

```python
def simple_structure_alignment(structure1, structure2):
    """Perform simple structure alignment using CA atoms"""
    
    # Get CA coordinates from both structures
    coords1 = []
    coords2 = []
    
    # Get first model from each structure
    model1 = list(structure1.get_models())[0]
    model2 = list(structure2.get_models())[0]
    
    # Collect CA atoms from chain A
    if 'A' in model1 and 'A' in model2:
        chain1 = model1['A']
        chain2 = model2['A']
        
        for res1, res2 in zip(chain1.get_residues(), chain2.get_residues()):
            if res1.id[0] == ' ' and res2.id[0] == ' ':  # Standard residues
                try:
                    ca1 = res1['CA']
                    ca2 = res2['CA']
                    coords1.append(ca1.get_coord())
                    coords2.append(ca2.get_coord())
                except KeyError:
                    continue
    
    if len(coords1) < 3 or len(coords2) < 3:
        print("Not enough equivalent CA atoms for alignment")
        return None
    
    # Convert to numpy arrays
    coords1 = np.array(coords1)
    coords2 = np.array(coords2)
    
    # Simple RMSD calculation
    squared_distances = np.sum((coords1 - coords2) ** 2, axis=1)
    rmsd = np.sqrt(np.mean(squared_distances))
    
    # Calculate average distance
    distances = np.linalg.norm(coords1 - coords2, axis=1)
    avg_distance = np.mean(distances)
    max_distance = np.max(distances)
    
    return {
        'rmsd': rmsd,
        'avg_distance': avg_distance,
        'max_distance': max_distance,
        'num_aligned_residues': len(coords1)
    }

def compare_structures(structures):
    """Compare multiple structures"""
    
    print("Structure Comparison:")
    print("=" * 40)
    
    for i, struct1 in enumerate(structures):
        for j, struct2 in enumerate(structures):
            if i < j:
                alignment = simple_structure_alignment(struct1, struct2)
                if alignment:
                    print(f"Structure {i+1} vs Structure {j+1}:")
                    print(f"  RMSD: {alignment['rmsd']:.3f} Å")
                    print(f"  Average distance: {alignment['avg_distance']:.3f} Å")
                    print(f"  Max distance: {alignment['max_distance']:.3f} Å")
                    print(f"  Aligned residues: {alignment['num_aligned_residues']}")
                    print()

# Create a slightly modified structure for comparison
def create_modified_structure():
    """Create a modified structure for alignment testing"""
    modified_pdb = """\
ATOM      1  N   MET A   1      27.360  39.840   5.260  1.00 30.00           N
ATOM      2  CA  MET A   1      26.660  38.650   5.720  1.00 30.00           C
ATOM      3  C   MET A   1      25.160  38.830   5.580  1.00 30.00           C
ATOM      4  O   MET A   1      24.530  39.760   6.100  1.00 30.00           O
ATOM      5  CB  MET A   1      27.280  37.390   5.100  1.00 30.00           C
ATOM      6  N   VAL A   2      24.560  37.940   4.800  1.00 30.00           N
ATOM      7  CA  VAL A   2      23.140  37.980   4.580  1.00 30.00           C
ATOM      8  C   VAL A   2      22.380  36.870   5.240  1.00 30.00           C
ATOM      9  O   VAL A   2      22.910  35.770   5.420  1.00 30.00           O
ATOM     10  CB  VAL A   2      22.700  38.080   3.120  1.00 30.00           C
TER
"""
    with open('modified.pdb', 'w') as f:
        f.write(modified_pdb)
    
    parser = PDBParser(QUIET=True)
    return parser.get_structure('modified', 'modified.pdb')

# Compare structures
structure1 = structure
structure2 = create_modified_structure()
structures = [structure1, structure2]
compare_structures(structures)
```

> ## Challenge 3: Structure Validation
>
> Create a function that performs basic structure validation checks.
>
>> ## Solution
>>
>> ```python
>> def validate_protein_structure(structure):
>>     """Perform basic protein structure validation"""
>>     
>>     validation_results = {
>>         'warnings': [],
>>         'errors': [],
>>         'statistics': {}
>>     }
>>     
>>     for model in structure:
>>         model_stats = {
>>             'total_chains': 0,
>>             'total_residues': 0,
>>             'total_atoms': 0,
>>             'chains': {}
>>         }
>>         
>>         for chain in model:
>>             model_stats['total_chains'] += 1
>>             chain_stats = {
>>                 'residues': 0,
>>                 'atoms': 0,
>>                 'standard_residues': 0,
>>                 'has_ter': False
>>             }
>>             
>>             for residue in chain:
>>                 model_stats['total_residues'] += 1
>>                 chain_stats['residues'] += 1
>>                 
>>                 if residue.id[0] == ' ':
>>                     chain_stats['standard_residues'] += 1
>>                 
>>                 # Check for essential backbone atoms in standard residues
>>                 if residue.id[0] == ' ':
>>                     essential_atoms = ['N', 'CA', 'C']
>>                     missing_atoms = []
>>                     for atom_name in essential_atoms:
>>                         if atom_name not in residue:
>>                             missing_atoms.append(atom_name)
>>                     
>>                     if missing_atoms:
>>                         validation_results['warnings'].append(
>>                             f"Chain {chain.id}, residue {residue.id[1]} "
>>                             f"({residue.get_resname()}): missing atoms {missing_atoms}"
>>                         )
>>                 
>>                 # Count atoms
>>                 atom_count = len(list(residue.get_atoms()))
>>                 model_stats['total_atoms'] += atom_count
>>                 chain_stats['atoms'] += atom_count
>>             
>>             model_stats['chains'][chain.id] = chain_stats
>>         
>>         validation_results['statistics'][model.id] = model_stats
>>     
>>     # Print validation report
>>     print("Structure Validation Report:")
>>     print("=" * 40)
>>     
>>     for model_id, stats in validation_results['statistics'].items():
>>         print(f"Model {model_id}:")
>>         print(f"  Chains: {stats['total_chains']}")
>>         print(f"  Residues: {stats['total_residues']}")
>>         print(f"  Atoms: {stats['total_atoms']}")
>>         
>>         for chain_id, chain_stats in stats['chains'].items():
>>             print(f"  Chain {chain_id}:")
>>             print(f"    Residues: {chain_stats['residues']}")
>>             print(f"    Standard residues: {chain_stats['standard_residues']}")
>>             print(f"    Atoms: {chain_stats['atoms']}")
>>     
>>     if validation_results['warnings']:
>>         print("\nWarnings:")
>>         for warning in validation_results['warnings'][:5]:  # Show first 5 warnings
>>             print(f"  ⚠ {warning}")
>>     
>>     if validation_results['errors']:
>>         print("\nErrors:")
>>         for error in validation_results['errors']:
>>             print(f"  ✗ {error}")
>>     
>>     return validation_results
>>
>> # Validate our sample structure
>> validation = validate_protein_structure(structure)
>> ```
>> {: .language-python}
> {: .solution}
{: .challenge}

## Working with Structure Selection

```python
def select_structure_elements(structure, selection_criteria):
    """Select specific elements from structure based on criteria"""
    
    selected = {
        'residues': [],
        'atoms': []
    }
    
    for model in structure:
        for chain in model:
            for residue in chain:
                # Apply residue selection criteria
                select_residue = True
                
                if 'residue_names' in selection_criteria:
                    if residue.get_resname() not in selection_criteria['residue_names']:
                        select_residue = False
                
                if 'residue_ids' in selection_criteria:
                    if residue.id[1] not in selection_criteria['residue_ids']:
                        select_residue = False
                
                if 'chain_ids' in selection_criteria:
                    if chain.id not in selection_criteria['chain_ids']:
                        select_residue = False
                
                if select_residue and residue.id[0] == ' ':
                    selected['residues'].append({
                        'chain': chain.id,
                        'residue': residue.id[1],
                        'name': residue.get_resname(),
                        'atoms': list(residue.get_atoms())
                    })
                
                # Apply atom selection criteria
                for atom in residue.get_atoms():
                    select_atom = select_residue  # Start with residue selection
                    
                    if 'atom_names' in selection_criteria:
                        if atom.name not in selection_criteria['atom_names']:
                            select_atom = False
                    
                    if 'elements' in selection_criteria:
                        if atom.element not in selection_criteria['elements']:
                            select_atom = False
                    
                    if select_atom:
                        selected['atoms'].append({
                            'chain': chain.id,
                            'residue': residue.id[1],
                            'residue_name': residue.get_resname(),
                            'atom_name': atom.name,
                            'element': atom.element,
                            'coords': atom.get_coord()
                        })
    
    return selected

def print_selection_results(selected, criteria):
    """Print results of structure selection"""
    
    print("Structure Selection Results:")
    print(f"Criteria: {criteria}")
    print("=" * 50)
    
    print(f"Selected residues: {len(selected['residues'])}")
    print(f"Selected atoms: {len(selected['atoms'])}")
    
    if selected['residues']:
        print("\nSelected residues (first 10):")
        for residue in selected['residues'][:10]:
            print(f"  {residue['chain']}:{residue['residue']} ({residue['name']}) - "
                  f"{len(residue['atoms'])} atoms")
    
    if selected['atoms']:
        print("\nSelected atoms (first 10):")
        for atom in selected['atoms'][:10]:
            print(f"  {atom['chain']}:{atom['residue']} {atom['residue_name']}."
                  f"{atom['atom_name']} ({atom['element']})")

# Example selections
print("Backbone atoms selection:")
backbone_selection = select_structure_elements(structure, {
    'atom_names': ['N', 'CA', 'C', 'O']
})
print_selection_results(backbone_selection, "Backbone atoms (N, CA, C, O)")

print("\nCarbon atoms selection:")
carbon_selection = select_structure_elements(structure, {
    'elements': ['C']
})
print_selection_results(carbon_selection, "Carbon atoms")
```


{% include links.md %}
