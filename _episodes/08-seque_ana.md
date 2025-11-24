---
title: "Sequence Analysis with Biopython"
teaching: 40
exercises: 30
questions:
- "How can I manipulate and analyze biological sequences using Biopython?"
- "What tools does Biopython provide for sequence alignment and comparison?"
- "How can I work with sequence features and annotations?"
- "What methods are available for sequence translation and transcription?"
objectives:
- "Create and manipulate sequence objects with Biopython"
- "Perform sequence alignments and calculate similarity scores"
- "Work with sequence features and annotations"
- "Translate DNA/RNA sequences to proteins"
- "Analyze sequence properties like GC content, molecular weight, etc."
keypoints:
- "Biopython's `Seq` objects provide biological sequence manipulation with validation"
- "`SeqIO` module handles reading and writing various sequence file formats"
- "Pairwise and multiple sequence alignments can be performed with Biopython"
- "Sequence features store biological annotations like genes, CDS, and regulatory elements"
- "Biopython includes utilities for sequence translation, transcription, and reverse complement"
- "Sequence properties like molecular weight and GC content are easily calculable"
- "`Seq` objects for biological sequence manipulation with validation"
- "`SeqIO` for reading/writing various sequence file formats"
- "Alignment tools for pairwise and multiple sequence comparisons"
- "Feature handling for biological annotations"
- "Sequence properties** like GC content, molecular weight, and codon usage"
- "Translation and transcription with genetic code support"
---

## Introduction to Biopython Sequence Analysis

Biopython provides comprehensive tools for biological sequence manipulation and analysis. In this lesson, we'll explore how to work with sequences, perform alignments, analyze sequence properties, and handle biological annotations.

## Creating and Manipulating Sequence Objects

Biopython's `Seq` object is the foundation for sequence manipulation, providing biological context beyond plain strings.

```python
from Bio.Seq import Seq
from Bio.SeqRecord import SeqRecord
from Bio import SeqIO

# Creating DNA, RNA, and protein sequences
dna_seq = Seq("ATGCGTACGTAGCTAGCTAG")
rna_seq = Seq("AUGCGUACGUAGCUAGCUAG")
protein_seq = Seq("MVTVSKRER")

print(f"DNA sequence: {dna_seq}")
print(f"RNA sequence: {rna_seq}")
print(f"Protein sequence: {protein_seq}")

# Basic sequence operations
print(f"DNA length: {len(dna_seq)}")
print(f"First 10 bases: {dna_seq[:10]}")
print(f"Reverse: {dna_seq[::-1]}")
```

### Biological Sequence Operations

```python
# Biological operations
print("Biological operations:")
print(f"Complement: {dna_seq.complement()}")
print(f"Reverse complement: {dna_seq.reverse_complement()}")
print(f"Transcribed RNA: {dna_seq.transcribe()}")
print(f"Back transcribed DNA: {rna_seq.back_transcribe()}")

# Translation
coding_dna = Seq("ATGGCCATTGTAATGGGCCGCTGAAAGGGTGCCCGATAG")
print(f"Standard translation: {coding_dna.translate()}")
print(f"Translation with stop codon: {coding_dna.translate(to_stop=True)}")
print(f"Translation table 2 (Vertebrate Mitochondrial): {coding_dna.translate(table=2)}")
```

