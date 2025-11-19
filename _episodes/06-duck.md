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
keypoints:
- "In bioinformatics, duck typing means that if an object provides all the expected sequence or processing methods, it is accepted regardless of its actual class."
- "Duck typing enables writing flexible and generic analysis tools that operate on any sequence-like object implementing required methods, without enforcing inheritance."
- "Inheritance should be used only when subclasses truly represent a specialized version of the parent bioinformatics concept."
- "Composition—combining classes by including others as components—is often a better alternative for complex bioinformatics software integration."
- "Protocols specify expected method sets for objects, facilitating interoperability without rigid class hierarchies."
- "Iterators are common protocols in bioinformatics for efficient streaming of large genome or sequence data."
- "Understanding duck typing and composition helps design modular, extensible, and maintainable bioinformatics code."
---

There is a principle that if something "looks like a duck, and swims like a duck, and quacks like a duck, then it is probably a duck".

{% include image.html url="../assets/img/duck.jpg"
alt="Photograph of a duck and three ducklings on a body of
water" caption="These ducks are swimming and look like ducks, although
the quacking can't be guaranteed from this image."%}

Python's type system adopts a similar philosophy—if an object behaves like a biological sequence with the methods you expect (e.g., `transcribe()`, `complement()`, `reverse_complement()`), then Python treats it as a sequence, regardless of its concrete class.

For example, an assembly algorithm might accept any object that has a `get_sequence()` method returning nucleotide data, without requiring a strict class inheritance.

Here is a duck-typed Newton-Raphson solver that works on any numeric type, including bioinformatics-related numerical objects if they support required operations:

~~~
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
{: .language-python}

This works with real numbers, complex numbers, or customized numerical objects representing biological quantities.

Bioinformatics-specific duck typing allows writing generic code operating on varied sequence-like objects or analysis results without enforcing strict inheritance.


## Protocols in Bioinformatics

Protocols specify required methods for an object to interact with Python features. For example, an iterator protocol for iterating over sequence records:

~~~
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
{: .language-python}

Using it with a `for` loop processes large datasets lazily.


## Composition vs. Inheritance in Bioinformatics

Composition assembles complex bioinformatics functionality by including specialized classes as attributes rather than deep inheritance hierarchies.

Example:

~~~
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
{: .language-python}

This design improves modularity and flexibility over large inheritance trees common in bioinformatics libraries.



*Duck typing* in Python means that if an object looks and behaves appropriately (e.g., has methods like `transcribe()`, `complement()`, `reverse_complement()` for sequences), Python will allow the operations without explicitly checking the object's inheritance.

In bioinformatics, this means you can write generalized algorithms that operate on any sequence-like object or numerical object supporting the needed operations, without enforcing strong inheritance relations.

## Protocols and Iterators in Bioinformatics

A bioinformatics iterator protocol example is a `FastaRecordIterator` that reads sequences one by one from a large FASTA file:

~~~
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
{: .language-python}

Iterators let you process large biological datasets efficiently by reading data lazily.


## Composition Over Inheritance in Bioinformatics

When handling complex bioinformatics workflows, composition can be better than inheritance to add functionality.

For example, a `SequenceAnalyzer` class can be composed with a `SequenceFetcher` and a `SequenceAligner`, coordinating functionality across these components without requiring deep class hierarchies.

This promotes modularity and easier maintenance especially when integrating multiple bioinformatics tools.


> ## Challenge 1: Iterator for high-quality RNA-seq reads
>
> Implement an iterator class that yields reads with average Phred score ≥ 30 from a FASTQ dataset.
>
> Assume each read object has `.sequence` and `.quality`.
> {: .challenge}

> > ## Solution
> >
> > ```python
> > class HighQualityReadIterator:
> >     def __init__(self, reads, min_qual=30):
> >         self.reads = reads
> >         self.index = 0
> >         self.min_qual = min_qual
> >
> >     def __iter__(self):
> >         return self
> >
> >     def __next__(self):
> >         while self.index < len(self.reads):
> >             read = self.reads[self.index]
> >             self.index += 1
> >             avg_q = sum(read.quality)/len(read.quality)
> >             if avg_q >= self.min_qual:
> >                 return read
> >         raise StopIteration
> > ```
> >
> > {: .language-python}
> > {: .solution}

---

> ## Challenge 2: Duck-typed biosequence interface
>
> Implement a class for sequences supporting `length()`, `complement()`, and `transcribe()` methods. Use duck typing to allow generic processing functions.
> {: .challenge}

> > ## Solution
> >
> > ```python
> > class DNASequence:
> >     def __init__(self, seq):
> >         self.seq = seq.upper()
> >     def length(self):
> >         return len(self.seq)
> >     def complement(self):
> >         return self.seq.translate(str.maketrans('ACGT','TGCA'))
> >     def transcribe(self):
> >         return self.seq.replace('T','U')
> >
> > def process_sequence(obj):
> >     print(f"Length: {obj.length()}")
> >     print(f"Complement: {obj.complement()}")
> >     print(f"RNA Transcription: {obj.transcribe()}")
> >
> > dna = DNASequence("ATGC")
> > process_sequence(dna)
> > ```
> >
> > {: .language-python}
> > {: .solution}

---

> ## Challenge 3: Composition-based phylogenetic pipeline
>
> Implement a `PhyloPipeline` using composition of `SequenceFetcher`, `Aligner`, and `TreeBuilder`.
> {: .challenge}

> > ## Solution
> >
> > ```python
> > class PhyloPipeline:
> >     def __init__(self, fetcher, aligner, tree_builder):
> >         self.fetcher = fetcher
> >         self.aligner = aligner
> >         self.tree_builder = tree_builder
> >
> >     def run(self):
> >         seqs = self.fetcher.fetch()
> >         aligned = self.aligner.align(seqs)
> >         return self.tree_builder.build_tree(aligned)
> > ```
> >
> > {: .language-python}
> > {: .solution}

---

> ## Challenge 4: Duck-typed variant object
>
> Implement SNP and indel objects with `get_position()`, `get_ref()`, and `get_alt()`.
> {: .challenge}

> > ## Solution
> >
> > ```python
> > class Variant:
> >     def __init__(self, chrom, pos, ref, alt):
> >         self.chrom = chrom
> >         self.pos = pos
> >         self.ref = ref
> >         self.alt = alt
> >     def get_position(self):
> >         return self.pos
> >     def get_ref(self):
> >         return self.ref
> >     def get_alt(self):
> >         return self.alt
> >
> > def summarize_variant(var):
> >     print(f"Position: {var.get_position()}, {var.get_ref()}->{var.get_alt()}")
> >
> > snp = Variant("chr1",12345,"A","G")
> > summarize_variant(snp)
> > ```
> >
> > {: .language-python}
> > {: .solution}

---

> ## Challenge 5: Single-cell iterator
>
> Implement an iterator that yields only cells passing a mitochondrial fraction threshold (e.g., <10%).
> {: .challenge}

> > ## Solution
> >
> > ```python
> > class HighQualityCellIterator:
> >     def __init__(self, cells, mito_thresh=0.1):
> >         self.cells = cells
> >         self.index = 0
> >         self.mito_thresh = mito_thresh
> >
> >     def __iter__(self):
> >         return self
> >
> >     def __next__(self):
> >         while self.index < len(self.cells):
> >             cell = self.cells[self.index]
> >             self.index += 1
> >             if cell.mito_fraction < self.mito_thresh:
> >                 return cell
> >         raise StopIteration
> > ```
> >
> > {: .language-python}
> > {: .solution}

---

> ## Challenge 6: CRISPR gRNA workflow
>
> Compose `Designer`, `OffTargetChecker`, and `EfficiencyPredictor` into a `CRISPRPipeline`.
> {: .challenge}

> > ## Solution
> >
> > ```python
> > class CRISPRPipeline:
> >     def __init__(self, designer, offchecker, predictor):
> >         self.designer = designer
> >         self.offchecker = offchecker
> >         self.predictor = predictor
> >
> >     def run(self, target):
> >         grnas = self.designer.design(target)
> >         off_targets = [self.offchecker.check(g) for g in grnas]
> >         scores = [self.predictor.score(g) for g in grnas]
> >         return list(zip(grnas, off_targets, scores))
> > ```
> >
> > {: .language-python}
> > {: .solution}

---

> ## Challenge 7: Iterator for GC-rich genes
>
> Yield genes with GC content > 50% from a genome dataset.
> {: .challenge}

> > ## Solution
> >
> > ```python
> > class GCRichGeneIterator:
> >     def __init__(self, genes):
> >         self.genes = genes
> >         self.index = 0
> >
> >     def __iter__(self):
> >         return self
> >
> >     def __next__(self):
> >         while self.index < len(self.genes):
> >             gene = self.genes[self.index]
> >             self.index += 1
> >             gc = (gene.sequence.count('G') + gene.sequence.count('C')) / len(gene.sequence)
> >             if gc > 0.5:
> >                 return gene
> >         raise StopIteration
> > ```
> >
> > {: .language-python}
> > {: .solution}

---

> ## Challenge 8: Protein sequence duck typing
>
> Implement `length()`, `translate()`, and `is_hydrophobic()` methods for protein sequences to allow generic functions.
> {: .challenge}

> > ## Solution
> >
> > ```python
> > class ProteinSequence:
> >     def __init__(self, seq):
> >         self.seq = seq
> >     def length(self):
> >         return len(self.seq)
> >     def translate(self):
> >         return self.seq  # already amino acid
> >     def is_hydrophobic(self, aa):
> >         return aa in "AILMFWV"
> > ```
> >
> > {: .language-python}
> > {: .solution}

---

> ## Challenge 9: Metagenomic abundance iterator
>
> Iterate over taxa and yield only those with abundance above a threshold.
> {: .challenge}

> > ## Solution
> >
> > ```python
> > class AbundantTaxonIterator:
> >     def __init__(self, taxa, min_abundance):
> >         self.taxa = taxa
> >         self.index = 0
> >         self.min_abundance = min_abundance
> >
> >     def __iter__(self):
> >         return self
> >
> >     def __next__(self):
> >         while self.index < len(self.taxa):
> >             taxon = self.taxa[self.index]
> >             self.index += 1
> >             if taxon.get_abundance() > self.min_abundance:
> >                 return taxon
> >         raise StopIteration
> > ```
> >
> > {: .language-python}
> > {: .solution}

---

> ## Challenge 10: Composition-based RNA-seq workflow
>
> Combine `ReadFetcher`, `QCFilter`, and `CountMatrixBuilder` into a modular pipeline.
> {: .challenge}

> > ## Solution
> >
> > ```python
> > class RNASeqPipeline:
> >     def __init__(self, fetcher, qcfilter, count_builder):
> >         self.fetcher = fetcher
> >         self.qcfilter = qcfilter
> >         self.count_builder = count_builder
> >
> >     def run(self):
> >         reads = self.fetcher.fetch()
> >         filtered_reads = self.qcfilter.filter(reads)
> >         return self.count_builder.build(filtered_reads)
> > ```
> >
> > {: .language-python}
> > {: .solution}

---

> ## Challenge 11: Iterator for low-expression genes
>
> Yield genes whose expression is below a threshold in single-cell or bulk RNA-seq.
> {: .challenge}

> > ## Solution
> >
> > ```python
> > class LowExpressionGeneIterator:
> >     def __init__(self, genes, threshold):
> >         self.genes = genes
> >         self.index = 0
> >         self.threshold = threshold
> >
> >     def __iter__(self):
> >         return self
> >
> >     def __next__(self):
> >         while self.index < len(self.genes):
> >             gene = self.genes[self.index]
> >             self.index += 1
> >             if gene.expression < self.threshold:
> >                 return gene
> >         raise StopIteration
> > ```
> >
> > {: .language-python}
> > {: .solution}

---

> ## Challenge 12: Metabolomics duck-typed feature
>
> Implement metabolite objects with `get_mass()`, `get_intensity()`, and `get_rt()` for generic processing.
> {: .challenge}

> > ## Solution
> >
> > ```python
> > class Metabolite:
> >     def __init__(self, mass, intensity, rt):
> >         self.mass = mass
> >         self.intensity = intensity
> >         self.rt = rt
> >
> >     def get_mass(self):
> >         return self.mass
> >
> >     def get_intensity(self):
> >         return self.intensity
> >
> >     def get_rt(self):
> >         return self.rt
> > ```
> >
> > {: .language-python}
> > {: .solution}

---

> ## Challenge 13: Phylogenetic bootstrap composition
>
> Combine `TreeBuilder` and `Bootstrapper` into a pipeline class that returns bootstrap-supported trees.
> {: .challenge}

> > ## Solution
> >
> > ```python
> > class BootstrapPipeline:
> >     def __init__(self, tree_builder, bootstrapper):
> >         self.tree_builder = tree_builder
> >         self.bootstrapper = bootstrapper
> >
> >     def run(self, sequences):
> >         tree = self.tree_builder.build_tree(sequences)
> >         return self.bootstrapper.bootstrap(tree)
> > ```
> >
> > {: .language-python}
> > {: .solution}

---

> ## Challenge 14: Iterator for highly variable SNPs
>
> Yield variants with minor allele frequency above a threshold.
> {: .challenge}

> > ## Solution
> >
> > ```python
> > class HighMAFVariantIterator:
> >     def __init__(self, variants, maf_threshold):
> >         self.variants = variants
> >         self.index = 0
> >         self.maf_threshold = maf_threshold
> >
> >     def __iter__(self):
> >         return self
> >
> >     def __next__(self):
> >         while self.index < len(self.variants):
> >             var = self.variants[self.index]
> >             self.index += 1
> >             if var.maf > self.maf_threshold:
> >                 return var
> >         raise StopIteration
> > ```
> >
> > {: .language-python}
> > {: .solution}

---

> ## Challenge 15: Composition for proteomics analysis
>
> Compose `PeptideExtractor`, `Quantifier`, and `Normalizer` into a modular workflow class.
> {: .challenge}

> > ## Solution
> >
> > ```python
> > class ProteomicsPipeline:
> >     def __init__(self, extractor, quantifier, normalizer):
> >         self.extractor = extractor
> >         self.quantifier = quantifier
> >         self.normalizer = normalizer
> >
> >     def run(self, raw_data):
> >         peptides = self.extractor.extract(raw_data)
> >         quantified = self.quantifier.quantify(peptides)
> >         normalized = self.normalizer.normalize(quantified)
> >         return normalized
> > ```
> >
> > {: .language-python}
> > {: .solution}



