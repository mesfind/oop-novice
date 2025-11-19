---
title: "Special methods in Bioinformatics"
teaching: 25
exercises: 15
questions:
- "How can classes allow their instances to work with standard Python operators for biological sequences or scores?"
- "How can classes allow their instances to behave like iterables for sequence records or genomic features?"
- "How can classes allow their instances to be called like functions in bioinformatics pipelines?"
objectives:
- "Be able to implement methods like `__add__`, `__eq__`, and `__gt__` for bioinformatics data."
- "Be able to implement methods like `__len__`, `__iter__`, and `__reversed__` for sequences or collections."
- "Be able to implement the `__call__` method to make objects act as callable analysis tools."
keypoints:
- "Implement methods like `__eq__`, `__add__`, and `__gt__` to allow comparison and combination of sequences, genes, or scores."
- "Implement `__repr__` to display more meaningful representations of sequences or objects."
- "Implement methods like `__len__`, `__iter__`, and `__reversed__` to make sequence objects behave like iterable collections."
- "Implement the `__call__` method to make instances act as analysis functions or pipelines."
---

In bioinformatics, we often have sequences or gene objects that we want to compare, combine, or iterate over. Consider two sequences with the same nucleotides:

~~~
seq1 = "ATGCGT"
seq2 = "ATGCGT"
if seq1 == seq2:
    print("Python thinks these sequences are identical.")
else:
    print("Python thinks these sequences are different.")
~~~
{: .language-python}

~~~
Python thinks these sequences are identical.
~~~
{: .output}

This works for simple strings, but for **sequence objects**, Python distinguishes objects unless we implement special methods. For example:

~~~
class DNASequence:
    def __init__(self, seq):
        self.seq = seq.upper()
~~~
{: .language-python}

Two DNASequence objects with the same sequence will not compare equal by default:

~~~
a_seq = DNASequence("ATGCGT")
b_seq = DNASequence("ATGCGT")
print(a_seq == b_seq)
~~~
{: .language-python}

~~~
False
~~~
{: .output}

Implementing the `__eq__` method allows Python to compare equality based on content rather than identity:

~~~
class DNASequence:
    def __init__(self, seq):
        self.seq = seq.upper()

    def __eq__(self, other):
        if not isinstance(other, DNASequence):
            return False
        return self.seq == other.seq

    def __repr__(self):
        return f"DNASequence({self.seq})"

a_seq = DNASequence("ATGCGT")
b_seq = DNASequence("ATGCGT")
print(a_seq == b_seq)
~~~
{: .language-python}

~~~
True
~~~
{: .output}

This pattern can be extended to compare genes by length, GC content, or other biological metrics using `__lt__`, `__gt__`, etc.



> ## Challenge: Compare sequences by GC content
>
> Implement a `Gene` class with `__eq__`, `__lt__`, and `__gt__` methods. Compare two genes first by GC content, then by length if GC content is equal.
>
> > ## Solution
> >
> > ~~~
> > class Gene:
> >     def __init__(self, name, seq):
> >         self.name = name
> >         self.seq = seq.upper()
> >
> >     def gc_content(self):
> >         gc = sum(1 for base in self.seq if base in "GC")
> >         return gc / len(self.seq)
> >
> >     def __eq__(self, other):
> >         return self.seq == other.seq
> >
> >     def __lt__(self, other):
> >         if self.gc_content() != other.gc_content():
> >             return self.gc_content() < other.gc_content()
> >         return len(self.seq) < len(other.seq)
> >
> >     def __repr__(self):
> >         return f"Gene({self.name}, {self.seq})"
> > ~~~
> > {: .language-python}
> > {: .solution}

---

### Arithmetic with bioinformatics objects

Special methods like `__add__` can combine sequences or aggregate scores. For example, a `QualityScore` class representing Phred scores:

~~~
class QualityScore:
    def __init__(self, scores):
        self.scores = scores

    def __add__(self, other):
        combined = [a+b for a, b in zip(self.scores, other.scores)]
        return QualityScore(combined)

    def __repr__(self):
        return f"QualityScore({self.scores})"

qs1 = QualityScore([30, 32, 28])
qs2 = QualityScore([31, 29, 30])
qs_total = qs1 + qs2
print(qs_total)
~~~
{: .language-python}

~~~
QualityScore([61, 61, 58])
~~~
{: .output}


### Callable objects

Bioinformatics pipelines often benefit from callable objects for modular analysis. For example:

~~~
class SequenceAnalyzer:
    def __init__(self, motif="ATG"):
        self.motif = motif

    def analyze(self, sequence):
        return sequence.count(self.motif)

    def __call__(self, sequence):
        return self.analyze(sequence)

analyzer = SequenceAnalyzer()
print(analyzer("ATGCGTATG"))
~~~
{: .language-python}

~~~
2
~~~
{: .output}

> ## Challenge: Extend analyzer
>
> Make a callable `MotifCounter` that counts multiple motifs in a sequence and returns a dictionary with counts.
>
> > ## Solution
> >
> > ```python
> > class MotifCounter:
> >     def __init__(self, motifs):
> >         self.motifs = motifs
> >
> >     def __call__(self, sequence):
> >         return {motif: sequence.count(motif) for motif in self.motifs}
> >
> > mc = MotifCounter(["ATG", "CGT"])
> > print(mc("ATGCGTATG"))
> > ```
> >
> > {: .language-python}
> >
> > ```
> > {'ATG': 2, 'CGT': 1}
> > ```
> >
> > {: .output}
> > {: .solution}

---

### Collections and iterables

Sequence objects can behave like collections:

~~~
class DNASequence:
    def __init__(self, seq):
        self.seq = seq

    def __len__(self):
        return len(self.seq)

    def __iter__(self):
        return iter(self.seq)

    def __reversed__(self):
        return reversed(self.seq)

seq = DNASequence("ATGCGT")
print(len(seq))
for base in seq:
    print(base)
for base in reversed(seq):
    print(base)
~~~
{: .language-python}

~~~
6
A
T
G
C
G
T
T
G
C
G
T
A
~~~
{: .output}

> ## Challenge: Implement `__getitem__` for DNASequence
>
> Access individual nucleotides using indexing; slices should raise `IndexError`.
>
> > ## Solution
> >
> > ```python
> > class DNASequence:
> >     def __init__(self, seq):
> >         self.seq = seq
> >
> >     def __getitem__(self, key):
> >         if isinstance(key, int):
> >             return self.seq[key]
> >         else:
> >             raise IndexError("Slicing not supported")
> >
> > dna = DNASequence("ATGCGT")
> > print(dna[2])
> > ```
> >
> > {: .language-python}
> >
> > ```
> > G
> > ```
> >
> > {: .output}
> > {: .solution}