> ## Challenge 1: Sequence Validation and Translation
>
> Create a function that validates a DNA sequence and translates it to protein, handling invalid sequences gracefully.
>
>> ## Solution
>>
>> ```python
>> def validate_and_translate(dna_sequence):
>>     """Validate DNA sequence and translate to protein"""
>>     from Bio.Seq import Seq
>>     
>>     # Convert to uppercase and remove whitespace
>>     clean_seq = ''.join(dna_sequence.upper().split())
>>     
>>     # Validate characters
>>     valid_bases = set('ATCGN')
>>     if not all(base in valid_bases for base in clean_seq):
>>         invalid_bases = set(clean_seq) - valid_bases
>>         raise ValueError(f"Invalid DNA bases: {invalid_bases}")
>>     
>>     # Create Seq object
>>     seq_obj = Seq(clean_seq)
>>     
>>     # Check if length is multiple of 3 for complete codons
>>     if len(seq_obj) % 3 != 0:
>>         print("Warning: Sequence length not multiple of 3")
>>     
>>     # Translate
>>     protein = seq_obj.translate(to_stop=True)
>>     
>>     return {
>>         'dna_sequence': seq_obj,
>>         'protein_sequence': protein,
>>         'dna_length': len(seq_obj),
>>         'protein_length': len(protein),
>>         'complete_translation': len(protein) * 3 == len(seq_obj)
>>     }
>>
>> # Test the function
>> test_sequences = [
>>     "ATGGCCATTGTAATGGGCCGCTGAAAGGGTGCCCGATAG",  # Valid
>>     "ATGGCCATTGTAATGGGCCGCTGAAAGGGTGCCCGATA",   # Not multiple of 3
>>     "ATGGCCATTXTAATGGGCCGCTGAAAGGGTGCCCGATAG"   # Invalid base
>> ]
>>
>> for i, seq in enumerate(test_sequences):
>>     print(f"Sequence {i+1}: {seq}")
>>     try:
>>         result = validate_and_translate(seq)
>>         print(f"  Protein: {result['protein_sequence']}")
>>         print(f"  Complete: {result['complete_translation']}")
>>     except ValueError as e:
>>         print(f"  Error: {e}")
>>     print()
>> ```
>> {: .language-python}
> {: .solution}
{: .challenge}

## Working with Sequence Files

Biopython's `SeqIO` module provides unified interface for reading and writing various sequence file formats.

```python
from Bio import SeqIO
import io

# Creating sequence records
record = SeqRecord(
    Seq("ATGCGTACGTAGCTAGCTAGCTAGCTAGCTAGCTAGCTAGCTAGC"),
    id="TEST001",
    name="TestGene",
    description="A test gene sequence",
    annotations={"molecule_type": "DNA", "date": "2024-01-01"}
)

print(f"ID: {record.id}")
print(f"Description: {record.description}")
print(f"Sequence: {record.seq}")
print(f"Length: {len(record)}")
print(f"Annotations: {record.annotations}")

# Writing to different formats
formats = ["fasta", "genbank", "embl"]
for fmt in formats:
    output = io.StringIO()
    SeqIO.write(record, output, fmt)
    print(f"\n{fmt.upper()} format:")
    print(output.getvalue()[:100] + "...")
```

### Reading Sequence Files

```python
# Example of reading sequences from string (in practice, you'd use files)
fasta_data = """>gene1 Human insulin
ATGCGTACGTAGCTAGCTAGCTAGCTAGCTAG
>gene2 Mouse insulin
ATGCGTACGTAGCTAGCTAGCTAGCTAGCTAGCTAGC
>gene3 Rat insulin  
ATGCGTACGTAGCTAGCTAGCTAGCTAGC"""

# Parse from string
records = list(SeqIO.parse(io.StringIO(fasta_data), "fasta"))
print(f"Parsed {len(records)} sequences")

for record in records:
    print(f"{record.id}: {len(record.seq)} bp, {record.description}")
```

> ## Challenge 2: Sequence File Converter
>
> Create a function that converts between different sequence file formats (FASTA, GenBank, EMBL).
>
>> ## Solution
>>
>> ```python
>> def convert_sequence_file(input_file, output_file, input_format, output_format):
>>     """Convert sequence files between different formats"""
>>     try:
>>         # Read sequences
>>         records = list(SeqIO.parse(input_file, input_format))
>>         print(f"Read {len(records)} sequences from {input_file}")
>>         
>>         # Write to new format
>>         count = SeqIO.write(records, output_file, output_format)
>>         print(f"Successfully converted {count} sequences to {output_format} format")
>>         print(f"Output file: {output_file}")
>>         
>>         return True
>>         
>>     except Exception as e:
>>         print(f"Error converting file: {e}")
>>         return False
>>
>> def convert_sequence_data(sequence_data, input_format, output_format):
>>     """Convert sequence data between formats using StringIO"""
>>     from io import StringIO
>>     
>>     # Read from string
>>     input_handle = StringIO(sequence_data)
>>     records = list(SeqIO.parse(input_handle, input_format))
>>     
>>     # Write to string
>>     output_handle = StringIO()
>>     SeqIO.write(records, output_handle, output_format)
>>     
>>     return output_handle.getvalue()
>>
>> # Example usage with string data
>> fasta_data = """>test_gene
>> ATGCGTACGTAGCTAGCTAGCTAGCTAGCTAGCTAGCTAGCTAGC"""
>>
>> # Convert FASTA to GenBank
>> genbank_data = convert_sequence_data(fasta_data, "fasta", "genbank")
>> print("FASTA to GenBank conversion:")
>> print(genbank_data)
>> ```
>> {: .language-python}
> {: .solution}
{: .challenge}

