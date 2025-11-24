---
title: "Special methods in Bioinformatics"
teaching: 25
exercises: 15
questions:
- "How can classes allow their instances to work with standard Python operators in bioinformatics?"
- "How can classes allow their instances to behave like iterables or collections of sequences?"
- "How can classes allow their instances to be called like functions?"
objectives:
- "Be able to implement methods like `__add__`, `__eq__`, and `__gt__` for bioinformatics objects."
- "Be able to implement methods like `__len__`, `__iter__`, and `__reversed__` for collections of biological data."
- "Be able to implement the `__call__` method for callable bioinformatics objects."
keypoints:
- "Implement methods like `__eq__`, `__add__`, and `__gt__` to allow operations such as arithmetic and comparisons on bioinformatics objects."
- "Implement `__repr__` to get meaningful printouts for sequences and other objects."
- "Implement methods like `__len__`, `__iter__`, and `__reversed__` to make classes behave like collections."
- "Implement `__call__` to allow classes to be used as callable bioinformatics functions."
- **Comparison methods** (`__eq__`, `__lt__`, etc.) allow meaningful comparisons of biological objects
- "**Arithmetic methods** (`__add__`, `__mul__`, etc.) enable natural operations on sequences and measurements"
- "**Collection methods** (`__len__`, `__getitem__`, `__iter__`) make objects behave like built-in containers"
- "**The `__call__` method** turns instances into callable functions, perfect for configurable analysis tools"
- "**String representation** (`__repr__`, `__str__`) provides meaningful output for debugging and display"

---

## Introduction to Special Methods

Python's special methods (also called "dunder" methods for double underscores) allow you to customize how your objects behave with Python's built-in operations. In bioinformatics, this is incredibly useful for creating intuitive APIs that work naturally with biological data.

Let's explore how these special methods can make our bioinformatics code more powerful and expressive.

## Comparison Methods for Biological Objects

By default, Python objects are only considered equal if they are the exact same object in memory. For biological data, we often want more meaningful comparisons.

```python
class Gene:
    def __init__(self, name, sequence):
        self.name = name
        self.sequence = sequence

# Default behavior - identity comparison
gene1 = Gene("geneA", "ATGCGT")
gene2 = Gene("geneA", "ATGCGT")
print(f"Default equality: {gene1 == gene2}")  # False - different objects
```

> ## Challenge: Gene equality by sequence
>
> Implement a `Gene` class that supports equality comparisons (`==`) based on sequence content rather than object identity. Two genes should be considered equal if they have the same sequence, regardless of their names.
>
>> ## Solution
>>
>> ```python
>> class Gene:
>>     def __init__(self, name, sequence):
>>         self.name = name
>>         self.sequence = sequence.upper()
>>
>>     def __eq__(self, other):
>>         """Define equality based on sequence content"""
>>         if not isinstance(other, Gene):
>>             return False
>>         return self.sequence == other.sequence
>>
>>     def __repr__(self):
>>         """Provide a meaningful string representation"""
>>         return f"Gene('{self.name}', '{self.sequence}')"
>>
>> # Testing
>> gene1 = Gene("geneA", "ATGCGT")
>> gene2 = Gene("geneB", "ATGCGT")  # Different name, same sequence
>> gene3 = Gene("geneA", "ATGCGA")  # Same name, different sequence
>>
>> print(f"Same sequence: {gene1 == gene2}")  # True
>> print(f"Different sequence: {gene1 == gene3}")  # False
>> print(f"String representation: {gene1}")
>> ```
>> {: .language-python}
>>
>> ```
>> Same sequence: True
>> Different sequence: False
>> String representation: Gene('geneA', 'ATGCGT')
>> ```
>> {: .output}
> {: .solution}
{: .challenge}

## Advanced Comparison Methods

We can implement other comparison methods to enable sorting and rich comparisons of biological objects.

