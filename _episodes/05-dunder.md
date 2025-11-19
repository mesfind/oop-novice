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
---

In bioinformatics, we often want to compare genes, sequences, or other biological entities. By default, Python only considers two objects equal if they are the exact same object in memory:

~~~
class Gene:
    def __init__(self, name, sequence):
        self.name = name
        self.sequence = sequence

gene1 = Gene("geneA", "ATGCGT")
gene2 = Gene("geneA", "ATGCGT")
print(gene1 == gene2)
~~~
{: .language-python}

~~~
False
~~~
{: .output}

We can fix this by implementing `__eq__` and other special methods.


> ## Challenge: Gene equality by sequence
>
> Implement a `Gene` class that supports equality comparisons (`==`) based on sequence rather than object identity.
>
> > ## Solution
> >
> > ~~~
> > class Gene:
> >     def __init__(self, name, sequence):
> >         self.name = name
> >         self.sequence = sequence.upper()
> >
> >     def __eq__(self, other):
> >         if not isinstance(other, Gene):
> >             return False
> >         return self.sequence == other.sequence
> >
> >     def __repr__(self):
> >         return f"Gene({self.name}, {self.sequence})"
> >
> > # Testing
> > gene1 = Gene("geneA", "ATGCGT")
> > gene2 = Gene("geneA", "ATGCGT")
> > gene1 == gene2
> > ~~~
> > 
> > {: .language-python}
> {: .solution}
> {: .challenge}

---

> ## Challenge: Compare genes by GC content
>
> Implement `__lt__` and `__gt__` for a `Gene` class so genes can be sorted by GC content.
>
> > ## Solution
> >
> > ~~~
> > class Gene:
> >     def __init__(self, name, sequence):
> >         self.name = name
> >         self.sequence = sequence.upper()
> >
> >     def gc_content(self):
> >         gc = sum(1 for b in self.sequence if b in "GC")
> >         return gc / len(self.sequence)
> >
> >     def __lt__(self, other):
> >         return self.gc_content() < other.gc_content()
> >
> >     def __gt__(self, other):
> >         return self.gc_content() > other.gc_content()
> >
> >     def __repr__(self):
> >         return f"Gene({self.name}, {self.sequence})"
> >
> > # Example
> > gene_list = [Gene("gene1", "ATGC"), Gene("gene2", "GCGC"), Gene("gene3", "ATAT")]
> > sorted(gene_list)
> > ~~~
> >
> > {: .language-python}
> {: .solution}
{: .challenge}

---

> ## Challenge: Sequence arithmetic
>
> Implement a `DNAError` class to store a DNA length with measurement error, and implement `__add__`, `__sub__`, `__mul__`, and `__truediv__` to propagate errors.
>
> > ## Solution
> >
> > ~~~
> > class DNAError:
> >     def __init__(self, length, error):
> >         self.length = length
> >         self.error = error
> >
> >     def __repr__(self):
> >         return f"{self.length} ± {self.error}"
> >
> >     def __add__(self, other):
> >         l = self.length + other.length
> >         e = (self.error**2 + other.error**2)**0.5
> >         return DNAError(l, e)
> >
> >     def __sub__(self, other):
> >         l = self.length - other.length
> >         e = (self.error**2 + other.error**2)**0.5
> >         return DNAError(l, e)
> >
> >     def __mul__(self, other):
> >         l = self.length * other.length
> >         e = l * ((self.error/self.length)**2 + (other.error/other.length)**2)**0.5
> >         return DNAError(l, e)
> >
> >     def __truediv__(self, other):
> >         l = self.length / other.length
> >         e = l * ((self.error/self.length)**2 + (other.error/other.length)**2)**0.5
> >         return DNAError(l, e)
> > ~~~
> >
> > {: .language-python}
> {: .solution}
{: .challenge}


> ## Challenge: Callable sequence plotter
>
> Create a `SequencePlotter` class that plots GC content along a sequence when called as a function using `__call__`.
>
> > ## Solution
> >
> > ~~~
> > import matplotlib.pyplot as plt
> >
> > class SequencePlotter:
> >     def __init__(self, window=10):
> >         self.window = window
> >
> >     def plot_gc(self, sequence):
> >         seq = sequence.upper()
> >         gc_values = [sum(1 for b in seq[i:i+self.window] if b in "GC")/self.window
> >                      for i in range(len(seq)-self.window+1)]
> >         plt.plot(gc_values)
> >         plt.show()
> >
> >     def __call__(self, sequence):
> >         return self.plot_gc(sequence)
> >
> > # Example usage
> > plotter = SequencePlotter(window=5)
> > plotter("ATGCGCGTATATGCGC")
> > ~~~
> >
> > {: .language-python}
> {: .solution}
{: .challenge}