## Sequence Alignment

Biopython provides tools for both pairwise and multiple sequence alignments.

### Pairwise Alignment

```python
from Bio import Align
from Bio.Align import substitution_matrices

def pairwise_alignment(seq1, seq2, matrix_name="BLOSUM62"):
    """Perform global pairwise alignment"""
    # Create aligner object
    aligner = Align.PairwiseAligner()
    
    # Set alignment parameters
    aligner.mode = 'global'
    aligner.substitution_matrix = substitution_matrices.load(matrix_name)
    aligner.open_gap_score = -10
    aligner.extend_gap_score = -0.5
    
    # Perform alignment
    alignments = aligner.align(seq1, seq2)
    
    print(f"Number of alignments: {len(alignments)}")
    print(f"Alignment score: {alignments.score:.2f}")
    
    # Get the best alignment
    best_alignment = alignments[0]
    
    print("\nBest alignment:")
    print(best_alignment)
    
    return best_alignment

# Example sequences
seq_a = "ACGTAGCTAGCTAGCTAGCT"
seq_b = "ACGTTGCTAGCTAGCTAGCT"

alignment = pairwise_alignment(seq_a, seq_b)
```

### Multiple Sequence Alignment

```python
from Bio.Align import MultipleSeqAlignment
from Bio.SeqRecord import SeqRecord

def create_multiple_alignment(sequences):
    """Create and analyze a multiple sequence alignment"""
    # Create sequence records
    records = []
    for i, seq in enumerate(sequences):
        record = SeqRecord(Seq(seq), id=f"seq_{i+1}", description=f"Sequence {i+1}")
        records.append(record)
    
    # Create alignment (in practice, you'd use an aligner like ClustalO or MUSCLE)
    alignment = MultipleSeqAlignment(records)
    
    print(f"Alignment length: {alignment.get_alignment_length()}")
    print(f"Number of sequences: {len(alignment)}")
    
    # Calculate basic statistics
    print("\nAlignment statistics:")
    for i in range(alignment.get_alignment_length()):
        column = alignment[:, i]
        unique_bases = len(set(column))
        if unique_bases == 1:
            print(f"Position {i+1}: Conserved ({column[0]})")
    
    return alignment

# Example sequences for multiple alignment
test_sequences = [
    "ACGTAGCTAGCTAGCTAGCT",
    "ACGTTGCTAGCTAGCTAGCT", 
    "ACGTAGCTAGCTAGCTAGCT",
    "ACGTCGCTAGCTAGCTAGCT"
]

msa = create_multiple_alignment(test_sequences)
```