```python
class Gene:
    def __init__(self, name, sequence):
        self.name = name
        self.sequence = sequence.upper()
    
    def gc_content(self):
        """Calculate GC content as a percentage"""
        gc_count = sum(1 for base in self.sequence if base in "GC")
        return gc_count / len(self.sequence) * 100
    
    def __eq__(self, other):
        if not isinstance(other, Gene):
            return False
        return self.sequence == other.sequence
    
    def __lt__(self, other):
        """Less than - compare by GC content"""
        return self.gc_content() < other.gc_content()
    
    def __gt__(self, other):
        """Greater than - compare by GC content"""
        return self.gc_content() > other.gc_content()
    
    def __le__(self, other):
        """Less than or equal"""
        return self.gc_content() <= other.gc_content()
    
    def __ge__(self, other):
        """Greater than or equal"""
        return self.gc_content() >= other.gc_content()
    
    def __repr__(self):
        return f"Gene('{self.name}', GC: {self.gc_content():.1f}%)"
```

> ## Challenge: Compare genes by GC content
>
> Implement `__lt__` and `__gt__` for a `Gene` class so genes can be sorted by GC content. Also implement the other comparison methods for completeness.
>
>> ## Solution
>>
>> ```python
>> class Gene:
>>     def __init__(self, name, sequence):
>>         self.name = name
>>         self.sequence = sequence.upper()
>>
>>     def gc_content(self):
>>         gc_count = sum(1 for base in self.sequence if base in "GC")
>>         return gc_count / len(self.sequence) * 100
>>
>>     def __lt__(self, other):
>>         return self.gc_content() < other.gc_content()
>>
>>     def __gt__(self, other):
>>         return self.gc_content() > other.gc_content()
>>
>>     def __le__(self, other):
>>         return self.gc_content() <= other.gc_content()
>>
>>     def __ge__(self, other):
>>         return self.gc_content() >= other.gc_content()
>>
>>     def __eq__(self, other):
>>         return self.sequence == other.sequence
>>
>>     def __repr__(self):
>>         return f"Gene('{self.name}', GC: {self.gc_content():.1f}%)"
>>
>> # Example usage
>> gene_list = [
>>     Gene("gene1", "ATGC"),      # 50% GC
>>     Gene("gene2", "GCGC"),      # 100% GC  
>>     Gene("gene3", "ATAT"),      # 0% GC
>>     Gene("gene4", "ATGCCG")     # 66.7% GC
>> ]
>>
>> print("Sorted by GC content:")
>> for gene in sorted(gene_list):
>>     print(f"  {gene}")
>>
>> print(f"\nGene2 > Gene1: {gene_list[1] > gene_list[0]}")
>> print(f"Gene3 < Gene4: {gene_list[2] < gene_list[3]}")
>> ```
>> {: .language-python}
>>
>> ```
>> Sorted by GC content:
>>   Gene('gene3', GC: 0.0%)
>>   Gene('gene1', GC: 50.0%)
>>   Gene('gene4', GC: 66.7%)
>>   Gene('gene2', GC: 100.0%)
>>
>> Gene2 > Gene1: True
>> Gene3 < Gene4: True
>> ```
>> {: .output}
> {: .solution}
{: .challenge}

## Arithmetic Operations for Bioinformatics

We can define mathematical operations for biological objects, like concatenating sequences or combining measurements with error propagation.

```python
class DNASequence:
    def __init__(self, sequence):
        self.sequence = sequence.upper()
    
    def __add__(self, other):
        """Concatenate two DNA sequences"""
        if not isinstance(other, DNASequence):
            raise TypeError("Can only add DNASequence objects")
        return DNASequence(self.sequence + other.sequence)
    
    def __mul__(self, n):
        """Repeat sequence n times"""
        if not isinstance(n, int):
            raise TypeError("Can only multiply by integer")
        return DNASequence(self.sequence * n)
    
    def __len__(self):
        """Return length of sequence"""
        return len(self.sequence)
    
    def __repr__(self):
        return f"DNASequence('{self.sequence}')"
```

