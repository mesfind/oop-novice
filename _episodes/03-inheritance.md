---
title: "Inheritance in Bioinformatics"
teaching: 20
exercises: 20
questions:
- "How can class relationships where one represents a specific subset of another be represented?"
- "How can functionality on one class be overridden or extended by its children?"
objectives:
- "Be able to use inheritance to construct parent-child relationships between classes"
- "Be able to override methods on child classes, and refer back to the parent class's implementations"
keypoints:
- "Adding a class in parentheses after a class definition indicates that the new class is a subclass of the bracketed class (parent class)."
- "The subclass inherits all of that parent class's attributes and methods."
- "Defining a method with the same name as one of the parent class's overrides it."
- "Use `super()` to access parent classes and their methods."
---

We have talked about using classes as a way to reduce repetition in the software we write. However, what happens if we want to write two classes that do similar but distinct biological data processing tasks? For example, if we wanted to write a `DNASequence` class as well as our generic `BiologicalSequence` class, would we need to repeat all of the code common to both of them? What if we wanted an `RNASequence` class or a `ProteinSequence` class as well? This repetitive code would quickly start to build up...

Thankfully, Python (and most other languages that have classes) give us a mechanism to avoid this in the form of _inheritance_. A class that inherits from a second class automatically gains all of the second's attributes and methods. The class that is being inherited from is called the _parent class_, _superclass_, or _base class_, while the new class inheriting from it is called the _child class_, _subclass_, or _derived class_.

We saw earlier that `ValueError` is a subclass of `Exception`, and that this can be used to handle both specific and more general exceptions in a hierarchy. We can also use this to define our own exceptions. Say, for example, we have a function to validate a DNA sequence. We know that invalid nucleotide bases are not allowed, so if we encounter these in our code we would like to raise the alarm as soon as possible; we could do this with an `assert`, but another way of expressing this could be by defining our own exception to flag this. A nucleotide other than A, C, G, T is an example of a bad _value_, so this would want to inherit from `ValueError`.

~~~
class InvalidSequenceError(ValueError):
    pass

def validate_dna_sequence(seq):
    if not all(base in "ACGT" for base in seq.upper()):
        raise InvalidSequenceError(f"Invalid character found in DNA sequence: {seq}")
~~~
{: .language-python}

The `pass` keyword here tells Python that while we have started an indented block, we don't actually have anything to put in it. (If we were to omit it, Python would complain at us that it expected a block and didn't get one.) So we have constructed a new class called `InvalidSequenceError`, which is an exact copy of `ValueError`, except that it knows that `ValueError` is its parent. Let's test this.

~~~
for seq in ["ACGT", "acgt", "ACBX"]:
    print(f"Validating sequence: {seq}")
    validate_dna_sequence(seq)
    print("Valid sequence.")
~~~
{: .language-python}

~~~
Validating sequence: ACGT
Valid sequence.
Validating sequence: acgt
Valid sequence.
Validating sequence: ACBX
Traceback (most recent call last):
  File "<stdin>", line 4, in <module>
  File "<stdin>", line 3, in validate_dna_sequence
__main__.InvalidSequenceError: Invalid character found in DNA sequence: ACBX
~~~
{: .output}

If we wanted to, we could catch this exception with `except InvalidSequenceError` or with `except ValueError` (or even `except Exception`).

What about if we want to add functionality? Let's consider an example of a generic `BiologicalSequence` class, which can calculate length and GC content.

~~~
class BiologicalSequence:
    def __init__(self, sequence):
        self.sequence = sequence.upper()

    def length(self):
        return len(self.sequence)

    def gc_content(self):
        gc_count = sum(base in "GC" for base in self.sequence)
        return gc_count / self.length()
        
some_seq = BiologicalSequence("ATGCGATACG")
print("Length:", some_seq.length())
print("GC Content:", some_seq.gc_content())
~~~
{: .language-python}

~~~
Length: 10
GC Content: 0.5
~~~
{: .output}