> ## Challenge 3: Sequence Similarity Analysis
>
> Create a function that calculates similarity metrics between multiple sequences and identifies conserved regions.
>
>> ## Solution
>>
>> ```python
>> def analyze_sequence_similarity(sequences):
>>     """Analyze similarity between multiple sequences"""
>>     from Bio.Align import MultipleSeqAlignment
>>     from Bio.SeqRecord import SeqRecord
>>     
>>     # Create alignment
>>     records = [SeqRecord(Seq(seq), id=f"seq_{i}") for i, seq in enumerate(sequences)]
>>     alignment = MultipleSeqAlignment(records)
>>     
>>     results = {
>>         'alignment_length': alignment.get_alignment_length(),
>>         'num_sequences': len(alignment),
>>         'conserved_positions': [],
>>         'similarity_matrix': [],
>>         'gc_content': []
>>     }
>>     
>>     # Find conserved positions
>>     for pos in range(alignment.get_alignment_length()):
>>         column = alignment[:, pos]
>>         if len(set(column)) == 1:  # All sequences have same base at this position
>>             results['conserved_positions'].append({
>>                 'position': pos + 1,
>>                 'base': column[0],
>>                 'is_conserved': True
>>             })
>>         else:
>>             results['conserved_positions'].append({
>>                 'position': pos + 1, 
>>                 'bases': list(set(column)),
>>                 'is_conserved': False
>>             })
>>     
>>     # Calculate pairwise similarities
>>     for i, seq1 in enumerate(sequences):
>>         row = []
>>         for j, seq2 in enumerate(sequences):
>>             if i == j:
>>                 similarity = 100.0
>>             else:
>>                 # Simple similarity calculation
>>                 matches = sum(1 for a, b in zip(seq1, seq2) if a == b)
>>                 similarity = (matches / min(len(seq1), len(seq2))) * 100
>>             row.append(round(similarity, 1))
>>         results['similarity_matrix'].append(row)
>>     
>>     # Calculate GC content for each sequence
>>     for seq in sequences:
>>         gc_count = sum(1 for base in seq if base in 'GC')
>>         gc_percent = (gc_count / len(seq)) * 100
>>         results['gc_content'].append(round(gc_percent, 1))
>>     
>>     # Print results
>>     print(f"Alignment Analysis:")
>>     print(f"Length: {results['alignment_length']} positions")
>>     print(f"Sequences: {results['num_sequences']}")
>>     print(f"Conserved positions: {sum(1 for pos in results['conserved_positions'] if pos['is_conserved'])}")
>>     
>>     print("\nSimilarity Matrix (%):")
>>     for row in results['similarity_matrix']:
>>         print("  " + " ".join(f"{val:5.1f}" for val in row))
>>     
>>     print("\nGC Content (%):")
>>     for i, gc in enumerate(results['gc_content']):
>>         print(f"  Sequence {i+1}: {gc}%")
>>     
>>     return results
>>
>> # Test with sample sequences
>> test_seqs = [
>>     "ATGCGATCGATCGATCGATCG",
>>     "ATGCGCTCGATCGATCGATCG", 
>>     "ATGCGATCGATCGATCGATCG",
>>     "ATGCGATCGATCGATCGATCA"
>> ]
>>
>> analysis = analyze_sequence_similarity(test_seqs)
>> ```
>> {: .language-python}
> {: .solution}
{: .challenge}

## Sequence Features and Annotations

Biopython provides robust handling of sequence features like genes, CDS, exons, and other biological annotations.

```python
from Bio.SeqFeature import SeqFeature, FeatureLocation
from Bio.SeqRecord import SeqRecord
from Bio.Seq import Seq

def create_annotated_sequence():
    """Create a sequence record with biological features"""
    
    # Create a DNA sequence
    dna_sequence = Seq("ATGGCCATTGTAATGGGCCGCTGAAAGGGTGCCCGATAGCTAGCTAGCTAGCTAGCTAGC")
    
    # Create sequence record
    record = SeqRecord(
        dna_sequence,
        id="TEST_GENE_001",
        name="TestGene",
        description="A test gene with multiple features",
        annotations={
            "molecule_type": "DNA",
            "topology": "linear",
            "data_file_division": "PHG",
            "date": "15-JAN-2024",
            "accessions": ["TEST001"],
            "source": "Synthetic construct"
        }
    )
    
    # Add features
    # CDS feature
    cds_feature = SeqFeature(
        location=FeatureLocation(0, 33),  # First 33 bases (11 codons)
        type="CDS",
        qualifiers={
            "gene": ["test_gene"],
            "product": ["test protein"],
            "transl_table": ["1"],
            "codon_start": ["1"],
            "translation": ["MAIVMGR*"]  # Translation of first 33 bases
        }
    )
    
    # Gene feature
    gene_feature = SeqFeature(
        location=FeatureLocation(0, 60),
        type="gene",
        qualifiers={
            "gene": ["test_gene"],
            "note": ["Synthetic test gene"]
        }
    )
    
    # Regulatory feature (promoter-like)
    regulatory_feature = SeqFeature(
        location=FeatureLocation(-20, 0),  # Upstream region
        type="regulatory",
        qualifiers={
            "regulatory_class": ["promoter"],
            "note": ["Putative promoter region"]
        }
    )
    
    # Add features to record
    record.features.extend([gene_feature, cds_feature, regulatory_feature])
    
    return record

# Create and examine annotated sequence
annotated_seq = create_annotated_sequence()

print(f"Sequence ID: {annotated_seq.id}")
print(f"Length: {len(annotated_seq)} bp")
print(f"Number of features: {len(annotated_seq.features)}")

print("\nFeatures:")
for feature in annotated_seq.features:
    print(f"  Type: {feature.type}")
    print(f"  Location: {feature.location}")
    if feature.qualifiers:
        for key, values in feature.qualifiers.items():
            print(f"    {key}: {', '.join(values)}")
    print()
```