> ## Challenge: Sequence arithmetic
>
> Implement a `SequenceWithError` class to store a biological sequence length with measurement error, and implement `__add__`, `__sub__`, `__mul__`, and `__truediv__` to propagate errors according to error propagation rules.
>
>> ## Solution
>>
>> ```python
>> import math
>>
>> class SequenceWithError:
>>     def __init__(self, length, error):
>>         self.length = length
>>         self.error = error
>>
>>     def __repr__(self):
>>         return f"SequenceWithError({self.length} ± {self.error})"
>>
>>     def __add__(self, other):
>>         """Add lengths - errors add in quadrature"""
>>         new_length = self.length + other.length
>>         new_error = math.sqrt(self.error**2 + other.error**2)
>>         return SequenceWithError(new_length, new_error)
>>
>>     def __sub__(self, other):
>>         """Subtract lengths - errors add in quadrature"""
>>         new_length = self.length - other.length
>>         new_error = math.sqrt(self.error**2 + other.error**2)
>>         return SequenceWithError(new_length, new_error)
>>
>>     def __mul__(self, other):
>>         """Multiply lengths - relative errors add in quadrature"""
>>         new_length = self.length * other.length
>>         rel_error = math.sqrt((self.error/self.length)**2 + (other.error/other.length)**2)
>>         new_error = new_length * rel_error
>>         return SequenceWithError(new_length, new_error)
>>
>>     def __truediv__(self, other):
>>         """Divide lengths - relative errors add in quadrature"""
>>         new_length = self.length / other.length
>>         rel_error = math.sqrt((self.error/self.length)**2 + (other.error/other.length)**2)
>>         new_error = new_length * rel_error
>>         return SequenceWithError(new_length, new_error)
>>
>> # Example usage
>> seq1 = SequenceWithError(100, 2)   # 100bp ± 2bp
>> seq2 = SequenceWithError(150, 3)   # 150bp ± 3bp
>>
>> print(f"Sum: {seq1 + seq2}")
>> print(f"Difference: {seq2 - seq1}")
>> print(f"Product: {seq1 * seq2}")
>> print(f"Ratio: {seq2 / seq1}")
>> ```
>> {: .language-python}
>>
>> ```
>> Sum: SequenceWithError(250 ± 3.605551275463989)
>> Difference: SequenceWithError(50 ± 3.605551275463989)
>> Product: SequenceWithError(15000 ± 390.5124837945881)
>> Ratio: SequenceWithError(1.5 ± 0.05408326913195984)
>> ```
>> {: .output}
> {: .solution}
{: .challenge}

## Making Objects Callable

The `__call__` method allows instances to be called like functions, which is useful for creating configurable bioinformatics tools.

```python
class SequenceAnalyzer:
    def __init__(self, analysis_type="gc"):
        self.analysis_type = analysis_type
    
    def __call__(self, sequence):
        """Make the analyzer callable with a sequence"""
        if self.analysis_type == "gc":
            gc_count = sum(1 for base in sequence if base in "GC")
            return gc_count / len(sequence)
        elif self.analysis_type == "at":
            at_count = sum(1 for base in sequence if base in "AT")
            return at_count / len(sequence)
        else:
            raise ValueError(f"Unknown analysis type: {self.analysis_type}")

# Usage
gc_analyzer = SequenceAnalyzer("gc")
at_analyzer = SequenceAnalyzer("at")

sequence = "ATGCGCTAGCT"
print(f"GC content: {gc_analyzer(sequence):.2%}")
print(f"AT content: {at_analyzer(sequence):.2%}")
```

> ## Challenge: Callable sequence plotter
>
> Create a `SequencePlotter` class that plots GC content along a sequence using a sliding window when called as a function. The class should be configurable for different window sizes.
>
>> ## Solution
>>
>> ```python
>> import matplotlib.pyplot as plt
>>
>> class SequencePlotter:
>>     def __init__(self, window_size=10):
>>         self.window_size = window_size
>>
>>     def calculate_gc_sliding(self, sequence):
>>         """Calculate GC content in sliding windows"""
>>         sequence = sequence.upper()
>>         gc_values = []
>>         positions = []
>>         
>>         for i in range(len(sequence) - self.window_size + 1):
>>             window = sequence[i:i + self.window_size]
>>             gc_count = sum(1 for base in window if base in "GC")
>>             gc_content = gc_count / self.window_size
>>             gc_values.append(gc_content)
>>             positions.append(i + self.window_size // 2)
>>         
>>         return positions, gc_values
>>
>>     def __call__(self, sequence, title="GC Content Sliding Window"):
>>         """Make the plotter callable"""
>>         positions, gc_values = self.calculate_gc_sliding(sequence)
>>         
>>         plt.figure(figsize=(10, 4))
>>         plt.plot(positions, gc_values)
>>         plt.title(f"{title} (window={self.window_size}bp)")
>>         plt.xlabel("Position in sequence")
>>         plt.ylabel("GC Content")
>>         plt.grid(True, alpha=0.3)
>>         plt.ylim(0, 1)
>>         plt.tight_layout()
>>         plt.show()
>>         
>>         return gc_values
>>
>> # Example usage
>> plotter_small = SequencePlotter(window_size=5)
>> plotter_large = SequencePlotter(window_size=20)
>>
>> test_sequence = "ATGCGCTAGCTAGCTAGCTAGCTAGCTAGCTAGCTAGC" * 5
>> 
>> # Use as callable objects
>> print("Small window analysis:")
>> gc_small = plotter_small(test_sequence, "Small Window GC Analysis")
>> 
>> print("Large window analysis:")  
>> gc_large = plotter_large(test_sequence, "Large Window GC Analysis")
>> ```
>> {: .language-python}
> {: .solution}
{: .challenge}