> ## Challenge: Iterable genome
>
> Implement a `Genome` class that allows iterating over genes using `__iter__` and accessing them by index with `__getitem__`.
>
> > ## Solution
> >
> > ~~~
> > class Genome:
> >     def __init__(self, genes):
> >         self.genes = genes
> >
> >     def __iter__(self):
> >         return iter(self.genes)
> >
> >     def __getitem__(self, index):
> >         if isinstance(index, int):
> >             return self.genes[index]
> >         else:
> >             raise IndexError
> >
> >     def __len__(self):
> >         return len(self.genes)
> >
> > # Example
> > g1 = Gene("gene1", "ATGC")
> > g2 = Gene("gene2", "GCGC")
> > genome = Genome([g1, g2])
> > for gene in genome:
> >     print(gene)
> > genome[1]
> > len(genome)
> > ~~~
> >
> > {: .language-python}
> {: .solution}
{: .challenge}


> ## Challenge: Reverse iteration
>
> Implement `__reversed__` for the `Genome` class so that `reversed(genome)` yields genes in reverse order.
>
> > ## Solution
> >
> > ~~~
> > class Genome:
> >     ...
> >     def __reversed__(self):
> >         return reversed(self.genes)
> > ~~~
> >
> > {: .language-python}
> {: .solution}
{: .challenge}


> ## Challenge: Complement arithmetic
>
> Implement a `DNASequence` class that supports `+` for concatenation and `*` to repeat the sequence N times. Include `__repr__`.
>
> > ## Solution
> >
> > ~~~
> > class DNASequence:
> >     def __init__(self, sequence):
> >         self.sequence = sequence.upper()
> >
> >     def __add__(self, other):
> >         return DNASequence(self.sequence + other.sequence)
> >
> >     def __mul__(self, n):
> >         return DNASequence(self.sequence * n)
> >
> >     def __repr__(self):
> >         return f"DNASequence('{self.sequence}')"
> >
> > # Testing
> > a = DNASequence("ATG")
> > b = DNASequence("CGT")
> > a + b
> > a * 3
> > ~~~
> >
> > {: .language-python}
> > {: .solution}
> > {: .challenge}


> ## Challenge: Gene filtering with `__call__`
>
> Modify the `Gene` class so that instances can be called as a function returning `True` if a given motif is present in the sequence.
>
> > ## Solution
> >
> > ~~~
> > class Gene:
> >     def __init__(self, name, sequence):
> >         self.name = name
> >         self.sequence = sequence.upper()
> >
> >     def __call__(self, motif):
> >         return motif.upper() in self.sequence
> >
> >     def __repr__(self):
> >         return f"Gene({self.name}, {self.sequence})"
> >
> > # Example
> > g = Gene("geneA", "ATGCGT")
> > g("GCG")
> > g("AAA")
> > ~~~
> >
> > {: .language-python}
> {: .solution}
{: .challenge}


> ## Challenge: Slice access for sequences
>
> Extend the `DNASequence` class to support slicing with `__getitem__`, returning a new `DNASequence` object.
>
> > ## Solution
> >
> > ~~~
> > class DNASequence:
> >     ...
> >     def __getitem__(self, key):
> >         return DNASequence(self.sequence[key])
> >
> > # Example
> > seq = DNASequence("ATGCGT")
> > seq[1:4]
> > ~~~
> >
> > {: .language-python}
> {: .solution}
{: .challenge}



> ## Challenge: Sorting sequences by length
>
> Implement `__lt__` in `DNASequence` to allow sorting a list of sequences by length.
>
> > ## Solution
> >
> > ~~~
> > class DNASequence:
> >     ...
> >     def __lt__(self, other):
> >         return len(self.sequence) < len(other.sequence)
> >
> > # Example
> > seqs = [DNASequence("ATG"), DNASequence("ATGCGT"), DNASequence("A")]
> > sorted(seqs)
> > ~~~
> > {: .language-python}
> > 
> {: .solution}
{: .challenge}


> ## Challenge: Iterable motifs
>
> Create a `MotifFinder` class that, given a sequence and a motif, allows iteration over all positions where the motif occurs using `__iter__`.
>
> > ## Solution
> >
> > ~~~
> > class MotifFinder:
> >     def __init__(self, sequence, motif):
> >         self.sequence = sequence.upper()
> >         self.motif = motif.upper()
> >
> >     def __iter__(self):
> >         self.index = 0
> >         return self
> >
> >     def __next__(self):
> >         pos = self.sequence.find(self.motif, self.index)
> >         if pos == -1:
> >             raise StopIteration
> >         self.index = pos + 1
> >         return pos
> >
> > # Example
> > finder = MotifFinder("ATGCGATGCGT", "GCG")
> > list(finder)
> > ~~~
> > {: .language-python}
> {: .solution}
{: .challenge}




