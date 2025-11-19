---
title: "Duck typing and interfaces in Bioinformatics"
teaching: 10
exercises: 15
questions:
- "How does Python decide what you can and can't do with an object in bioinformatics?"
- "When is inheritance not appropriate for bioinformatics data types?"
- "What alternatives are there to inheritance when designing bioinformatics software?"
objectives:
- "Understand how duck typing works, and how interfaces assist with understanding this."
- "Understand the circumstances where inheritance can be a hindrance rather than a help."
- "Be aware of concepts such as composition which can help where inheritance fails."
keypoints:
- "Provided a bioinformatics class exposes all required functionality for an operation to work, Python allows it."
- "Only use inheritance to express relationships where the subclass is the same kind of thing as the superclass."
- "Implementing interfaces and adding functionality with composition can be better alternatives to inheritance in some cases."
***

There is a principle that if something "looks like a duck, and swims like a duck, and quacks like a duck, then it is probably a duck".

{% include image.html url="../assets/img/duck.jpg"
alt="Photograph of a duck and three ducklings on a body of
water" caption="These ducks are swimming and look like ducks, although
the quacking can't be guaranteed from this image."%}

Python's type system adopts a similar philosophy—if an object behaves like a biological sequence with the methods you expect (e.g., `transcribe()`, `complement()`, `reverse_complement()`), then Python treats it as a sequence, regardless of its concrete class.

For example, an iterative algorithm for genome sequence assembly may accept any object with a `get_sequence()` method that returns a nucleotide string, regardless of the object's class.

As a concrete coding example, here is a Newton–Raphson method solver that works on any object supporting arithmetic operations—even complex numbers or biological quantities modeled as objects—demonstrating duck typing:

~~~python
def newton(function, derivative, initial_estimate, num_iters=10):
    """Solves f(x)=0 using Newton-Raphson method. `function` and `derivative`
    should support arithmetic operations appropriate to `initial_estimate`."""

    current_estimate = initial_estimate
    for _ in range(num_iters):
        current_estimate = (
            current_estimate
            - function(current_estimate) / derivative(current_estimate)
        )
    return current_estimate
~~~

This function works with real numbers, complex numbers, or even custom bioinformatics objects that overload required arithmetic:

~~~python
from math import sin, cos

print(newton(sin, cos, 1))   # works with real numbers
print(newton(sin, cos, 1 + 1j))  # works with complex numbers
~~~

In bioinformatics, this means you can write generalized algorithms that operate on any sequence-like object or numerical object supporting the needed operations, without enforcing strong inheritance relations.

***

## Protocols and Iterators in Bioinformatics

A bioinformatics iterator protocol example is a `FastaRecordIterator` that reads sequences one by one from a large FASTA file:

~~~python
class FastaRecordIterator:
    def __init__(self, fasta_file):
        self.fasta_file = open(fasta_file)
        self.next_record = None

    def __iter__(self):
        return self

    def __next__(self):
        # logic to read the next FASTA record from file
        line = self.fasta_file.readline()
        if not line:
            self.fasta_file.close()
            raise StopIteration
        # parse FASTA record...
        record = line.strip()  # simplified
        return record
~~~

Iterators let you process large biological datasets efficiently by reading data lazily.

***

## Composition Over Inheritance in Bioinformatics

When handling complex bioinformatics workflows, composition can be better than inheritance to add functionality.

For example, a `SequenceAnalyzer` class can be composed with a `SequenceFetcher` and a `SequenceAligner`, coordinating functionality across these components without requiring deep class hierarchies.

This promotes modularity and easier maintenance especially when integrating multiple bioinformatics tools.

***

> ## Bioinformatics challenge
>
> Write a class complying with the iterator protocol to iterate over the first `n` GC-rich genes from a genome dataset.
>
> Consider where using duck typing or composition may be preferable to building deep inheritance for your research software.

{: .challenge}

