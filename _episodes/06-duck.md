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
---

There is a principle that if something "looks like a duck, and swims like a duck, and quacks like a duck, then it is probably a duck".

{% include image.html url="../assets/img/duck.jpg"
alt="Photograph of a duck and three ducklings on a body of
water" caption="These ducks are swimming and look like ducks, although
the quacking can't be guaranteed from this image."%}

Python's type system adopts a similar philosophy—if an object behaves like a biological sequence with the methods you expect (e.g., `transcribe()`, `complement()`, `reverse_complement()`), then Python treats it as a sequence, regardless of its concrete class.

For example, an assembly algorithm might accept any object that has a `get_sequence()` method returning nucleotide data, without requiring a strict class inheritance.

Here is a duck-typed Newton-Raphson solver that works on any numeric type, including bioinformatics-related numerical objects if they support required operations:

~~~python
def newton(function, derivative, initial_estimate, num_iters=10):
    """Solves f(x)=0 using Newton-Raphson method. Supports duck typing."""
    current_estimate = initial_estimate
    for _ in range(num_iters):
        current_estimate = (
            current_estimate
            - function(current_estimate) / derivative(current_estimate)
        )
    return current_estimate
~~~

This works with real numbers, complex numbers, or customized numerical objects representing biological quantities.

Bioinformatics-specific duck typing allows writing generic code operating on varied sequence-like objects or analysis results without enforcing strict inheritance.

***

## Protocols in Bioinformatics

Protocols specify required methods for an object to interact with Python features. For example, an iterator protocol for iterating over sequence records:

~~~python
class FastaIterator:
    def __init__(self, fasta_path):
        self.file = open(fasta_path)

    def __iter__(self):
        return self

    def __next__(self):
        line = self.file.readline()
        if not line:
            self.file.close()
            raise StopIteration
        # Simplified: return sequence line only
        return line.strip()
~~~

Using it with a `for` loop processes large datasets lazily.

***

## Composition vs. Inheritance in Bioinformatics

Composition assembles complex bioinformatics functionality by including specialized classes as attributes rather than deep inheritance hierarchies.

Example:

~~~python
class SequenceFetcher:
    def fetch(self):
        pass  # Fetch sequences from database or file

class SequenceAligner:
    def align(self, seqs):
        pass  # Align sequences

class Pipeline:
    def __init__(self, fetcher, aligner):
        self.fetcher = fetcher
        self.aligner = aligner

    def run(self):
        sequences = self.fetcher.fetch()
        return self.aligner.align(sequences)
~~~

This design improves modularity and flexibility over large inheritance trees common in bioinformatics libraries.

***

> ## Challenge: Write an iterator class to yield GC-rich gene sequences from large genome data.

> ## Challenge: Design a duck-typed interface for biological sequence objects supporting `length()`, `complement()`, and `transcribe()` methods.

> ## Challenge: Refactor a bioinformatics workflow to use composition rather than inheritance for tool integration.

{: .challenge}

Here are the missing keypoints and solutions for the challenges, formatted mindfully in context with bioinformatics:

***
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
keypoints:
- "In bioinformatics, duck typing means that if an object provides all the expected sequence or processing methods, it is accepted regardless of its actual class."
- "Duck typing enables writing flexible and generic analysis tools that operate on any sequence-like object implementing required methods, without enforcing inheritance."
- "Inheritance should be used only when subclasses truly represent a specialized version of the parent bioinformatics concept."
- "Composition—combining classes by including others as components—is often a better alternative for complex bioinformatics software integration."
- "Protocols specify expected method sets for objects, facilitating interoperability without rigid class hierarchies."
- "Iterators are common protocols in bioinformatics for efficient streaming of large genome or sequence data."
- "Understanding duck typing and composition helps design modular, extensible, and maintainable bioinformatics code."
---

*Duck typing* in Python means that if an object looks and behaves appropriately (e.g., has methods like `transcribe()`, `complement()`, `reverse_complement()` for sequences), Python will allow the operations without explicitly checking the object's inheritance.

### Challenge 1: Iterator for GC-rich genes

> Write a class implementing the iterator protocol that yields genes with GC content above 50% from a genome dataset. Assume genes are represented as objects with a `.sequence` string attribute.

Solution:

~~~python
class GCRichGeneIterator:
    def __init__(self, genes):
        self.genes = genes
        self.index = 0

    def __iter__(self):
        return self

    def __next__(self):
        while self.index < len(self.genes):
            gene = self.genes[self.index]
            self.index += 1
            gc = (gene.sequence.count('G') + gene.sequence.count('C')) / len(gene.sequence)
            if gc > 0.5:
                return gene
        raise StopIteration
~~~

Usage example:

~~~python
class Gene:
    def __init__(self, name, seq):
        self.name = name
        self.sequence = seq

genes = [
    Gene('gene1', 'ATGCGC'),
    Gene('gene2', 'ATATTAT'),
    Gene('gene3', 'GCGCCG')
]

gc_iterator = GCRichGeneIterator(genes)
for gc_rich_gene in gc_iterator:
    print(gc_rich_gene.name)
~~~


~~~
gene1
gene3
~~~
{: output}


### Challenge 2: Duck-typed biosequence interface

> Design an interface (informal in Python) for biological sequence objects with methods `length()`, `complement()`, and `transcribe()`.

Solution example:

~~~python
class BioSequenceInterface:
    def length(self):
        raise NotImplementedError()

    def complement(self):
        raise NotImplementedError()

    def transcribe(self):
        raise NotImplementedError()

class DNASequence:
    def __init__(self, seq):
        self.seq = seq.upper()
    def length(self):
        return len(self.seq)
    def complement(self):
        comp = self.seq.translate(str.maketrans('ACGT', 'TGCA'))
        return comp
    def transcribe(self):
        return self.seq.replace('T', 'U')

# Usage with duck typing
def process_sequence(obj):
    print(f"Length: {obj.length()}")
    print(f"Complement: {obj.complement()}")
    print(f"RNA Transcription: {obj.transcribe()}")

dna = DNASequence("ATGC")
process_sequence(dna)
~~~

***

### Challenge 3: Composition over inheritance

> Refactor a pipeline composed of `SequenceFetcher`, `SequenceAligner`, and `ResultAnalyzer` to use composition instead of inheritance.

Solution snippet:

~~~python
class SequenceFetcher:
    def fetch(self):
        # fetch sequences
        return ["ATGC", "CGTA"]

class SequenceAligner:
    def align(self, sequences):
        # align sequences
        return "Aligned sequences"

class ResultAnalyzer:
    def analyze(self, aligned_sequences):
        return "Analysis results"

class BioPipeline:
    def __init__(self, fetcher, aligner, analyzer):
        self.fetcher = fetcher
        self.aligner = aligner
        self.analyzer = analyzer

    def run(self):
        seqs = self.fetcher.fetch()
        aligned = self.aligner.align(seqs)
        results = self.analyzer.analyze(aligned)
        return results

pipeline = BioPipeline(SequenceFetcher(), SequenceAligner(), ResultAnalyzer())
print(pipeline.run())
~~~



