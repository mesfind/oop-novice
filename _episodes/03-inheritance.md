---
title: "Inheritance in Bioinformatics"
teaching: 20
exercises: 20
questions:
- "How can a subclass represent a specific biological type of a parent class?"
- "How can child classes extend or replace parent behaviour?"
objectives:
- "Use inheritance to organise biological sequence classes"
- "Override and extend parent methods using super"
keypoints:
- "Subclass syntax uses ParentClass inside parentheses"
- "Child classes inherit attributes and methods from the parent"
- "A method with the same name replaces the parent implementation"
- "super allows controlled reuse of parent logic"
keypoints:
- **Subclass syntax**: Use `ChildClass(ParentClass)` to create inheritance
- **Inheritance**: Child classes automatically get all methods and attributes from parents
- **Method overriding**: A method with the same name in the child class replaces the parent implementation
- **super()**: Use `super()` to call parent class methods and extend their functionality
- **Mixins**: Special classes that provide additional functionality to be mixed into other classes
- **Composition**: Sometimes "has-a" relationships are better than "is-a" relationships for code organization
---

## What is Inheritance?

In programming, inheritance allows you to create new classes that are specialized versions of existing classes. The original class is called the **parent class** or **superclass**, and the new class is called the **child class** or **subclass**.

In bioinformatics, this is particularly useful because different types of biological sequences (DNA, RNA, proteins) share common features but also have unique characteristics. Inheritance helps us avoid repeating code while maintaining the specific behaviors of each sequence type.

## Basic Inheritance Structure

Here's a simple diagram showing how inheritance works with biological sequences:

```
BiologicalSequence (parent class - general features)
    ↓
DNASequence (child class - DNA-specific features)
    ↓  
RNASequence (child class - RNA-specific features)
```

Each child class **inherits** all the methods and attributes from its parent, but can also add new ones or modify existing ones.

## Creating a Custom Exception

Before we start with sequences, let's create a custom error class for handling invalid biological sequences. This will help us provide clear error messages when someone tries to create a sequence with invalid characters.

```python
class InvalidSequenceError(ValueError):
    """Raised when a sequence contains invalid characters for its type"""
    pass
```

This creates a new type of error that we can use specifically for sequence validation problems.

## Parent Class: BiologicalSequence

Let's start with a general biological sequence class that contains features common to all sequence types. This parent class will handle basic sequence operations that work the same for DNA, RNA, and proteins.

```python
class BiologicalSequence:
    def __init__(self, seq):
        # Convert to uppercase and store the sequence
        self.seq = seq.upper()

    def length(self):
        """Return the length of the sequence"""
        return len(self.seq)

    def gc(self):
        """Calculate GC content as fraction of sequence"""
        return sum(b in "GC" for b in self.seq) / len(self.seq)
    
    def __str__(self):
        """Return the sequence when printed"""
        return self.seq
```

This class provides:
- Sequence storage (converted to uppercase)
- Length calculation
- GC content calculation
- A string representation

## DNA Subclass

Now let's create a DNA-specific class that **inherits** from BiologicalSequence. The DNA class will add DNA-specific validation and methods.

```python
class DNASequence(BiologicalSequence):
    def __init__(self, seq):
        # Call the parent class constructor first
        super().__init__(seq)
        # Then add DNA-specific validation
        self._check()

    def _check(self):
        """Validate DNA sequence contains only A,C,G,T"""
        if not all(b in "ACGT" for b in self.seq):
            raise InvalidSequenceError(f"Invalid DNA character in {self.seq}")

    def complement(self):
        """Return complement of DNA sequence"""
        m = str.maketrans("ACGT", "TGCA")
        return self.seq.translate(m)
    
    def reverse_complement(self):
        """Return reverse complement - common in bioinformatics"""
        return self.complement()[::-1]
```

Key points:
- `DNASequence(BiologicalSequence)` means DNASequence inherits from BiologicalSequence
- `super().__init__(seq)` calls the parent class constructor
- `_check()` adds DNA-specific validation
- `complement()` and `reverse_complement()` are DNA-specific methods

## RNA Subclass

Similarly, we can create an RNA subclass with RNA-specific features:

```python
class RNASequence(BiologicalSequence):
    def __init__(self, seq):
        # Call parent constructor then add RNA validation
        super().__init__(seq)
        self._check()

    def _check(self):
        """Validate RNA sequence contains only A,C,G,U"""
        if not all(b in "ACGU" for b in self.seq):
            raise InvalidSequenceError(f"Invalid RNA character in {self.seq}")
    
    def reverse_transcription(self):
        """Convert RNA to DNA sequence by replacing U with T"""
        return self.seq.replace('U', 'T')
```

The RNA class:
- Inherits all methods from BiologicalSequence
- Has RNA-specific validation (allows U but not T)
- Adds a reverse_transcription method

## Testing Our Inheritance Structure

Let's test that our DNA and RNA classes work correctly and understand what they inherit:

```python
# Test DNA class
dna = DNASequence("ATGCGTA")
print(f"DNA sequence: {dna}")  # Uses __str__ from parent
print(f"DNA length: {dna.length()}")  # Uses length() from parent  
print(f"DNA GC content: {dna.gc():.2f}")  # Uses gc() from parent
print(f"DNA complement: {dna.complement()}")  # Uses DNA-specific method
print(f"DNA reverse complement: {dna.reverse_complement()}")  # DNA-specific method

# Test RNA class
rna = RNASequence("AUGCCUA")
print(f"RNA sequence: {rna}")  # Uses __str__ from parent
print(f"RNA length: {rna.length()}")  # Uses length() from parent
print(f"RNA GC content: {rna.gc():.2f}")  # Uses gc() from parent
print(f"DNA after reverse transcription: {rna.reverse_transcription()}")  # RNA-specific method

# This will raise an error due to invalid character:
# bad_dna = DNASequence("ATGCX")
```

## Gene Inheritance Example

Let's look at a more complex example with genes. We'll create a general Gene class and then a specialized ProteinCodingGene subclass.

```python
class Gene:
    def __init__(self, locus, seq):
        self.locus = locus  # Gene identifier
        self.seq = seq.upper()  # Gene sequence
        self._check()  # Basic validation

    def _check(self):
        """Basic gene validation"""
        if not self.seq:
            raise ValueError("Gene sequence cannot be empty")

    def __str__(self):
        return f"Gene {self.locus}: {self.seq}"
```

Now let's create a protein-coding gene subclass that adds additional requirements:

```python
class ProteinCodingGene(Gene):
    def __init__(self, locus, seq, transcript_id):
        # Call parent constructor to set locus and seq
        super().__init__(locus, seq)
        # Add protein-coding specific attribute
        self.transcript_id = transcript_id
        # Add protein-coding specific validation
        self._check()

    def _check(self):
        """Extend parent validation with start codon check"""
        # First call the parent's _check method
        super()._check()
        # Then add protein-coding specific check
        if not self.seq.startswith("ATG"):
            raise ValueError("Protein-coding gene must start with ATG")
    
    def translate(self):
        """Simple translation using standard genetic code"""
        codon_table = {
            'ATG': 'M', 'TTT': 'F', 'TTC': 'F', 'TTA': 'L', 'TTG': 'L',
            'TCT': 'S', 'TCC': 'S', 'TCA': 'S', 'TCG': 'S',
            'TAT': 'Y', 'TAC': 'Y', 'TAA': '*', 'TAG': '*', 'TGA': '*',
        }
        protein = []
        for i in range(0, len(self.seq)-2, 3):
            codon = self.seq[i:i+3]
            protein.append(codon_table.get(codon, 'X'))  # X for unknown codons
        return ''.join(protein)
```

Key points about ProteinCodingGene:
- It inherits all attributes and methods from Gene
- `super().__init__(locus, seq)` reuses the parent constructor
- `super()._check()` calls the parent's validation before adding new checks
- It adds a new attribute (`transcript_id`) and new method (`translate()`)

## Using Mixins for Flexible Functionality

Mixins are special classes that provide additional functionality that can be added to multiple different classes. They're like "feature packages" that you can mix into your classes.

### Kmer Mixin

```python
class KmerMixin:
    def kmers(self, k):
        """Generate all k-mers of length k from sequence"""
        return [self.seq[i:i+k] for i in range(len(self.seq)-k+1)]
    
    def gc_window(self, window_size=100):
        """Calculate GC content in sliding windows"""
        windows = []
        for i in range(0, len(self.seq)-window_size+1):
            window_seq = self.seq[i:i+window_size]
            gc_count = sum(b in "GC" for b in window_seq)
            windows.append(gc_count/window_size)
        return windows
```

### FASTA Reading Mixin

```python
class FASTAReadMixin:
    @classmethod
    def from_fasta(cls, fasta_text):
        """Create sequence object from FASTA format text"""
        lines = fasta_text.splitlines()
        # Extract sequence (ignore header lines starting with >)
        seq = "".join(line for line in lines if not line.startswith(">"))
        # Create new instance of whatever class this is mixed into
        return cls(seq)
```

## Combining Classes with Multiple Inheritance

We can now create specialized classes by combining our sequence classes with mixins:

```python
class DNAWithKmers(DNASequence, KmerMixin):
    """DNA sequence with k-mer functionality"""
    pass  # Inherits everything from both parents

class RNAViralSequence(RNASequence, FASTAReadMixin):
    """RNA sequence that can be loaded from FASTA"""
    pass
```

These combined classes get all the functionality from both parent classes without us having to rewrite any code.

## Testing Combined Functionality

Let's test our combined classes to see how they work:

```python
# DNA with k-mer functionality
dna_kmers = DNAWithKmers("ATGCGATCGAT")
print(f"DNA sequence: {dna_kmers}")
print(f"3-mers: {dna_kmers.kmers(3)}")  # From KmerMixin
print(f"Complement: {dna_kmers.complement()}")  # From DNASequence

# RNA that can load from FASTA
fasta_data = """>rna_sequence
AUGCCUAAUGC"""
rna_viral = RNAViralSequence.from_fasta(fasta_data)  # From FASTAReadMixin
print(f"Loaded RNA: {rna_viral}")
print(f"Reverse transcription: {rna_viral.reverse_transcription()}")  # From RNASequence
```

## Composition: An Alternative to Inheritance

Sometimes, instead of using inheritance ("is-a" relationship), it's better to use composition ("has-a" relationship). Composition means one object contains another object as part of its data.

```python
class Genome:
    def __init__(self, name):
        self.name = name
        self.genes = []  # Genome HAS genes (composition)

    def add_gene(self, gene):
        """Add a gene to the genome"""
        self.genes.append(gene)

    def total_length(self):
        """Total length of all gene sequences"""
        return sum(len(gene.seq) for gene in self.genes)
    
    def get_genes_by_type(self, gene_type):
        """Filter genes by type"""
        return [gene for gene in self.genes if isinstance(gene, gene_type)]
```

This Genome class doesn't inherit from anything - it simply contains genes as part of its data structure.

## Exercises

> ## Exercise 1: Create an RNA Subclass with Reverse Transcription
>
> Create an RNA subclass that provides a method to return a DNASequence object representing the reverse transcribed DNA.
>
>> ## Solution
>>
>> ~~~
>> class RNASequenceWithRT(RNASequence):
>>     def reverse_transcribe(self):
>>         dna_seq = self.seq.replace('U', 'T')
>>         return DNASequence(dna_seq)
>> 
>> # Test
>> rna = RNASequenceWithRT("AUGCCUAA")
>> dna = rna.reverse_transcribe()
>> print(f"DNA: {dna}")
>> print(f"Type: {type(dna)}")
>> ~~~
>> {: .language-python}
> {: .solution}
{: .challenge}

> ## Exercise 2: Extend ProteinCodingGene with Complete Translation
>
> Complete the translation method in ProteinCodingGene using a proper codon table and handle stop codons appropriately.
>
>> ## Solution
>>
>> ~~~
>> class ProteinCodingGene(Gene):
>>     def translate(self):
>>         codon_table = {
>>             'ATG': 'M', 
>>             'TTT': 'F', 'TTC': 'F', 
>>             'TTA': 'L', 'TTG': 'L', 'CTT': 'L', 'CTC': 'L', 'CTA': 'L', 'CTG': 'L',
>>             'TCT': 'S', 'TCC': 'S', 'TCA': 'S', 'TCG': 'S', 'AGT': 'S', 'AGC': 'S',
>>             'TAT': 'Y', 'TAC': 'Y',
>>             'TAA': '*', 'TAG': '*', 'TGA': '*',  # Stop codons
>>         }
>>         protein = []
>>         for i in range(0, len(self.seq)-2, 3):
>>             codon = self.seq[i:i+3]
>>             if codon in codon_table:
>>                 aa = codon_table[codon]
>>                 if aa == '*':  # Stop codon
>>                     break
>>                 protein.append(aa)
>>             else:
>>                 protein.append('X')  # Unknown codon
>>         return ''.join(protein)
>> 
>> # Test
>> gene = ProteinCodingGene("test_gene", "ATGGGATGA", "transcript1")
>> print(f"Protein: {gene.translate()}")
>> ~~~
>> {: .language-python}
> {: .solution}
{: .challenge}

> ## Exercise 3: Combine FASTA Reading with DNA Sequence
>
> Create a class that can load DNA sequences directly from FASTA format by combining DNASequence with FASTAReadMixin.
>
>> ## Solution
>>
>> ~~~
>> class DNAFromFASTA(DNASequence, FASTAReadMixin):
>>     pass
>> 
>> # Test
>> fasta_data = """>dna_sequence
>> ATGCGATCGATCG"""
>> dna = DNAFromFASTA.from_fasta(fasta_data)
>> print(f"Loaded DNA: {dna}")
>> print(f"Complement: {dna.complement()}")
>> ~~~
>> {: .language-python}
> {: .solution}
{: .challenge}

> ## Exercise 4: Add GC Window Method Using Mixin
>
> Use the KmerMixin to add GC window analysis to DNA sequences and test it with a longer sequence.
>
>> ## Solution
>>
>> ~~~
>> class DNAWithGCAnalysis(DNASequence, KmerMixin):
>>     pass
>> 
>> # Test
>> dna_gc = DNAWithGCAnalysis("ATGC" * 25)
>> windows = dna_gc.gc_window(window_size=10)
>> print(f"GC windows: {windows[:5]}")  # First 5 windows
>> ~~~
>> {: .language-python}
> {: .solution}
{: .challenge}

{% include links.md %}