Now, we know more details about DNA sequences than we do about generic biological sequences, so we can create a specialized subclass of `BiologicalSequence` called `DNASequence`. For example, for DNA sequences, we want to validate the sequence and calculate its complement.

~~~
class DNASequence(BiologicalSequence):
    def __init__(self, sequence):
        super().__init__(sequence)
        self.validate()

    def validate(self):
        if not all(base in "ACGT" for base in self.sequence):
            raise InvalidSequenceError(f"Invalid DNA sequence: {self.sequence}")

    def complement(self):
        complement_map = str.maketrans("ACGT", "TGCA")
        return self.sequence.translate(complement_map)

dna = DNASequence("ATGC")
print("Sequence:", dna.sequence)
print("Complement:", dna.complement())
~~~
{: .language-python}

~~~
Sequence: ATGC
Complement: TACG
~~~
{: .output}

We've done a few new things here. Firstly, we've overridden the `__init__` method of the `BiologicalSequence` parent class, since we now need to validate the sequence specifically for DNA. This means that only the `__init__` method from the `DNASequence` class is called, and not the one in the `BiologicalSequence` class directly. However, we use `super().__init__(sequence)` to call the parent class's initializer to set the `sequence` attribute, avoiding code repetition. Next, we've defined a new method `complement`, which is only available on the `DNASequence` class.

One niggling issue is that we are still repeating ourselves a little here. The logic to convert a sequence to uppercase exists in both classes. Using `super()` helps manage this layering properly.

> ## Not implemented
>
> If we anticipate subclasses may provide particular methods, but we can't or don't want to provide them on the superclass, we can add a stub method that raises `NotImplementedError` instead, so it becomes clear if an implementation has been forgotten. For example, the `transcribe` method of `BiologicalSequence` could be:
>
> ~~~
> def transcribe(self):
>     raise NotImplementedError("Subclasses should implement transcription.")
> ~~~
> {: .language-python}
{: .callout}

> ## Inheriting from `object`
>
> Sometimes, especially in older Python versions, you will see classes inherit from `object`. This was needed to create "new-style" classes in Python 2. In Python 3 and later, all classes inherit from `object` automatically, so it is not necessary.
{: .callout}

> ## `super()` placement challenge
>
> Suppose we want to build a `ProteinCodingGene` class that inherits from the `Gene` class. We want to ensure gene sequences are validated by the parent `Gene` class before adding translation logic. Here is how we could structure them:
>
> ~~~
> class Gene:
>     def __init__(self, locus, sequence):
>         self.locus = locus
>         self.sequence = sequence.upper()
>         self.validate()
>
>     def validate(self):
>         # Base validation to be extended
>         if not self.sequence:
>             raise ValueError("Sequence cannot be empty.")
>
> class ProteinCodingGene(Gene):
>     def __init__(self, locus, sequence, transcript_id):
>         super().__init__(locus, sequence)
>         self.transcript_id = transcript_id
>
>     def validate(self):
>         # Extend gene validation to check start codon for protein coding genes
>         super().validate()
>         if not self.sequence.startswith("ATG"):
>             raise ValueError("Protein coding gene must start with 'ATG'")
>
>     def translate(self):
>         # Simplified translation stub
>         return "M"  # Just the first amino acid Methionine here
> ~~~
>
> This example shows how important it is to place the call to `super()` correctly so parent's validations are retained and extended.
{: .challenge}

> ## Bioinformatics Exercises
>
> 1. Write a subclass `RNASequence` of `BiologicalSequence` which overrides the `validate` method to allow only bases "ACGU".
>
> 2. Extend the `ProteinCodingGene` class to implement a `translate` method converting DNA codons to amino acids using a codon table.
>
> 3. Create a `Genome` class that contains multiple `Gene` objects and implements methods to:
>    - Add genes
>    - Calculate total genome length (sum of gene lengths)
>    - Retrieve all protein-coding genes
{: .challenge}