### Working with GenBank Files

```python
def parse_genbank_features(genbank_record):
    """Extract and analyze features from a GenBank record"""
    
    print(f"Record: {genbank_record.id} - {genbank_record.description}")
    print(f"Sequence length: {len(genbank_record)} bp")
    print(f"Number of features: {len(genbank_record.features)}")
    
    # Count features by type
    feature_types = {}
    for feature in genbank_record.features:
        feature_type = feature.type
        feature_types[feature_type] = feature_types.get(feature_type, 0) + 1
    
    print("\nFeature types:")
    for ftype, count in sorted(feature_types.items()):
        print(f"  {ftype}: {count}")
    
    # Extract CDS features and their translations
    print("\nCDS Features:")
    for feature in genbank_record.features:
        if feature.type == "CDS":
            print(f"  Location: {feature.location}")
            if 'gene' in feature.qualifiers:
                print(f"  Gene: {feature.qualifiers['gene'][0]}")
            if 'product' in feature.qualifiers:
                print(f"  Product: {feature.qualifiers['product'][0]}")
            if 'translation' in feature.qualifiers:
                translation = feature.qualifiers['translation'][0]
                print(f"  Protein length: {len(translation)} aa")
                print(f"  Translation: {translation[:30]}..." if len(translation) > 30 else f"  Translation: {translation}")
            print()

# Example of creating a mock GenBank record for demonstration
mock_seq = Seq("ATGGCCATTGTAATGGGCCGCTGAAAGGGTGCCCGATAG" * 10)  # Longer sequence
mock_record = SeqRecord(mock_seq, id="MOCK001", annotations={"molecule_type": "DNA"})

# Add some features
mock_record.features = [
    SeqFeature(FeatureLocation(0, 36), type="CDS", qualifiers={
        "gene": ["insulin"], 
        "product": ["insulin precursor"],
        "translation": ["MALWMRLLPLLALLALWGPDPAAAFVNQHLCGSHLVEALYLVCGERGFFYTPKT"]
    }),
    SeqFeature(FeatureLocation(40, 76), type="CDS", qualifiers={
        "gene": ["glucagon"],
        "product": ["glucagon precursor"], 
        "translation": ["HSQGTFTSDYSKYLDSRRAQDFVQWLMNTKRNRNNIA"]
    })
]

parse_genbank_features(mock_record)
```

