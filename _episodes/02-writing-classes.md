---
title: "Writing classes in Bioinformatics"
teaching: 20
exercises: 25
questions:
- "How are classes written in Python for bioinformatics applications?"
- "What do methods look like in bioinformatics classes?"
- "How can a bioinformatics class customize instance construction and validation?"
objectives:
- "Write bioinformatics-related classes from scratch"
- "Write methods targeting biological sequence or genomic data"
- "Write custom `__init__` methods for initialization and validation in bioinformatics contexts"
keypoints:
- "Classes in Python are blocks started with the `class` keyword"
- "Method definitions look like functions, but must take a `self` argument"
- "The `__init__` method is called when instances are constructed"
- "Bioinformatics classes often include validation and domain-specific logic in `__init__`"
- "**Encapsulate biological data** with built-in validation"
- "**Create specialized behaviors** through methods tailored to biological operations"
- "**Ensure data integrity** by validating inputs during object creation"
- "**Build hierarchical relationships** through inheritance that mirror biological relationships"
- "**Create reusable, maintainable code** for complex bioinformatics analyses"
---

## Introduction to Writing Bioinformatics Classes

In bioinformatics, we often need to create custom data structures that represent biological entities like sequences, genes, or genomic features. Python classes provide a powerful way to model these entities with built-in validation and specialized methods.

Let's start by understanding the basic structure of a class and then build up to more complex bioinformatics examples.

## Basic Class Structure

A class is defined using the `class` keyword followed by the class name. Inside the class, we define methods (functions) that operate on instances of the class.

```python
class BiologicalSequence:
    def __init__(self, sequence):
        self.sequence = sequence.upper()

    def get_sequence(self):
        return self.sequence

    def summary(self):
        raise NotImplementedError("Subclass should implement summary")
```

Let's break down what's happening here:

- **`class BiologicalSequence:`** - Defines a new class called `BiologicalSequence`
- **`def __init__(self, sequence):`** - The constructor method that runs when we create a new instance
- **`self.sequence = sequence.upper()`** - Stores the sequence as an attribute, converted to uppercase
- **`get_sequence(self)`** - A method that returns the stored sequence
- **`summary(self)`** - A method that subclasses must implement

> ## Understanding `self`
>
> The `self` parameter refers to the current instance of the class. When you call a method on an object, Python automatically passes the object as the first argument. This allows the method to access and modify the object's attributes.
>
> While you could technically use any name for this first parameter, `self` is the universal convention in Python. Using any other name would confuse other programmers reading your code.
{: .callout}

> ## Class Naming Conventions
>
> In Python, we use **PascalCase** for class names:
> - Start with a capital letter
> - Capitalize the first letter of each word
> - No underscores between words
>
> Examples: `BiologicalSequence`, `DNASequence`, `ProteinCodingGene`
>
> This makes it easy to distinguish classes from variables and functions, which use snake_case.
{: .callout}

## Creating Specialized Sequence Classes

Now let's create specialized subclasses for different types of biological sequences:

```python
class DNASequence(BiologicalSequence):
    def summary(self):
        length = len(self.sequence)
        gc_count = sum(base in "GC" for base in self.sequence)
        gc_content = gc_count / length if length > 0 else 0
        print(f"DNA Sequence Summary:")
        print(f"  Length: {length} bases")
        print(f"  GC content: {gc_content:.2%}")
        print(f"  AT content: {1 - gc_content:.2%}")

    def validate(self):
        """Validate that the sequence contains only valid DNA bases"""
        valid_bases = set("ACGT")
        if not all(base in valid_bases for base in self.sequence):
            invalid_bases = set(self.sequence) - valid_bases
            raise ValueError(f"Invalid DNA bases found: {invalid_bases}")
    
    def reverse_complement(self):
        """Return the reverse complement of the DNA sequence"""
        complement_map = str.maketrans("ACGT", "TGCA")
        return self.sequence.translate(complement_map)[::-1]


class ProteinSequence(BiologicalSequence):
    def summary(self):
        length = len(self.sequence)
        unique_aas = sorted(set(self.sequence))
        hydrophobic_aas = set("ACFILMPVW")
        hydrophobic_count = sum(aa in hydrophobic_aas for aa in self.sequence)
        
        print(f"Protein Sequence Summary:")
        print(f"  Length: {length} amino acids")
        print(f"  Unique AAs: {', '.join(unique_aas)}")
        print(f"  Hydrophobic content: {hydrophobic_count/length:.2%}")

    def validate(self):
        """Validate that the sequence contains only valid amino acids"""
        allowed_aas = "ACDEFGHIKLMNPQRSTVWY*"
        if not all(aa in allowed_aas for aa in self.sequence):
            invalid_aas = set(self.sequence) - set(allowed_aas)
            raise ValueError(f"Invalid amino acids found: {invalid_aas}")
    
    def molecular_weight(self):
        """Calculate approximate molecular weight in Daltons"""
        # Average molecular weight per amino acid is approximately 110 Da
        return len(self.sequence) * 110
```

Now let's use our classes:

```python
# Create DNA and protein sequences
dna_seq = DNASequence("ATGCGTAC")
prot_seq = ProteinSequence("MKTLLL")

# Use their methods
dna_seq.summary()
print(f"Reverse complement: {dna_seq.reverse_complement()}")
print()

prot_seq.summary()
print(f"Approximate molecular weight: {prot_seq.molecular_weight()} Da")
```

> ## Create an RNASequence Class
>
> Create an `RNASequence` class that inherits from `BiologicalSequence` and includes:
> - A `summary()` method showing length and GC content
> - A `validate()` method that checks for valid RNA bases (A, C, G, U)
> - A `reverse_transcribe()` method that returns a DNA sequence (replace U with T)
>
>> ## Solution
>>
>> ```python
>> class RNASequence(BiologicalSequence):
>>     def summary(self):
>>         length = len(self.sequence)
>>         gc_count = sum(base in "GC" for base in self.sequence)
>>         gc_content = gc_count / length if length > 0 else 0
>>         print(f"RNA Sequence Summary:")
>>         print(f"  Length: {length} bases")
>>         print(f"  GC content: {gc_content:.2%}")
>> 
>>     def validate(self):
>>         """Validate that the sequence contains only valid RNA bases"""
>>         valid_bases = set("ACGU")
>>         if not all(base in valid_bases for base in self.sequence):
>>             invalid_bases = set(self.sequence) - valid_bases
>>             raise ValueError(f"Invalid RNA bases found: {invalid_bases}")
>>     
>>     def reverse_transcribe(self):
>>         """Return the DNA sequence after reverse transcription"""
>>         return self.sequence.replace('U', 'T')
>> 
>> # Test the class
>> rna_seq = RNASequence("AUGCCUAA")
>> rna_seq.summary()
>> print(f"DNA after reverse transcription: {rna_seq.reverse_transcribe()}")
>> ```
>> {: .language-python}
> {: .solution}
{: .challenge}

## Customizing Object Initialization with `__init__`

The `__init__` method is crucial for bioinformatics classes because it allows us to validate data immediately when objects are created. This ensures we never have invalid biological data in our objects.