## Creating Collection-like Behavior

We can make our bioinformatics objects behave like collections, allowing iteration, indexing, and other collection operations.

```python
class Genome:
    """A collection of genes that behaves like a sequence"""
    
    def __init__(self, name, genes):
        self.name = name
        self.genes = list(genes)
    
    def __len__(self):
        """Return number of genes in genome"""
        return len(self.genes)
    
    def __getitem__(self, index):
        """Access genes by index or slice"""
        if isinstance(index, slice):
            return Genome(f"{self.name}_slice", self.genes[index])
        return self.genes[index]
    
    def __iter__(self):
        """Iterate over genes"""
        return iter(self.genes)
    
    def __contains__(self, gene):
        """Check if gene is in genome"""
        return gene in self.genes
    
    def __repr__(self):
        return f"Genome('{self.name}', {len(self)} genes)"
```

> ## Challenge: Iterable genome
>
> Implement a `Genome` class that allows iterating over genes using `__iter__`, accessing them by index with `__getitem__`, and getting the length with `__len__`. Also support slicing to create sub-genomes.
>
>> ## Solution
>>
>> ```python
>> class Genome:
>>     def __init__(self, name, genes):
>>         self.name = name
>>         self.genes = list(genes)
>>
>>     def __iter__(self):
>>         """Allow iteration over genes"""
>>         return iter(self.genes)
>>
>>     def __getitem__(self, index):
>>         """Access genes by index or slice"""
>>         if isinstance(index, slice):
>>             # Return a new Genome for slices
>>             return Genome(f"{self.name}_slice", self.genes[index])
>>         return self.genes[index]
>>
>>     def __len__(self):
>>         """Return number of genes"""
>>         return len(self.genes)
>>
>>     def __contains__(self, gene):
>>         """Check if gene is in genome"""
>>         return gene in self.genes
>>
>>     def __repr__(self):
>>         return f"Genome('{self.name}', {len(self)} genes)"
>>
>> # Example usage
>> genes = [
>>     Gene("gene1", "ATGCGT"),
>>     Gene("gene2", "GCGCAT"), 
>>     Gene("gene3", "ATATAT"),
>>     Gene("gene4", "GGGCCC")
>> ]
>>
>> genome = Genome("TestGenome", genes)
>>
>> print(f"Genome: {genome}")
>> print(f"Number of genes: {len(genome)}")
>> print(f"First gene: {genome[0]}")
>> print(f"Gene2 in genome: {genes[1] in genome}")
>>
>> print("\nIterating over genes:")
>> for gene in genome:
>>     print(f"  {gene}")
>>
>> print("\nSlicing:")
>> sub_genome = genome[1:3]
>> print(f"Sub-genome: {sub_genome}")
>> for gene in sub_genome:
>>     print(f"  {gene}")
>> ```
>> {: .language-python}
>>
>> ```
>> Genome: Genome('TestGenome', 4 genes)
>> Number of genes: 4
>> First gene: Gene('gene1', GC: 50.0%)
>> Gene2 in genome: True
>>
>> Iterating over genes:
>>   Gene('gene1', GC: 50.0%)
>>   Gene('gene2', GC: 66.7%)
>>   Gene('gene3', GC: 0.0%)
>>   Gene('gene4', GC: 100.0%)
>>
>> Slicing:
>> Sub-genome: Genome('TestGenome_slice', 2 genes)
>>   Gene('gene2', GC: 66.7%)
>>   Gene('gene3', GC: 0.0%)
>> ```
>> {: .output}
> {: .solution}
{: .challenge}

