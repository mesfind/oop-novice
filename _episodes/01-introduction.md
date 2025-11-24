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

## What are Objects in Python?

In Python, **everything is an object**. This means that any piece of data you work with - whether it's a number, string, list, or a complex biological sequence - is an **object** that belongs to a specific **class** (also called a **type**).

Think of a class as a blueprint and objects as the actual buildings constructed from that blueprint. The blueprint defines what the building can do, and each building is a specific instance.

## Objects in Bioinformatics: Seq vs String

In bioinformatics, we often use specialized objects like the `Seq` object from Biopython instead of plain Python strings. Let's see why:

```python
from Bio.Seq import Seq

# Create a DNA sequence as a Seq object
dna_seq = Seq("ATGCGA")
print(f"Seq object: {dna_seq}")

# Create the same sequence as a plain string
dna_string = "ATGCGA"
print(f"String: {dna_string}")
```

Both print the same sequence, but they have different capabilities:

```python
# Seq objects have bioinformatics-specific methods
print(f"Reverse complement: {dna_seq.reverse_complement()}")

# This will cause an error - strings don't have bioinformatics methods
# print(dna_string.reverse_complement())  # Uncomment to see the error
```

The `Seq` object understands biological operations like reverse complement, while a plain string does not.

## Understanding Types and Classes

Every object in Python has a type (class) that defines what it can do. Let's examine the types of our sequences:

```python
print(f"Type of dna_seq: {type(dna_seq)}")
print(f"Type of dna_string: {type(dna_string)}")
```

```
<class 'Bio.Seq.Seq'>
<class 'str'>
```

This shows that `dna_seq` is an instance of the `Bio.Seq.Seq` class (designed for biological sequences), while `dna_string` is an instance of the built-in `str` class.

> ## Class or Type?
>
> In Python, the terms **class** and **type** are used interchangeably. Both refer to the blueprint that defines what an object can do and what data it can store.
{: .callout}

> ## Let's find some types
>
> Can you find the type of other bioinformatics objects you use, like a `SeqRecord` or a `MultipleSeqAlignment`?
>
>> ## Solution
>>
>> ```python
>> from Bio.SeqRecord import SeqRecord
>> from Bio.Seq import Seq
>> from Bio.Align import MultipleSeqAlignment
>>
>> # Create a SeqRecord and check its type
>> record = SeqRecord(Seq("ATGC"))
>> print(f"SeqRecord type: {type(record)}")
>>
>> # Create an alignment and check its type  
>> alignment = MultipleSeqAlignment([record])
>> print(f"Alignment type: {type(alignment)}")
>> ```
>> {: .language-python}
>>
>> ```
>> SeqRecord type: <class 'Bio.SeqRecord.SeqRecord'>
>> Alignment type: <class 'Bio.Align.MultipleSeqAlignment'>
>> ```
>> {: .output}
> {: .solution}
{: .challenge}

## Mutable vs Immutable Objects

In bioinformatics, it's important to understand whether objects can be changed after creation:

- **Immutable objects** cannot be modified after creation
- **Mutable objects** can be modified after creation

```python
# Seq objects are immutable - they cannot be changed
from Bio.Seq import Seq
dna_seq = Seq("ATGC")
print(f"Original: {dna_seq}")

# This would cause an error because Seq is immutable:
# dna_seq[0] = "G"  # Uncomment to see the error

# Lists are mutable - they can be changed
sequences = [Seq("ATGC"), Seq("CGTA")]
print(f"Original list: {sequences}")

# We can modify the list
sequences.append(Seq("TTTT"))
print(f"Modified list: {sequences}")
```

Immutable objects like `Seq` are safer for bioinformatics because they prevent accidental changes to important biological data.

## Instances and Methods

When we create an object from a class, we say we create an **instance** of that class. Each instance has access to **methods** (functions) defined by its class.