```python
class Gene:
    def __init__(self, name, sequence, chromosome=None, start=None, end=None):
        # Store basic attributes
        self.name = name
        self.sequence = sequence.upper()
        self.chromosome = chromosome
        self.start = start
        self.end = end
        
        # Calculate length if not provided
        if self.end is None and self.start is not None:
            self.end = self.start + len(sequence) - 1
        
        # Validate the gene data
        self.validate()
    
    def validate(self):
        """Validate gene data for consistency"""
        errors = []
        
        # Check sequence
        if not self.sequence:
            errors.append("Gene sequence cannot be empty")
        
        # Check for valid DNA bases
        valid_bases = set("ACGT")
        if not all(base in valid_bases for base in self.sequence):
            invalid_bases = set(self.sequence) - valid_bases
            errors.append(f"Invalid DNA bases: {invalid_bases}")
        
        # Check coordinates if provided
        if self.start is not None and self.end is not None:
            if self.start > self.end:
                errors.append("Start position cannot be greater than end position")
            if self.end - self.start + 1 != len(self.sequence):
                errors.append("Sequence length doesn't match coordinate span")
        
        if errors:
            raise ValueError(f"Gene validation errors: {'; '.join(errors)}")
    
    def gc_content(self):
        """Calculate GC content of the gene"""
        gc_count = self.sequence.count('G') + self.sequence.count('C')
        return gc_count / len(self.sequence)
    
    def __str__(self):
        """String representation of the gene"""
        loc_info = f" on chr{self.chromosome}:{self.start}-{self.end}" if self.chromosome else ""
        return f"Gene {self.name}{loc_info}: {self.sequence[:20]}..." if len(self.sequence) > 20 else self.sequence
```

Let's use our Gene class:

```python
# Create a gene with genomic coordinates
my_gene = Gene("TP53", "ATGCGTAACCGGTT", chromosome=17, start=7668402)

print(my_gene)
print(f"GC content: {my_gene.gc_content():.2%}")
print(f"Length: {len(my_gene.sequence)} bases")

# This will raise a validation error:
# bad_gene = Gene("BAD", "ATGCX")  # Invalid base
```

> ## Add a Method to Calculate Codon Usage
>
> Extend the `Gene` class with a method `codon_usage()` that returns a dictionary showing the frequency of each codon in the gene sequence. Assume the gene sequence length is a multiple of 3.
>
>> ## Solution
>>
>> ```python
>> class Gene:
>>     # ... existing code ...
>>     
>>     def codon_usage(self):
>>         """Calculate frequency of each codon in the gene"""
>>         if len(self.sequence) % 3 != 0:
>>             raise ValueError("Sequence length must be multiple of 3 for codon analysis")
>>         
>>         codon_counts = {}
>>         for i in range(0, len(self.sequence), 3):
>>             codon = self.sequence[i:i+3]
>>             codon_counts[codon] = codon_counts.get(codon, 0) + 1
>>         
>>         # Convert counts to frequencies
>>         total_codons = len(self.sequence) // 3
>>         codon_freq = {codon: count/total_codons for codon, count in codon_counts.items()}
>>         return codon_freq
>> 
>> # Test the method
>> gene = Gene("TEST", "ATGCGATAGCCGTA", chromosome=1, start=100)
>> usage = gene.codon_usage()
>> for codon, freq in sorted(usage.items()):
>>     print(f"{codon}: {freq:.2%}")
>> ```
>> {: .language-python}
> {: .solution}
{: .challenge}

## Inheritance and Specialized Gene Classes

We can create more specialized gene classes that inherit from our base `Gene` class:

```python
class ProteinCodingGene(Gene):
    def __init__(self, name, sequence, protein_id, chromosome=None, start=None, end=None):
        # Call the parent class constructor
        super().__init__(name, sequence, chromosome, start, end)
        
        # Add protein-specific attributes
        self.protein_id = protein_id
        self.protein_sequence = None
        
        # Additional validation for protein-coding genes
        self._validate_protein_coding()
    
    def _validate_protein_coding(self):
        """Additional validation specific to protein-coding genes"""
        if not self.sequence.startswith('ATG'):
            raise ValueError("Protein-coding gene must start with ATG (start codon)")
        
        # Check for valid stop codon at the end
        stop_codons = {'TAA', 'TAG', 'TGA'}
        if self.sequence[-3:] not in stop_codons:
            raise ValueError("Protein-coding gene must end with a stop codon")
    
    def translate(self):
        """Translate the DNA sequence to protein sequence"""
        codon_table = {
            'ATA':'I', 'ATC':'I', 'ATT':'I', 'ATG':'M',
            'ACA':'T', 'ACC':'T', 'ACG':'T', 'ACT':'T',
            'AAC':'N', 'AAT':'N', 'AAA':'K', 'AAG':'K',
            'AGC':'S', 'AGT':'S', 'AGA':'R', 'AGG':'R',
            'CTA':'L', 'CTC':'L', 'CTG':'L', 'CTT':'L',
            'CCA':'P', 'CCC':'P', 'CCG':'P', 'CCT':'P',
            'CAC':'H', 'CAT':'H', 'CAA':'Q', 'CAG':'Q',
            'CGA':'R', 'CGC':'R', 'CGG':'R', 'CGT':'R',
            'GTA':'V', 'GTC':'V', 'GTG':'V', 'GTT':'V',
            'GCA':'A', 'GCC':'A', 'GCG':'A', 'GCT':'A',
            'GAC':'D', 'GAT':'D', 'GAA':'E', 'GAG':'E',
            'GGA':'G', 'GGC':'G', 'GGG':'G', 'GGT':'G',
            'TCA':'S', 'TCC':'S', 'TCG':'S', 'TCT':'S',
            'TTC':'F', 'TTT':'F', 'TTA':'L', 'TTG':'L',
            'TAC':'Y', 'TAT':'Y', 'TAA':'*', 'TAG':'*',
            'TGC':'C', 'TGT':'C', 'TGA':'*', 'TGG':'W',
        }
        
        protein = ""
        for i in range(0, len(self.sequence) - 2, 3):
            codon = self.sequence[i:i+3]
            protein += codon_table.get(codon, '?')
        
        self.protein_sequence = protein
        return protein
    
    def summary(self):
        """Provide a comprehensive summary of the protein-coding gene"""
        print(f"Protein-Coding Gene: {self.name}")
        print(f"  Protein ID: {self.protein_id}")
        print(f"  Location: chr{self.chromosome}:{self.start}-{self.end}")
        print(f"  DNA length: {len(self.sequence)} bp")
        print(f"  GC content: {self.gc_content():.2%}")
        
        if self.protein_sequence:
            print(f"  Protein length: {len(self.protein_sequence)} aa")
        else:
            protein_seq = self.translate()
            print(f"  Protein length: {len(protein_seq)} aa")
```

Let's test our specialized class:

```python
# Create a protein-coding gene
coding_gene = ProteinCodingGene(
    "BRCA1", 
    "ATGGGATGAACCATGAATAA",  # Starts with ATG, ends with TAA
    "NP_009225",
    chromosome=17, 
    start=43000000
)

coding_gene.summary()
print(f"Translated protein: {coding_gene.translate()}")
```

> ## Create a NonCodingGene Class
>
> Create a `NonCodingGene` class that inherits from `Gene` and adds:
> - A `gene_type` attribute (e.g., "rRNA", "tRNA", "lncRNA")
> - A `validate_non_coding()` method that checks for characteristics of non-coding genes
> - A `secondary_structure()` method that returns a placeholder for RNA secondary structure prediction
>
>> ## Solution
>>
>> ```python
>> class NonCodingGene(Gene):
>>     def __init__(self, name, sequence, gene_type, chromosome=None, start=None, end=None):
>>         super().__init__(name, sequence, chromosome, start, end)
>>         self.gene_type = gene_type
>>         self.validate_non_coding()
>>     
>>     def validate_non_coding(self):
>>         """Validate characteristics of non-coding genes"""
>>         if self.gene_type not in ["rRNA", "tRNA", "lncRNA", "miRNA", "snRNA"]:
>>             raise ValueError(f"Unknown non-coding gene type: {self.gene_type}")
>>         
>>         # Additional validation could go here based on gene type
>>         if self.gene_type == "tRNA" and len(self.sequence) not in range(70, 90):
>>             print(f"Warning: tRNA typically 70-90 nucleotides, got {len(self.sequence)}")
>>     
>>     def secondary_structure(self):
>>         """Placeholder for RNA secondary structure prediction"""
>>         return f"Predicted secondary structure for {self.gene_type} {self.name}"
>>     
>>     def summary(self):
>>         """Summary specific to non-coding genes"""
>>         print(f"Non-Coding Gene: {self.name}")
>>         print(f"  Type: {self.gene_type}")
>>         print(f"  RNA length: {len(self.sequence)} nucleotides")
>>         print(f"  GC content: {self.gc_content():.2%}")
>> 
>> # Test the class
>> trna_gene = NonCodingGene("tRNA-Ala", "GGGGCAGTGGCGCAGTTGG", "tRNA", chromosome=6, start=1000)
>> trna_gene.summary()
>> print(trna_gene.secondary_structure())
>> ```
>> {: .language-python}
> {: .solution}
{: .challenge}

