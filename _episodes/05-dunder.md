---
title: "Special methods in Bioinformatics"
teaching: 25
exercises: 15
questions:
- "How can bioinformatics classes allow their instances to work with standard Python operators?"
- "How can bioinformatics classes allow their instances to behave like iterables or collections?"
- "How can bioinformatics classes allow their instances to be called like functions?"
objectives:
- "Be able to implement methods like `__add__`, `__eq__`, and `__gt__` in bioinformatics context."
- "Be able to implement methods like `__len__`, `__iter__`, and `__reversed__`."
- "Be able to implement the `__call__` method."
keypoints:
- "Implement methods like `__eq__`, `__add__`, and `__gt__` to allow operations such as sequence comparisons and concatenations."
- "Implement `__repr__` to get more meaningful printouts when you output bioinformatics objects."
- "Implement methods like `__len__`, `__iter__`, and `__reversed__` to make bioinformatics objects behave like a collection or iterable."
- "Implement the `__call__` method to make instances of a bioinformatics class callable like functions."
***

In bioinformatics, we may want to compare DNA sequence objects based on their biological properties. For example, two sequences with the exact same nucleotide content could be considered equal.

Let's create a simple `DNASequence` class and implement the `__eq__` special method for equality comparison:

~~~python
class DNASequence:
    def __init__(self, sequence):
        self.sequence = sequence.upper()

    def __eq__(self, other):
        if not isinstance(other, DNASequence):
            return False
        return self.sequence == other.sequence

    def __repr__(self):
        return f"DNASequence('{self.sequence}')"
        
seq1 = DNASequence("ATGC")
seq2 = DNASequence("ATGC")
seq3 = DNASequence("GCTA")

print(seq1 == seq2)  # True
print(seq1 == seq3)  # False
print(seq1)          # DNASequence('ATGC')
~~~
{: .language-python}

Output:

~~~
True
False
DNASequence('ATGC')
~~~

Great! This lets us compare sequence objects easily and also get friendly string representations.

***

We can also define relational operators to compare sequences according to custom biological metrics. For example, define ordering based on GC content:

~~~python
class DNASequence:
    def __init__(self, sequence):
        self.sequence = sequence.upper()

    def gc_content(self):
        return (self.sequence.count('G') + self.sequence.count('C')) / len(self.sequence)

    def __lt__(self, other):
        if not isinstance(other, DNASequence):
            return NotImplemented
        return self.gc_content() < other.gc_content()

    def __repr__(self):
        return f"DNASequence('{self.sequence}')"

seqA = DNASequence("ATGC")
seqB = DNASequence("GCGC")

print(seqA < seqB)  # True, because seqB has higher GC content
~~~
{: .language-python}

***

We might want to combine sequences with the `+` operator by concatenation:

~~~python
class DNASequence:
    ...
    def __add__(self, other):
        if not isinstance(other, DNASequence):
            return NotImplemented
        return DNASequence(self.sequence + other.sequence)

seq1 = DNASequence("ATG")
seq2 = DNASequence("CGA")
combined = seq1 + seq2
print(combined)  # DNASequence('ATGCGA')
~~~
{: .language-python}

***

To behave like a collection, we can implement methods like `__len__`, `__iter__`, and `__reversed__`:

~~~python
class DNASequence:
    ...
    def __len__(self):
        return len(self.sequence)

    def __iter__(self):
        return iter(self.sequence)

    def __reversed__(self):
        return reversed(self.sequence)

seq = DNASequence("ATGC")
print(len(seq))  # 4

for base in seq:
    print(base)
    
print(''.join(reversed(seq)))  # CGTA
~~~
{: .language-python}

***

Finally, by implementing `__call__`, we can make DNA sequence objects behave like functions. For example, to transcribe DNA to RNA:

~~~python
class DNASequence:
    ...
    def __call__(self):
        return self.sequence.replace('T', 'U')

dna = DNASequence("ATGC")
print(dna())  # AUGC
~~~
{: .language-python}

***

> ## Challenge: Implement a `ProteinSequence` class with similar special methods considering amino acid properties.
