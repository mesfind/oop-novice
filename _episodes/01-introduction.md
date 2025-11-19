---
title: "Objects in Python for Bioinformatics"
teaching: 20
exercises: 15
questions:
- "What are objects in Python?"
- "What is a class or type?"
- "Can objects belong to more than one class?"
- "How can objects be created from a class?"
objectives:
- "Be able to distinguish between class and object"
- "Be able to construct objects via a class's constructor"
- "Be able to distinguish between equality and identity of objects"
keypoints:
- "Anything that we can store in a variable in Python is an object"
- "Every object in Python has a class (or type)"
- "`list`, `numpy.ndarray`, and bioinformatics classes like `Seq` are commonly-used classes; objects are instances of these classes"
- "Calling the class as a function constructs new objects of that class"
- "Classes can inherit from other classes; objects of the subclass are automatically also of the parent class"
---

You may recall that in bioinformatics we often use the `Seq` object from the Biopython library to represent sequences. For example, to create a DNA sequence object:

~~~
from Bio.Seq import Seq
dna_seq = Seq("ATGCGA")
print(dna_seq)
~~~
{: .language-python}

~~~
ATGCGA
~~~
{: .output}

This `dna_seq` is an object of the class `Seq`, which provides convenient sequence methods.

Let's compare this with standard Python types. For example, a DNA sequence stored as a string:

~~~
dna_string = "ATGCGA"
print(dna_string)
~~~
{: .language-python}

~~~
ATGCGA
~~~
{: .output}

While similar, `dna_string` is a plain string and does not have bioinformatics-specific methods.

Let's see if we can call a sequence method on these:

~~~
print(dna_seq.reverse_complement())
print(dna_string.reverse_complement())  # This will error
~~~
{: .language-python}

Python will signal an error for the plain string, but not for `Seq` because it knows how to do this method.

## What type is it?

Let's investigate what types these are:

~~~
type(dna_seq)
type(dna_string)
~~~
{: .language-python}

~~~
lass 'Bio.Seq.Seqeq'>
lass 'strtr'>
~~~
{: .output}

So, `dna_seq` is an object of the type `Bio.Seq.Seq`, which is a class designed for biological sequences. The standard string is of type `str`.

> ## Class or Type?
>
> In Python, _class_ and _type_ can be used interchangeably. Classes provide the blueprint for their objects.
{: .callout}

> ## Let's find some types
>
> Can you find the type of other bioinformatics objects you use, like a `SeqRecord` or a `MultipleSeqAlignment`?
>
>> ## Solution
>>
>> ~~~
>> from Bio.SeqRecord import SeqRecord
>> from Bio.Seq import Seq
>>
>> record = SeqRecord(Seq("ATGC"))
>> print(type(record))
>> ~~~
>>
>> ~~~
>> lass 'Bioio.SeqRecord.SeqRecord'>
>> ~~~
> {: .solution}
{: .challenge}

## Changing things

In Python, objects can be _immutable_ or _mutable_. 

Sequence objects from Biopython like `Seq` are immutable — you cannot change them after creation, which is important for consistency.

Conversely, a list of sequences (e.g. a list of `Seq` objects) is mutable — you can add or modify individual sequences.

Try this example:

~~~
sequences = [Seq("ATGC"), Seq("CGTA")]
sequences = Seq("TTTT")
print(sequences)
~~~
{: .language-python}

## Instances and Methods

An object of a particular class is called an _instance_. For example, `dna_seq` is an instance of class `Seq`. We can check this using `isinstance`:

~~~
from Bio.Seq import Seq
dna_seq = Seq("AGCT")
print(isinstance(dna_seq, Seq))
~~~
{: .language-python}

~~~
True
~~~
{: .output}

Objects have methods which provide functionality. For example, the `Seq` class provides the `complement()` method:

~~~
print(dna_seq.complement())
~~~
{: .language-python}

> ## Finding out what things are
>
> Use `type()` and `isinstance()` to check the types of your biological data objects.
{: .challenge}

## Making objects

Objects are made by calling their class name like a function. For example:

~~~
from Bio.Seq import Seq
new_seq = Seq("GATTACA")
print(type(new_seq))
~~~
{: .language-python}

~~~
lass 'Bio.Seq.Seqeq'>
~~~
{: .output}

> ## Make a list of Seq objects
>
> Given sequences `["ATG", "CCC", "TTA"]`, create a list of `Seq` objects.
>
>> ## Solution
>>
>> ~~~
>> from Bio.Seq import Seq
>> seqs = [Seq(s) for s in ["ATG", "CCC", "TTA"]]
>> print(seqs)
>> ~~~
> {: .solution}
{: .challenge}

## Equality and identity

In bioinformatics, it's important to distinguish when two sequences are the same sequence (equality) versus the same object in memory (identity). 

Example:

~~~
seq1 = Seq("ATG")
seq2 = Seq("ATG")
seq3 = seq1

print(seq1 == seq2)  # True, sequences have equal content
print(seq1 is seq2)  # False, different objects
print(seq1 is seq3)  # True, same object
~~~
{: .language-python}

## Inheritance

Bioinformatics libraries often use inheritance to extend functionality. For example, the `MutableSeq` class inherits from `Seq` and allows mutation:

~~~
from Bio.Seq import MutableSeq
mutable_seq = MutableSeq("ATGC")
print(mutable_seq)
mutable_seq = "G"
print(mutable_seq)
~~~
{: .language-python}

~~~
ATGC
GTGC
~~~
{: .output}

Here `MutableSeq` is a subclass of `Seq`. Objects of `MutableSeq` have all methods of `Seq` plus mutation capabilities.

Inheritance is also used to build exception hierarchies for bioinformatics errors, e.g., a custom `SequenceError` inheriting from `ValueError`.

Understanding classes and objects, their identity and equality, and the use of inheritance is essential for managing biological data programmatically.

{% include links.md %}
