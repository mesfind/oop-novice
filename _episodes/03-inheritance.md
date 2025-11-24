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
---

Inheritance supports shared behaviour in biological sequence classes. This reduces repetition when building DNA, RNA, or protein objects. The following examples use short code blocks to keep focus on core patterns.

## Why Inheritance Matters in Bioinformatics

In bioinformatics, we often work with different types of biological sequences that share common characteristics but also have unique properties. For example, DNA, RNA, and protein sequences all have:
- A sequence string
- A length
- GC content calculation
- Validation rules

But each also has specific requirements:
- DNA has complement rules
- RNA has uracil instead of thymine
- Proteins have amino acid alphabets

Without inheritance, we'd duplicate code across these sequence types. With inheritance, we can create a hierarchy that shares common functionality while allowing specialization.

## Basic Inheritance Diagram

```
BiologicalSequence (parent class)
    ↓
DNASequence (child class)
    ↓
RNASequence (child class)
```

## Custom Exception for Sequence Validation

```python
class InvalidSequenceError(ValueError):
    """Raised when a sequence contains invalid characters"""
    pass
```

## Parent Class: BiologicalSequence

```python
class BiologicalSequence:
    def __init__(self, seq):
        self.seq = seq.upper()

    def length(self):
        return len(self.seq)

    def gc(self):
        """Calculate GC content as fraction of sequence"""
        return sum(b in "GC" for b in self.seq) / len(self.seq)
    
    def __str__(self):
        return self.seq
```

> ## Testing the Parent Class
>
> Let's create a basic biological sequence and test its methods:
>
> ```python
> generic_seq = BiologicalSequence("atgc")
> print(f"Sequence: {generic_seq}")
> print(f"Length: {generic_seq.length()}")
> print(f"GC content: {generic_seq.gc():.2f}")
> ```
>
> This works, but we need more specific sequence types for real bioinformatics work.
{: .callout}

## DNA Subclass with Validation

```python
class DNASequence(BiologicalSequence):
    def __init__(self, seq):
        super().__init__(seq)  # Call parent constructor
        self._check()          # Add DNA-specific validation

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

## RNA Subclass

```python
class RNASequence(BiologicalSequence):
    def __init__(self, seq):
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

> ## Testing DNA and RNA Classes
>
> ```python
> # DNA example
> dna = DNASequence("ATGCGTA")
> print(f"DNA: {dna}")
> print(f"Complement: {dna.complement()}")
> print(f"Reverse complement: {dna.reverse_complement()}")
> 
> # RNA example  
> rna = RNASequence("AUGCCUA")
> print(f"RNA: {rna}")
> print(f"DNA after reverse transcription: {rna.reverse_transcription()}")
> 
> # This will raise an error:
> bad_dna = DNASequence("ATGCX")  # Invalid character
> ```
{: .callout}

## Gene Hierarchy Example

Let's create a more complex example with genes and protein-coding genes:

```python
class Gene:
    def __init__(self, locus, seq):
        self.locus = locus
        self.seq = seq.upper()
        self._check()

    def _check(self):
        """Basic gene validation"""
        if not self.seq:
            raise ValueError("Gene sequence cannot be empty")

    def __str__(self):
        return f"Gene {self.locus}: {self.seq}"
```

```python
class ProteinCodingGene(Gene):
    def __init__(self, locus, seq, transcript_id):
        super().__init__(locus, seq)  # Reuse parent initialization
        self.transcript_id = transcript_id
        self._check()  # Add protein-coding specific validation

    def _check(self):
        """Extend parent validation with start codon check"""
        super()._check()  # First do basic gene validation
        if not self.seq.startswith("ATG"):
            raise ValueError("Protein-coding gene must start with ATG")
    
    def translate(self):
        """Simple translation using standard genetic code"""
        codon_table = {
            'ATG': 'M', 'TTT': 'F', 'TTC': 'F', 'TTA': 'L', 'TTG': 'L',
            # ... more codons would go here in real implementation
        }
        protein = []
        for i in range(0, len(self.seq)-2, 3):
            codon = self.seq[i:i+3]
            protein.append(codon_table.get(codon, 'X'))  # X for unknown
        return ''.join(protein)
```

## Mixins for Modular Behaviour

Mixins allow us to add functionality to multiple classes without deep inheritance hierarchies:

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

```python
class FASTAReadMixin:
    @classmethod
    def from_fasta(cls, fasta_text):
        """Create sequence object from FASTA format text"""
        lines = fasta_text.splitlines()
        seq = "".join(line for line in lines if not line.startswith(">"))
        return cls(seq)  # Create new instance of whatever class this is mixed into
```

## Combining Classes with Multiple Inheritance