> ## Challenge: Reverse iteration
>
> Implement `__reversed__` for the `Genome` class so that `reversed(genome)` yields genes in reverse order.
>
>> ## Solution
>>
>> ```python
>> class Genome:
>>     # ... previous methods ...
>>     
>>     def __reversed__(self):
>>         """Allow reverse iteration over genes"""
>>         return reversed(self.genes)
>>
>> # Example usage
>> genome = Genome("TestGenome", [
>>     Gene("gene1", "ATGC"),
>>     Gene("gene2", "GCGC"), 
>>     Gene("gene3", "ATAT")
>> ])
>>
>> print("Forward iteration:")
>> for gene in genome:
>>     print(f"  {gene}")
>>
>> print("\nReverse iteration:")
>> for gene in reversed(genome):
>>     print(f"  {gene}")
>> ```
>> {: .language-python}
>>
>> ```
>> Forward iteration:
>>   Gene('gene1', GC: 50.0%)
>>   Gene('gene2', GC: 100.0%)
>>   Gene('gene3', GC: 0.0%)
>>
>> Reverse iteration:
>>   Gene('gene3', GC: 0.0%)
>>   Gene('gene2', GC: 100.0%)
>>   Gene('gene1', GC: 50.0%)
>> ```
>> {: .output}
> {: .solution}
{: .challenge}

## Advanced Sequence Operations

Let's create a more comprehensive DNA sequence class with rich operations.

```python
class DNASequence:
    def __init__(self, sequence):
        self.sequence = sequence.upper()
        self._validate()
    
    def _validate(self):
        """Validate DNA sequence"""
        valid_bases = set("ACGT")
        if not all(base in valid_bases for base in self.sequence):
            invalid = set(self.sequence) - valid_bases
            raise ValueError(f"Invalid DNA bases: {invalid}")
    
    def __add__(self, other):
        """Concatenate sequences"""
        return DNASequence(self.sequence + other.sequence)
    
    def __mul__(self, n):
        """Repeat sequence"""
        return DNASequence(self.sequence * n)
    
    def __getitem__(self, index):
        """Support indexing and slicing"""
        if isinstance(index, slice):
            return DNASequence(self.sequence[index])
        return self.sequence[index]
    
    def __len__(self):
        return len(self.sequence)
    
    def __eq__(self, other):
        return self.sequence == other.sequence
    
    def __lt__(self, other):
        """Compare by length"""
        return len(self) < len(other)
    
    def __contains__(self, motif):
        """Check if motif is in sequence"""
        return motif in self.sequence
    
    def __repr__(self):
        return f"DNASequence('{self.sequence}')"
    
    def gc_content(self):
        gc = sum(1 for base in self.sequence if base in "GC")
        return gc / len(self.sequence)
```

> ## Challenge: Complement arithmetic
>
> Implement a `DNASequence` class that supports `+` for concatenation and `*` to repeat the sequence N times. Include `__repr__` and also add methods for complement and reverse complement operations.
>
>> ## Solution
>>
>> ```python
>> class DNASequence:
>>     def __init__(self, sequence):
>>         self.sequence = sequence.upper()
>>
>>     def __add__(self, other):
>>         """Concatenate two DNA sequences"""
>>         return DNASequence(self.sequence + other.sequence)
>>
>>     def __mul__(self, n):
>>         """Repeat sequence n times"""
>>         if not isinstance(n, int) or n < 0:
>>             raise ValueError("Can only multiply by positive integer")
>>         return DNASequence(self.sequence * n)
>>
>>     def complement(self):
>>         """Return complement sequence"""
>>         complement_map = str.maketrans("ACGT", "TGCA")
>>         return DNASequence(self.sequence.translate(complement_map))
>>
>>     def reverse_complement(self):
>>         """Return reverse complement sequence"""
>>         return self.complement()[::-1]
>>
>>     def __getitem__(self, index):
>>         """Support slicing"""
>>         if isinstance(index, slice):
>>             return DNASequence(self.sequence[index])
>>         return self.sequence[index]
>>
>>     def __repr__(self):
>>         return f"DNASequence('{self.sequence}')"
>>
>>     def __len__(self):
>>         return len(self.sequence)
>>
>> # Testing
>> a = DNASequence("ATG")
>> b = DNASequence("CGT")
>> 
>> print(f"Concatenation: {a + b}")
>> print(f"Repetition: {a * 3}")
>> print(f"Complement: {a.complement()}")
>> print(f"Reverse complement: {a.reverse_complement()}")
>> print(f"Slice: {DNASequence('ATGCGTA')[2:5]}")
>> ```
>> {: .language-python}
>>
>> ```
>> Concatenation: DNASequence('ATGCGT')
>> Repetition: DNASequence('ATGATGATG')
>> Complement: DNASequence('TAC')
>> Reverse complement: DNASequence('CAT')
>> Slice: DNASequence('GCG')
>> ```
>> {: .output}
> {: .solution}
{: .challenge}