```python
from Bio.Seq import Seq

# Create an instance of the Seq class
dna_seq = Seq("AGCT")
print(f"Is dna_seq a Seq instance? {isinstance(dna_seq, Seq)}")

# Use methods provided by the Seq class
print(f"Sequence: {dna_seq}")
print(f"Complement: {dna_seq.complement()}")
print(f"Transcribe to RNA: {dna_seq.transcribe()}")
```

> ## Finding out what things are
>
> Practice using `type()` and `isinstance()` to check the types of your biological data objects.
>
>> ## Solution
>>
>> ```python
>> from Bio.Seq import Seq
>> from Bio.SeqRecord import SeqRecord
>>
>> # Create some bioinformatics objects
>> my_seq = Seq("ATGCCC")
>> my_record = SeqRecord(my_seq)
>> my_list = [my_seq, my_record]
>>
>> # Check their types
>> print(f"my_seq is type: {type(my_seq)}")
>> print(f"my_record is type: {type(my_record)}")
>> print(f"my_list is type: {type(my_list)}")
>>
>> # Check if they are instances of specific classes
>> print(f"my_seq is a Seq: {isinstance(my_seq, Seq)}")
>> print(f"my_record is a SeqRecord: {isinstance(my_record, SeqRecord)}")
>> print(f"my_list is a list: {isinstance(my_list, list)}")
>> ```
>> {: .language-python}
> {: .solution}
{: .challenge}

## Creating Objects (Instantiation)

Objects are created by **calling the class name as a function**. This process is called **instantiation**.

```python
from Bio.Seq import Seq

# Create a new Seq object by calling the Seq class
new_seq = Seq("GATTACA")
print(f"Sequence: {new_seq}")
print(f"Type: {type(new_seq)}")

# The parentheses after Seq contain arguments for the constructor
# Different classes need different arguments
```

> ## Make a list of Seq objects
>
> Given sequences `["ATG", "CCC", "TTA"]`, create a list of `Seq` objects.
>
>> ## Solution
>>
>> ```python
>> from Bio.Seq import Seq
>> 
>> # Create a list of Seq objects using a list comprehension
>> seqs = [Seq(s) for s in ["ATG", "CCC", "TTA"]]
>> print(seqs)
>> 
>> # Check that each item is indeed a Seq object
>> for i, seq_obj in enumerate(seqs):
>>     print(f"Item {i}: {seq_obj} (type: {type(seq_obj)})")
>> ```
>> {: .language-python}
>> 
>> ```
>> [Seq('ATG'), Seq('CCC'), Seq('TTA')]
>> Item 0: ATG (type: <class 'Bio.Seq.Seq'>)
>> Item 1: CCC (type: <class 'Bio.Seq.Seq'>)
>> Item 2: TTA (type: <class 'Bio.Seq.Seq'>)
>> ```
>> {: .output}
> {: .solution}
{: .challenge}

## Object Identity vs Equality

In bioinformatics, it's crucial to understand the difference between:
- **Equality** (`==`): Do two objects contain the same data?
- **Identity** (`is`): Are two objects the same object in memory?

```python
from Bio.Seq import Seq

# Create three sequence objects
seq1 = Seq("ATG")
seq2 = Seq("ATG")  # Same content, different object
seq3 = seq1        # Same object

print("=== Equality Check (==) ===")
print(f"seq1 == seq2: {seq1 == seq2}")  # True - same sequence content
print(f"seq1 == seq3: {seq1 == seq3}")  # True - same sequence content

print("\n=== Identity Check (is) ===")
print(f"seq1 is seq2: {seq1 is seq2}")  # False - different objects
print(f"seq1 is seq3: {seq1 is seq3}")  # True - same object

print("\n=== Memory Addresses ===")
print(f"seq1 id: {id(seq1)}")
print(f"seq2 id: {id(seq2)}")
print(f"seq3 id: {id(seq3)}")
```

This distinction is important when working with large biological datasets to avoid unnecessary memory usage.