```python
class DNAWithKmers(DNASequence, KmerMixin):
    """DNA sequence with k-mer functionality"""
    pass  # Gets all methods from both parents

class RNAViralSequence(RNASequence, FASTAReadMixin):
    """RNA sequence that can be loaded from FASTA"""
    pass
```

> ## Testing Combined Classes
>
> ```python
> # DNA with k-mers
> dna_kmers = DNAWithKmers("ATGCGATCGAT")
> print(f"3-mers: {dna_kmers.kmers(3)}")
> print(f"GC windows: {dna_kmers.gc_window(5)}")
> 
> # RNA from FASTA
> fasta_data = """>rna_sequence
> AUGCCUAAUGC"""
> rna_viral = RNAViralSequence.from_fasta(fasta_data)
> print(f"Loaded RNA: {rna_viral}")
> ```
{: .callout}

## Composition as an Alternative

Sometimes composition (has-a relationship) is better than inheritance (is-a relationship):

```python
class Genome:
    def __init__(self, name):
        self.name = name
        self.genes = []  # Composition: genome has genes

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

> ## Testing Composition
>
> ```python
> genome = Genome("test_genome")
> genome.add_gene(Gene("gene1", "ATGCATGC"))
> genome.add_gene(ProteinCodingGene("gene2", "ATGGGATGA", "transcript1"))
> 
> print(f"Total genome length: {genome.total_length()}")
> print(f"Protein coding genes: {genome.get_genes_by_type(ProteinCodingGene)}")
> ```
{: .callout}

## Exercises

### Exercise 1: Create an RNA Subclass with Reverse Transcription

Extend the `RNASequence` class with a method that returns a `DNASequence` object representing the reverse transcribed DNA.

```python
# Your code here
```

> ## Solution
> ```python
> class RNASequenceWithRT(RNASequence):
>     def reverse_transcribe(self):
>         dna_seq = self.seq.replace('U', 'T')
>         return DNASequence(dna_seq)
> 
> # Test
> rna = RNASequenceWithRT("AUGCCUAA")
> dna = rna.reverse_transcribe()
> print(f"DNA: {dna}")
> print(f"Type: {type(dna)}")
> ```
{: .solution}

### Exercise 2: Extend ProteinCodingGene with Translation

Complete the translation method in `ProteinCodingGene` using a proper codon table and handle stop codons.

```python
# Your code here
```

> ## Solution
> ```python
> class ProteinCodingGene(Gene):
>     # ... existing code ...
>     
>     def translate(self):
>         codon_table = {
>             'ATG': 'M', 
>             'TTT': 'F', 'TTC': 'F', 
>             'TTA': 'L', 'TTG': 'L', 'CTT': 'L', 'CTC': 'L', 'CTA': 'L', 'CTG': 'L',
>             'TCT': 'S', 'TCC': 'S', 'TCA': 'S', 'TCG': 'S',
>             'TAT': 'Y', 'TAC': 'Y',
>             'TAA': '*', 'TAG': '*', 'TGA': '*',  # Stop codons
>             # ... more codons ...
>         }
>         protein = []
>         for i in range(0, len(self.seq)-2, 3):
>             codon = self.seq[i:i+3]
>             if codon in codon_table:
>                 aa = codon_table[codon]
>                 if aa == '*':  # Stop codon
>                     break
>                 protein.append(aa)
>             else:
>                 protein.append('X')  # Unknown codon
>         return ''.join(protein)
> 
> # Test
> gene = ProteinCodingGene("test", "ATGGGATGA", "t1")
> print(f"Protein: {gene.translate()}")
> ```
{: .solution}

### Exercise 3: Combine FASTA Reading with DNA Sequence

Create a class that can load DNA sequences directly from FASTA format.

```python
# Your code here
```

> ## Solution
> ```python
> class DNAFromFASTA(DNASequence, FASTAReadMixin):
>     pass
> 
> # Test
> fasta_data = """>dna_sequence
> ATGCGATCGATCG"""
> dna = DNAFromFASTA.from_fasta(fasta_data)
> print(f"Loaded DNA: {dna}")
> print(f"Complement: {dna.complement()}")
> ```
{: .solution}

### Exercise 4: Add GC Window Method Using Mixin

Use the `KmerMixin` to add GC window analysis to DNA sequences and test it.

```python
# Your code here
```

> ## Solution
> ```python
> class DNAWithGCAnalysis(DNASequence, KmerMixin):
>     pass
> 
> # Test
> dna_gc = DNAWithGCAnalysis("ATGC" * 25)  # Make a longer sequence
> windows = dna_gc.gc_window(window_size=10)
> print(f"GC windows: {windows[:5]}")  # First 5 windows
> ```
{: .solution}

Understanding inheritance helps create organized, reusable bioinformatics code that models biological relationships naturally while avoiding duplication.

{% include links.md %}