> ## Challenge: Gene filtering with `__call__`
>
> Modify the `Gene` class so that instances can be called as a function returning `True` if a given motif is present in the sequence. Also support regular expression patterns for more complex motif searching.
>
>> ## Solution
>>
>> ```python
>> import re
>>
>> class Gene:
>>     def __init__(self, name, sequence):
>>         self.name = name
>>         self.sequence = sequence.upper()
>>
>>     def __call__(self, motif, use_regex=False):
>>         """Make gene callable for motif searching"""
>>         if use_regex:
>>             return bool(re.search(motif, self.sequence))
>>         else:
>>             return motif.upper() in self.sequence
>>
>>     def find_motif_positions(self, motif):
>>         """Find all positions where motif occurs"""
>>         positions = []
>>         start = 0
>>         motif = motif.upper()
>>         
>>         while True:
>>             pos = self.sequence.find(motif, start)
>>             if pos == -1:
>>                 break
>>             positions.append(pos)
>>             start = pos + 1
>>         
>>         return positions
>>
>>     def __repr__(self):
>>         return f"Gene('{self.name}', '{self.sequence}')"
>>
>> # Example usage
>> g = Gene("geneA", "ATGCGTATGCG")
>>
>> print(f"Contains 'GCG': {g('GCG')}")
>> print(f"Contains 'AAA': {g('AAA')}")
>> print(f"Positions of 'GCG': {g.find_motif_positions('GCG')}")
>>
>> # Test with regex
>> print(f"Regex pattern 'GC.T': {g('GC.T', use_regex=True)}")
>> ```
>> {: .language-python}
>>
>> ```
>> Contains 'GCG': True
>> Contains 'AAA': False
>> Positions of 'GCG': [2, 8]
>> Regex pattern 'GC.T': True
>> ```
>> {: .output}
> {: .solution}
{: .challenge}

> ## Challenge: Slice access for sequences
>
> Extend the `DNASequence` class to support slicing with `__getitem__`, returning a new `DNASequence` object. Also implement `__setitem__` for a mutable version.
>
>> ## Solution
>>
>> ```python
>> class DNASequence:
>>     def __init__(self, sequence):
>>         self.sequence = sequence.upper()
>>
>>     def __getitem__(self, key):
>>         """Support indexing and slicing"""
>>         if isinstance(key, slice):
>>             return DNASequence(self.sequence[key])
>>         return self.sequence[key]
>>
>>     def __setitem__(self, key, value):
>>         """Support item assignment (for mutable sequences)"""
>>         if isinstance(key, slice):
>>             # Convert to list, modify, then back to string
>>             seq_list = list(self.sequence)
>>             seq_list[key] = value.upper()
>>             self.sequence = ''.join(seq_list)
>>         else:
>>             seq_list = list(self.sequence)
>>             seq_list[key] = value.upper()
>>             self.sequence = ''.join(seq_list)
>>
>>     def __repr__(self):
>>         return f"DNASequence('{self.sequence}')"
>>
>>     def __len__(self):
>>         return len(self.sequence)
>>
>> # Example usage
>> seq = DNASequence("ATGCGTTAGCT")
>>
>> print(f"Original: {seq}")
>> print(f"First 5 bases: {seq[:5]}")
>> print(f"Every other base: {seq[::2]}")
>> print(f"Base at position 3: {seq[3]}")
>>
>> # For mutable operations (if needed)
>> mutable_seq = DNASequence("ATGC")
>> print(f"Before mutation: {mutable_seq}")
>> mutable_seq[1] = "C"  # Change second base
>> print(f"After mutation: {mutable_seq}")
>> ```
>> {: .language-python}
>>
>> ```
>> Original: DNASequence('ATGCGTTAGCT')
>> First 5 bases: DNASequence('ATGCG')
>> Every other base: DNASequence('AGCTT')
>> Base at position 3: C
>> Before mutation: DNASequence('ATGC')
>> After mutation: DNASequence('ACGC')
>> ```
>> {: .output}
> {: .solution}
{: .challenge}