## Advanced: Using Properties for Controlled Access

Python properties allow us to control how attributes are accessed and modified:

```python
class GenomicRegion:
    def __init__(self, chromosome, start, end, sequence=""):
        self._chromosome = chromosome
        self._start = start
        self._end = end
        self._sequence = sequence.upper()
        self.validate()
    
    @property
    def chromosome(self):
        return self._chromosome
    
    @chromosome.setter
    def chromosome(self, value):
        if not value or not isinstance(value, (str, int)):
            raise ValueError("Chromosome must be a non-empty string or integer")
        self._chromosome = value
    
    @property
    def start(self):
        return self._start
    
    @start.setter
    def start(self, value):
        if not isinstance(value, int) or value < 0:
            raise ValueError("Start must be a non-negative integer")
        if self._end and value > self._end:
            raise ValueError("Start cannot be greater than end")
        self._start = value
    
    @property
    def length(self):
        if self._start is not None and self._end is not None:
            return self._end - self._start + 1
        return len(self._sequence)
    
    def validate(self):
        """Validate genomic coordinates"""
        if self._start is not None and self._end is not None:
            if self._start > self._end:
                raise ValueError("Start position cannot be greater than end position")
    
    def __str__(self):
        return f"chr{self.chromosome}:{self.start}-{self._end} (length: {self.length})"

# Using properties for controlled access
region = GenomicRegion(1, 1000, 2000, "ATGCGT")
print(region)

# This will work:
region.start = 1500

# This will raise an error:
# region.start = 2500  # Would be greater than end
```

## Putting It All Together: A Complete Example

Let's create a comprehensive example that demonstrates these concepts in a bioinformatics context:

```python
class SequenceCollection:
    """A collection to manage multiple biological sequences"""
    
    def __init__(self, name):
        self.name = name
        self.sequences = []
    
    def add_sequence(self, sequence):
        """Add a sequence to the collection"""
        if not isinstance(sequence, BiologicalSequence):
            raise ValueError("Can only add BiologicalSequence objects")
        self.sequences.append(sequence)
    
    def get_sequence_by_name(self, name):
        """Retrieve a sequence by name"""
        for seq in self.sequences:
            if hasattr(seq, 'name') and seq.name == name:
                return seq
        return None
    
    def gc_content_distribution(self):
        """Calculate GC content distribution across all sequences"""
        gc_contents = []
        for seq in self.sequences:
            if hasattr(seq, 'gc_content'):
                gc_contents.append(seq.gc_content())
        return gc_contents
    
    def summary(self):
        """Provide summary of the entire collection"""
        print(f"Sequence Collection: {self.name}")
        print(f"Total sequences: {len(self.sequences)}")
        
        if self.sequences:
            avg_gc = sum(self.gc_content_distribution()) / len(self.sequences)
            print(f"Average GC content: {avg_gc:.2%}")

# Create and use a sequence collection
collection = SequenceCollection("My Gene Set")

# Add some sequences
collection.add_sequence(DNASequence("ATGCGTACGT"))
collection.add_sequence(ProteinSequence("MKTFLLL"))
collection.add_sequence(DNASequence("GGGCCCATAT"))

collection.summary()
```

{% include links.md %}