> ## Challenge 4: Feature Extraction and Analysis
>
> Create a function that extracts specific types of features from sequence records and analyzes their properties.
>
>> ## Solution
>>
>> ```python
>> def analyze_sequence_features(records, feature_type="CDS"):
>>     """Analyze specific feature types across multiple sequence records"""
>>     
>>     analysis_results = {
>>         'total_records': len(records),
>>         'feature_type': feature_type,
>>         'features_found': 0,
>>         'feature_details': [],
>>         'statistics': {}
>>     }
>>     
>>     for record in records:
>>         record_features = [f for f in record.features if f.type == feature_type]
>>         
>>         for feature in record_features:
>>             analysis_results['features_found'] += 1
>>             
>>             feature_info = {
>>                 'record_id': record.id,
>>                 'location': str(feature.location),
>>                 'length': len(feature),
>>                 'qualifiers': {}
>>             }
>>             
>>             # Extract common qualifiers
>>             for qualifier in ['gene', 'product', 'note', 'translation']:
>>                 if qualifier in feature.qualifiers:
>>                     feature_info['qualifiers'][qualifier] = feature.qualifiers[qualifier][0]
>>             
>>             analysis_results['feature_details'].append(feature_info)
>>     
>>     # Calculate statistics
>>     if analysis_results['features_found'] > 0:
>>         lengths = [f['length'] for f in analysis_results['feature_details']]
>>         analysis_results['statistics'] = {
>>             'total_features': analysis_results['features_found'],
>>             'avg_length': sum(lengths) / len(lengths),
>>             'min_length': min(lengths),
>>             'max_length': max(lengths),
>>             'features_per_record': analysis_results['features_found'] / len(records)
>>         }
>>     
>>     # Print summary
>>     print(f"Feature Analysis: {feature_type}")
>>     print(f"Records analyzed: {analysis_results['total_records']}")
>>     print(f"Features found: {analysis_results['features_found']}")
>>     
>>     if analysis_results['statistics']:
>>         stats = analysis_results['statistics']
>>         print(f"Average feature length: {stats['avg_length']:.1f} bp")
>>         print(f"Feature length range: {stats['min_length']} - {stats['max_length']} bp")
>>         print(f"Features per record: {stats['features_per_record']:.2f}")
>>     
>>     print("\nFeature details (first 5):")
>>     for feature in analysis_results['feature_details'][:5]:
>>         print(f"  Record: {feature['record_id']}")
>>         print(f"    Location: {feature['location']}")
>>         print(f"    Length: {feature['length']} bp")
>>         if 'gene' in feature['qualifiers']:
>>             print(f"    Gene: {feature['qualifiers']['gene']}")
>>         if 'product' in feature['qualifiers']:
>>             print(f"    Product: {feature['qualifiers']['product']}")
>>         print()
>>     
>>     return analysis_results
>>
>> # Create test records with features
>> test_records = []
>> for i in range(3):
>>     seq = Seq("ATGGCCATTGTAATGGGCCGCTGAAAGGGTGCCCGATAG" * (i + 2))
>>     record = SeqRecord(seq, id=f"TEST_{i+1:03d}")
>>     
>>     # Add CDS features
>>     record.features = [
>>         SeqFeature(FeatureLocation(0, 36), type="CDS", qualifiers={
>>             "gene": [f"gene_{i+1}A"],
>>             "product": [f"test protein A{i+1}"],
>>             "translation": ["MALWMRLLPLLALLALWGPDPAAAFVNQHLCGSHLV"]
>>         }),
>>         SeqFeature(FeatureLocation(40, 76), type="CDS", qualifiers={
>>             "gene": [f"gene_{i+1}B"], 
>>             "product": [f"test protein B{i+1}"],
>>             "note": ["synthetic sequence"]
>>         })
>>     ]
>>     test_records.append(record)
>>
>> # Analyze CDS features
>> cds_analysis = analyze_sequence_features(test_records, "CDS")
>> ```
>> {: .language-python}
> {: .solution}
{: .challenge}

## Sequence Properties and Analysis

Biopython provides various utilities for analyzing sequence properties.