## Inheritance in Bioinformatics

**Inheritance** allows new classes to be based on existing classes, inheriting their properties and methods while adding new functionality.

```python
from Bio.Seq import Seq, MutableSeq

# MutableSeq inherits from Seq but allows modification
mutable_seq = MutableSeq("ATGC")
print(f"Original: {mutable_seq}")

# MutableSeq can be changed (unlike Seq)
mutable_seq[0] = "G"
print(f"Modified: {mutable_seq}")

# Check inheritance relationship
print(f"MutableSeq is a subclass of Seq: {issubclass(MutableSeq, Seq)}")
print(f"mutable_seq is a Seq: {isinstance(mutable_seq, Seq)}")
print(f"mutable_seq is a MutableSeq: {isinstance(mutable_seq, MutableSeq)}")
```

Inheritance is widely used in bioinformatics libraries:
- Custom exceptions inherit from built-in exceptions
- Specialized sequence types inherit from basic sequence classes
- Database record classes inherit from base record classes

## Practical Example: Custom Sequence Class

Let's create a simple custom sequence class to understand these concepts better:

```python
class DNASequence:
    """A simple DNA sequence class"""
    
    def __init__(self, sequence):
        # Constructor - called when creating new instances
        self.sequence = sequence.upper()
        self._validate()
    
    def _validate(self):
        """Validate that sequence contains only DNA bases"""
        valid_bases = set('ACGT')
        if not all(base in valid_bases for base in self.sequence):
            raise ValueError("Sequence contains invalid DNA characters")
    
    def gc_content(self):
        """Calculate GC content"""
        gc_count = self.sequence.count('G') + self.sequence.count('C')
        return gc_count / len(self.sequence)
    
    def __str__(self):
        """String representation"""
        return self.sequence
    
    def __len__(self):
        """Length of the sequence"""
        return len(self.sequence)

# Create instances of our custom class
dna1 = DNASequence("ATGCGTA")
dna2 = DNASequence("atgcgta")  # Will be converted to uppercase

print(f"DNA 1: {dna1}")
print(f"DNA 2: {dna2}")
print(f"GC content: {dna1.gc_content():.2f}")
print(f"Length: {len(dna1)}")
print(f"Type: {type(dna1)}")

# This will raise an error due to invalid characters:
# bad_dna = DNASequence("ATGCX")
```

> ## Create a Protein Sequence Class
>
> Create a simple `ProteinSequence` class that:
> - Validates the sequence contains only valid amino acid codes (ACDEFGHIKLMNPQRSTVWY*)
> - Has a method to calculate molecular weight (you can use a simple approximation)
> - Has a string representation method
>
>> ## Solution
>>
>> ```python
>> class ProteinSequence:
>>     """A simple protein sequence class"""
>>     
>>     def __init__(self, sequence):
>>         self.sequence = sequence.upper()
>>         self._validate()
>>     
>>     def _validate(self):
>>         """Validate that sequence contains only valid amino acids"""
>>         valid_aa = set('ACDEFGHIKLMNPQRSTVWY*')
>>         if not all(aa in valid_aa for aa in self.sequence):
>>             raise ValueError("Sequence contains invalid amino acid codes")
>>     
>>     def approximate_weight(self):
>>         """Approximate molecular weight in Daltons"""
>>         # Simple approximation: average amino acid weight ~110 Da
>>         return len(self.sequence) * 110
>>     
>>     def __str__(self):
>>         return self.sequence
>>     
>>     def __len__(self):
>>         return len(self.sequence)
>>
>> # Test the class
>> protein = ProteinSequence("ACDEFGHIK")
>> print(f"Protein: {protein}")
>> print(f"Length: {len(protein)}")
>> print(f"Approx weight: {protein.approximate_weight()} Da")
>> ```
>> {: .language-python}
> {: .solution}
{: .challenge}


{% include links.md %}
