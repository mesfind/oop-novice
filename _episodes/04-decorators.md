---
title: "Decorators, class methods, and properties in Bioinformatics"
teaching: 20
exercises: 25
questions:
- "What is a decorator?"
- "How do I tag methods that operate at the class level in bioinformatics workflows?"
- "How can I add logic to control updates to instance variables in biological data objects?"
objectives:
- "Understand decorators and their implementation in biological data processing"
- "Be able to use `@classmethod` and `@property` in bioinformatics classes"
keypoints:
- "A decorator adds functionality to a function or class. To apply one, add `@decoratorname` one line above the definition."
- Use the `@classmethod` decorator for methods applied to the class rather than an instance.
- Use the `@property` decorator to control access to instance variables such as sequences or metadata.
---

Sometimes when we write bioinformatics software we want to attach additional logic to functions without modifying the functions themselves. Python provides syntax that makes this practical.

Say we want to monitor which functions are used during a genomic analysis. We can write a wrapper that reports entry and exit for any function that processes sequence data.

~~~python
def track_this(function):
    def new_function():
        print("Entering", function)
        function()
        print("Leaving", function)
    return new_function
~~~

To test this idea with simple sequence utilities:

~~~python
def load_sequence():
    print("Loading sequence.")

load_sequence = track_this(load_sequence)

def filter_sequence():
    print("Filtering sequence.")

filter_sequence = track_this(filter_sequence)

def pipeline():
    load_sequence()
    filter_sequence()

pipeline = track_this(pipeline)
pipeline()
~~~

This prints entry and exit messages for each analysis step. However, rewriting each function assignment is tedious. Python provides a cleaner way.

~~~python
@track_this
def load_sequence():
    print("Loading sequence.")

@track_this
def filter_sequence():
    print("Filtering sequence.")

@track_this
def pipeline():
    load_sequence()
    filter_sequence()

pipeline()
~~~

This syntax is called a decorator.

Our `track_this` decorator is currently limited because it accepts no arguments. Functions in bioinformatics often take inputs such as sequences, IDs, and metadata. If we try to decorate such a function, we get an error.

> ## Decorators and arguments
>
> ~~~python
> @track_this
> def count_nucleotides(seq):
>     print(len(seq))
>
> count_nucleotides("ATGCT")
> ~~~
>
> ~~~
> TypeError
> new_function() takes 0 positional arguments but 1 was given
> ~~~

We can fix this by allowing positional and keyword arguments.

~~~python
def track_this(function):
    def new_function(*args, **kwargs):
        print("Entering", function)
        function(*args, **kwargs)
        print("Leaving", function)
    return new_function
~~~

`*args` stores positional arguments.  
`**kwargs` stores keyword arguments.

> ## Double checking
>
> Write a decorator that runs a computational step twice and checks that both outputs match. Use it on methods that compute GC content or sequence length.

> ### Solution
>
> ~~~python
> class InconsistentResultsError(AssertionError):
>     pass
>
> def check_consistency(function):
>     def wrapped(*args, **kwargs):
>         results = [function(*args, **kwargs) for _ in range(2)]
>         if results[0] != results[1]:
>             raise InconsistentResultsError
>         return results[0]
>     return wrapped
>
> class SequenceRecord:
>     def __init__(self, seq):
>         self.seq = seq
>
>     @check_consistency
>     def length(self):
>         return len(self.seq)
>
>     @check_consistency
>     def gc_content(self):
>         g = self.seq.count("G")
>         c = self.seq.count("C")
>         return (g + c) / len(self.seq)
>
> rec = SequenceRecord("ATGCCG")
> print(rec.length())
> print(rec.gc_content())
> ~~~

## Class methods

Class methods are useful for alternate constructors in bioinformatics data structures. For example, we may want a constructor that builds a synthetic sequence of repeated nucleotides.

~~~python
class SequenceRecord:
    def __init__(self, seq):
        self.seq = seq

    @classmethod
    def poly_n(cls, base, length):
        return cls(base * length)

    def gc_content(self):
        g = self.seq.count("G")
        c = self.seq.count("C")
        return (g + c) / len(self.seq)
~~~

Testing:

~~~python
polyA = SequenceRecord.poly_n("A", 10)
print(polyA.seq)
print(polyA.gc_content())
~~~

> ## Synthetic sequences
>
> Add a class method that builds a random DNA sequence of a given length.

## Properties

In biological data classes, unrestricted attribute assignment can damage downstream analysis. For example, changing a sequence without validation may break methods that assume valid nucleotide characters.

We can protect sequence assignment using properties.

~~~python
class SequenceRecord:
    def __init__(self, seq):
        self.sequence = seq

    @property
    def sequence(self):
        return self._sequence

    @sequence.setter
    def sequence(self, seq):
        allowed = set("ACGTN")
        assert set(seq).issubset(allowed)
        self._sequence = seq

    def length(self):
        return len(self._sequence)
~~~

Testing:

~~~python
rec = SequenceRecord("ATGC")
print(rec.sequence)
rec.sequence = "AATTGG"
print(rec.length())
~~~

Invalid assignment raises an error.

> ## More robust plotters
>
> Modify a genomic coverage plotter so that its color attribute is controlled through a property with validation.

> ### Solution
>
> ~~~python
> from matplotlib.colors import is_color_like
> from matplotlib.pyplot import subplots
> import numpy as np
>
> class CoveragePlotter:
>     def __init__(self, color="black", x_min=0, x_max=1000):
>         self.color = color
>         self.x_min = x_min
>         self.x_max = x_max
>
>     @property
>     def color(self):
>         return self._color
>
>     @color.setter
>     def color(self, value):
>         assert is_color_like(value)
>         self._color = value
>
>     def plot(self, coverage):
>         fig, ax = subplots()
>         x = np.arange(self.x_min, self.x_max)
>         ax.plot(x, coverage[: len(x)], color=self._color)
> ~~~
