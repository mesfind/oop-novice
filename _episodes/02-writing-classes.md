---
title: "Writing classes in Bioinformatics"
teaching: 20
exercises: 25
questions:
- "How are classes written in Python for bioinformatics applications?"
- "What do methods look like in bioinformatics classes?"
- "How can a bioinformatics class customize instance construction and validation?"
objectives:
- "Write bioinformatics-related classes from scratch"
- "Write methods targeting biological sequence or genomic data"
- "Write custom `__init__` methods for initialization and validation in bioinformatics contexts"
keypoints:
- "Classes in Python are blocks started with the `class` keyword"
- "Method definitions look like functions, but must take a `self` argument"
- "The `__init__` method is called when instances are constructed"
- "Bioinformatics classes often include validation and domain-specific logic in `__init__`"
---

In the previous section, we've seen how objects can have different behavior, provided by methods, which in turn are provided by the class of an object.

But what if we want to make our own classes and objects for representing biological data?

If we wanted to create a hierarchy of biological sequences, with consistent methods for summary and validation, we could define a class that does this:

~~~
class BiologicalSequence:
    def __init__(self, sequence):
        self.sequence = sequence.upper()

    def get_sequence(self):
        return self.sequence

    def summary(self):
        raise NotImplementedError("Subclass should implement summary")
~~~
{: .language-python}

We can then create specialized subclasses for DNA and protein sequences:

~~~
class DNASequence(BiologicalSequence):
    def summary(self):
        length = len(self.sequence)
        gc_content = sum(base in "GC" for base in self.sequence) / length
        print(f"DNA sequence length: {length}")
        print(f"GC content: {gc_content:.2%}")

    def validate(self):
        if not all(base in "ACGT" for base in self.sequence):
            raise ValueError("Invalid character found in DNA sequence")


class ProteinSequence(BiologicalSequence):
    def summary(self):
        length = len(self.sequence)
        unique_aas = set(self.sequence)
        print(f"Protein sequence length: {length}")
        print(f"Unique amino acids: {', '.join(unique_aas)}")

    def validate(self):
        allowed_aas = "ACDEFGHIKLMNPQRSTVWY"
        if not all(aa in allowed_aas for aa in self.sequence):
            raise ValueError("Invalid amino acid found in protein sequence")
~~~
{: .language-python}

Similarly to how `def` is used to define a function, the `class` keyword is used to define a new class. Both functions and variables can be created inside the class block, and these will be accessible on any objects (also known as instances) of the class that are created.

When functions are defined within a class, they become *methods* of the class. Methods are functions that operate on the object itself. In order for the method to access and modify the object's data, it needs a way to refer to the specific instance it's working with. This is done through the `self` parameter.

By convention, the first argument of a method is always named `self`. Python automatically passes the instance of the class (the object) as the first argument when you call a method on that object. You don't need to pass it explicitly.

To access variables (attributes) attached to the object, you prefix their names with `self.`. For example, `self.sequence` refers to the `sequence` attribute of the current object.

In the `BiologicalSequence` class:

*   **`__init__(self, sequence)`**: This is the constructor method. It's automatically called when you create a new instance of the class. It initializes the object's attributes, such as `self.sequence`, with the provided sequence. The double underscores indicate it is a special method (often called a "dunder" method).

*   **`get_sequence(self)`**: This method returns the sequence currently stored within the object.

*   **`summary(self)`**: This method is designed to provide a summary of the sequence's contents. In the base class, it raises a `NotImplementedError`, indicating that subclasses must override this method to provide a meaningful summary specific to their sequence type.

> ## Other names than `self`
>
> While it is possible to use any variable name for the first argument of a method, and Python will not complain, other programmers will. Since one aim when programming is to be as clear as possible to others who may read the program later, we strongly recommend following the convention of calling the first argument to methods `self`.
{: .callout}

> ## Naming classes
>
> Another convention in Python is that class names start with a capital letter, and instead of underscores, initial letters of subsequent words are also capitalized. This makes it easier to distinguish classes from objects and other variables at a glance.
{: .callout}

So far this code hasn't visibly done anything; while we have defined a class, we have yet to use it. Let's do that now.

~~~
dna_seq = DNASequence("ATGCGTAC")
prot_seq = ProteinSequence("MKTLLL")

dna_seq.summary()
prot_seq.summary()
~~~
{: .language-python}

## Customizing Object Initialization

The class keyword is used to define a new class, and both functions and variables can be created inside the class block. These will be accessible on any objects of the class that are created. When functions are defined within a class, they become methods of instances of the class. In order for the function to be aware of the object they need to refer to, methods are always given the instance as their first argument. By convention, this argument is called self. To access variables attached to the object, their names must be prefixed by self.

For example, a `Gene` class could represent genomic loci with customizable initialization and validation:

~~~
class Gene:
    def __init__(self, name, sequence):
        self.name = name
        self.sequence = sequence.upper()
        self.validate()

    def validate(self):
        if not self.sequence:
            raise ValueError("Gene sequence cannot be empty.")
        allowed_bases = "ACGT"
        if not all(base in allowed_bases for base in self.sequence):
            raise ValueError("Invalid base in gene sequence.")
~~~
{: .language-python}

This class validates gene data right on object creation, preventing incorrect objects from existing.

If we wanted to create a specialized `ProteinCodingGene` that inherits from `Gene` and adds specific annotations or methods, we would override `__init__` and call `super().__init__`:

~~~
class ProteinCodingGene(Gene):
    def __init__(self, name, sequence, protein_id):
        super().__init__(name, sequence)
        self.protein_id = protein_id

    def translate(self):
        # Example: simplistic translation stub
        codon_map = {'ATG': 'M', 'TGG': 'W'}  # etc.
        protein = ""
        for i in range(0, len(self.sequence) - 2, 3):
            codon = self.sequence[i:i+3]
            protein += codon_map.get(codon, '?')
        return protein
~~~
{: .language-python}

Using classes on the other hand gives a neat way of achieving controlled initialization of bioinformatics objects, encapsulating domain logic, and preparing for extensibility.

This structure sets the stage for more complex bioinformatics pipelines and data models, allowing clean, modular, and maintainable codebases.

{% include links.md %}

[1](https://academic.oup.com/bioinformatics/article/28/22/2996/239703)
[2](https://biopython.org/wiki/The_Biopython_Structural_Bioinformatics_FAQ)
[3](https://open.oregonstate.education/computationalbiology/chapter/objects-and-classes/)
[4](https://omicstutorials.com/python-via-bioinformatics-examples-2/)
[5](http://hplgit.github.io/bioinf-py/doc/pub/html/index.html)
[6](https://microbenotes.com/python-bioinformatics-tools-applications/)
[7](https://www.kaggle.com/code/shtrausslearning/biopython-bioinformatics-basics)
[8](https://www.bioinformaticscrashcourse.com/7_DataAnalysisWithPython.html)
[9](https://www.youtube.com/watch?v=uPHeqVb4Mo0)