```python
from Bio.SeqUtils import GC, molecular_weight, six_frame_translations
from Bio.Data import CodonTable

def analyze_sequence_properties(sequence):
    """Comprehensive analysis of sequence properties"""
    
    results = {}
    
    # Basic properties
    results['length'] = len(sequence)
    results['gc_content'] = GC(sequence)
    
    # Molecular weight (for DNA)
    results['molecular_weight'] = molecular_weight(sequence, seq_type="DNA")
    
    # Nucleotide composition
    results['nucleotide_composition'] = {
        'A': sequence.count('A'),
        'T': sequence.count('T'), 
        'G': sequence.count('G'),
        'C': sequence.count('C'),
        'other': len(sequence) - (sequence.count('A') + sequence.count('T') + 
                                 sequence.count('G') + sequence.count('C'))
    }
    
    # Calculate percentages
    total = len(sequence)
    for base in results['nucleotide_composition']:
        if base != 'other':
            results['nucleotide_composition'][f'{base}_percent'] = (
                results['nucleotide_composition'][base] / total * 100
            )
    
    # Six-frame translation
    if len(sequence) >= 3:
        results['six_frame_translations'] = six_frame_translations(sequence)
    
    return results

def print_sequence_analysis(sequence, description=""):
    """Print formatted sequence analysis"""
    
    analysis = analyze_sequence_properties(sequence)
    
    print(f"Sequence Analysis: {description}")
    print("=" * 50)
    print(f"Length: {analysis['length']} bp")
    print(f"GC Content: {analysis['gc_content']:.2f}%")
    print(f"Molecular Weight: {analysis['molecular_weight']:.2f} g/mol")
    
    print("\nNucleotide Composition:")
    comp = analysis['nucleotide_composition']
    for base in ['A', 'T', 'G', 'C']:
        print(f"  {base}: {comp[base]} ({comp[f'{base}_percent']:.1f}%)")
    
    if comp['other'] > 0:
        print(f"  Other: {comp['other']} bases")
    
    # Print first frame of six-frame translation if available
    if 'six_frame_translations' in analysis:
        print("\nSix-frame translation (first frame):")
        translations = analysis['six_frame_translations']
        lines = translations.split('\n')
        for line in lines[:10]:  # Show first 10 lines
            print(line)

# Test with a sample sequence
test_sequence = Seq("ATGGCCATTGTAATGGGCCGCTGAAAGGGTGCCCGATAGCTAGCTAGCTAGCTAGCTAGC")
print_sequence_analysis(test_sequence, "Test Gene Sequence")
```

## Advanced Sequence Manipulation

### Working with Codon Tables

```python
def analyze_codon_usage(sequence, genetic_code=1):
    """Analyze codon usage in a coding sequence"""
    
    if len(sequence) % 3 != 0:
        raise ValueError("Sequence length must be multiple of 3 for codon analysis")
    
    # Get genetic code table
    codon_table = CodonTable.ambiguous_dna_by_id[genetic_code]
    
    codon_counts = {}
    amino_acid_counts = {}
    
    # Count codons and amino acids
    for i in range(0, len(sequence), 3):
        codon = str(sequence[i:i+3])
        
        if codon in codon_table.forward_table:
            amino_acid = codon_table.forward_table[codon]
        elif codon in codon_table.stop_codons:
            amino_acid = '*'
        else:
            amino_acid = '?'
        
        # Update counts
        codon_counts[codon] = codon_counts.get(codon, 0) + 1
        amino_acid_counts[amino_acid] = amino_acid_counts.get(amino_acid, 0) + 1
    
    # Calculate frequencies
    total_codons = len(sequence) // 3
    codon_frequencies = {codon: count/total_codons for codon, count in codon_counts.items()}
    aa_frequencies = {aa: count/total_codons for aa, count in amino_acid_counts.items()}
    
    return {
        'codon_counts': codon_counts,
        'amino_acid_counts': amino_acid_counts,
        'codon_frequencies': codon_frequencies,
        'amino_acid_frequencies': aa_frequencies,
        'total_codons': total_codons,
        'genetic_code': genetic_code
    }

def print_codon_analysis(analysis):
    """Print formatted codon usage analysis"""
    
    print("Codon Usage Analysis:")
    print(f"Genetic Code: {analysis['genetic_code']}")
    print(f"Total Codons: {analysis['total_codons']}")
    
    print("\nAmino Acid Composition:")
    for aa, count in sorted(analysis['amino_acid_counts'].items()):
        freq = analysis['amino_acid_frequencies'][aa]
        print(f"  {aa}: {count} ({freq:.1%})")
    
    print("\nCodon Usage (sorted by frequency):")
    for codon, freq in sorted(analysis['codon_frequencies'].items(), 
                             key=lambda x: x[1], reverse=True)[:10]:
        count = analysis['codon_counts'][codon]
        print(f"  {codon}: {count} ({freq:.1%})")

# Example usage
coding_sequence = Seq("ATGGCCATTGTAATGGGCCGCTGAAAGGGTGCCCGATAG")
codon_analysis = analyze_codon_usage(coding_sequence)
print_codon_analysis(codon_analysis)
```



{% include links.md %}