> ## Challenge: Sorting sequences by length
>
> Implement `__lt__` in `DNASequence` to allow sorting a list of sequences by length. Also implement the other comparison methods for completeness.
>
>> ## Solution
>>
>> ```python
>> class DNASequence:
>>     def __init__(self, sequence):
>>         self.sequence = sequence.upper()
>>
>>     def __lt__(self, other):
>>         """Compare by sequence length"""
>>         return len(self.sequence) < len(other.sequence)
>>
>>     def __le__(self, other):
>>         return len(self.sequence) <= len(other.sequence)
>>
>>     def __gt__(self, other):
>>         return len(self.sequence) > len(other.sequence)
>>
>>     def __ge__(self, other):
>>         return len(self.sequence) >= len(other.sequence)
>>
>>     def __eq__(self, other):
>>         return self.sequence == other.sequence
>>
>>     def __len__(self):
>>         return len(self.sequence)
>>
>>     def __repr__(self):
>>         return f"DNASequence('{self.sequence}')"
>>
>> # Example usage
>> sequences = [
>>     DNASequence("ATG"),
>>     DNASequence("ATGCGT"), 
>>     DNASequence("A"),
>>     DNASequence("ATGCCGTA")
>> ]
>>
>> print("Sorted by length:")
>> for seq in sorted(sequences):
>>     print(f"  {seq} (length: {len(seq)})")
>>
>> print(f"\nLongest > Shortest: {sequences[3] > sequences[2]}")
>> ```
>> {: .language-python}
>>
>> ```
>> Sorted by length:
>>   DNASequence('A') (length: 1)
>>   DNASequence('ATG') (length: 3)
>>   DNASequence('ATGCGT') (length: 6)
>>   DNASequence('ATGCCGTA') (length: 8)
>>
>> Longest > Shortest: True
>> ```
>> {: .output}
> {: .solution}
{: .challenge}

> ## Challenge: Iterable motifs
>
> Create a `MotifFinder` class that, given a sequence and a motif, allows iteration over all positions where the motif occurs using `__iter__` and `__next__`. Also support reverse iteration.
>
>> ## Solution
>>
>> ```python
>> class MotifFinder:
>>     def __init__(self, sequence, motif):
>>         self.sequence = sequence.upper()
>>         self.motif = motif.upper()
>>         self.current_index = 0
>>
>>     def __iter__(self):
>>         """Reset iterator and return self"""
>>         self.current_index = 0
>>         return self
>>
>>     def __next__(self):
>>         """Find next occurrence of motif"""
>>         pos = self.sequence.find(self.motif, self.current_index)
>>         if pos == -1:
>>             raise StopIteration
>>         self.current_index = pos + 1
>>         return pos
>>
>>     def __reversed__(self):
>>         """Find motifs in reverse order"""
>>         positions = []
>>         pos = -1
>>         while True:
>>             pos = self.sequence.find(self.motif, pos + 1)
>>             if pos == -1:
>>                 break
>>             positions.append(pos)
>>         return reversed(positions)
>>
>>     def all_positions(self):
>>         """Return all positions as a list"""
>>         return list(iter(self))
>>
>> # Example usage
>> finder = MotifFinder("ATGCGATGCGTATGCG", "GCG")
>>
>> print("Forward iteration:")
>> for pos in finder:
>>     print(f"  Motif found at position: {pos}")
>>
>> print("\nReverse iteration:")
>> for pos in reversed(finder):
>>     print(f"  Motif found at position: {pos}")
>>
>> print(f"\nAll positions: {finder.all_positions()}")
>> ```
>> {: .language-python}
>>
>> ```
>> Forward iteration:
>>   Motif found at position: 2
>>   Motif found at position: 7
>>   Motif found at position: 13
>>
>> Reverse iteration:
>>   Motif found at position: 13
>>   Motif found at position: 7
>>   Motif found at position: 2
>>
>> All positions: [2, 7, 13]
>> ```
>> {: .output}
> {: .solution}
{: .challenge}


{% include links.md %}
