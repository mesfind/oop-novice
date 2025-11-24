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
- "In bioinformatics, duck typing means that if an object provides all the expected sequence or processing methods, it is accepted regardless of its actual class."
- "Duck typing enables writing flexible and generic analysis tools that operate on any sequence-like object implementing required methods, without enforcing inheritance."
- "Inheritance should be used only when subclasses truly represent a specialized version of the parent bioinformatics concept."
- "Composition—combining classes by including others as components—is often a better alternative for complex bioinformatics software integration."
- "Protocols specify expected method sets for objects, facilitating interoperability without rigid class hierarchies."
- "Iterators are common protocols in bioinformatics for efficient streaming of large genome or sequence data."
- "Understanding duck typing and composition helps design modular, extensible, and maintainable bioinformatics code."
- "Enable interoperability between different libraries and data sources"
- "Promote modular design that's easier to test and maintain"
- "Allow flexible workflows that can adapt to new tools and methods"
- "Support efficient processing of large datasets through protocols like iterators"

---

## Introduction to Duck Typing in Bioinformatics

The phrase "duck typing" comes from the saying: "If it looks like a duck, swims like a duck, and quacks like a duck, then it probably is a duck." In Python, this means that objects are considered compatible based on their behavior (methods and attributes) rather than their explicit type or inheritance hierarchy.

{% include image.html url="../assets/img/duck.jpg"
alt="Photograph of a duck and three ducklings on a body of
water" caption="These ducks are swimming and look like ducks, although
the quacking can't be guaranteed from this image."%}

In bioinformatics, duck typing is particularly powerful because it allows us to write generic functions that work with any object that provides the expected methods, regardless of whether they come from Biopython, our custom classes, or other libraries.

## How Duck Typing Works in Practice

Let's look at a simple example with biological sequences:

```python
# Custom sequence class
class SimpleDNASequence:
    def __init__(self, sequence):
        self.sequence = sequence.upper()
    
    def transcribe(self):
        """Transcribe DNA to RNA"""
        return self.sequence.replace('T', 'U')
    
    def complement(self):
        """Return complement sequence"""
        comp_map = str.maketrans('ACGT', 'TGCA')
        return self.sequence.translate(comp_map)
    
    def reverse_complement(self):
        """Return reverse complement"""
        return self.complement()[::-1]

# Function that works with any sequence-like object
def analyze_sequence(seq_obj):
    """Analyze any object that has sequence methods"""
    print(f"Original: {seq_obj.sequence}")
    print(f"Transcribed: {seq_obj.transcribe()}")
    print(f"Reverse complement: {seq_obj.reverse_complement()}")

# This works with our custom class
my_seq = SimpleDNASequence("ATGCGT")
analyze_sequence(my_seq)

# This would also work with Biopython Seq objects
# from Bio.Seq import Seq
# bio_seq = Seq("ATGCGT")
# analyze_sequence(bio_seq)  # Would work if Seq has these methods
```

The key insight is that `analyze_sequence()` doesn't care about the specific class of `seq_obj` - it only cares that the object has the methods it needs to call.

## Protocols and Iterators in Bioinformatics

Protocols are informal interfaces in Python - they define what methods an object should have to work with certain Python features. One common protocol is the **iterator protocol**, which is essential for processing large biological datasets efficiently.

```python
class FastaRecordIterator:
    """Iterator that reads FASTA records one at a time from a file"""
    
    def __init__(self, fasta_path):
        self.file = open(fasta_path)
        self.current_header = None
    
    def __iter__(self):
        """Return the iterator object itself"""
        return self
    
    def __next__(self):
        """Return the next FASTA record"""
        header = None
        sequence_lines = []
        
        # Read until we find a header or end of file
        while True:
            line = self.file.readline()
            if not line:  # End of file
                if header is None:
                    self.file.close()
                    raise StopIteration
                else:
                    break
            
            line = line.strip()
            if line.startswith('>'):
                if header is not None:
                    # We found the next header, so return current record
                    self.current_header = line[1:]
                    break
                else:
                    # First header
                    header = line[1:]
            else:
                sequence_lines.append(line)
        
        if header is None:
            self.file.close()
            raise StopIteration
            
        return {'header': header, 'sequence': ''.join(sequence_lines)}

# Usage
fasta_iterator = FastaRecordIterator("sequences.fasta")
for record in fasta_iterator:
    print(f"Processing {record['header']}, length: {len(record['sequence'])}")
    # Process one record at a time, saving memory
```

> ## Challenge 1: Iterator for high-quality RNA-seq reads
>
> Implement an iterator class that yields only reads with average Phred quality score ≥ 30 from a collection of RNA-seq reads. Assume each read has `.sequence` and `.quality` attributes, where `.quality` is a list of Phred scores.
>
>> ## Solution
>>
>> ```python
>> class HighQualityReadIterator:
>>     def __init__(self, reads, min_avg_quality=30):
>>         self.reads = reads
>>         self.index = 0
>>         self.min_avg_quality = min_avg_quality
>>
>>     def __iter__(self):
>>         """Return the iterator object itself"""
>>         self.index = 0
>>         return self
>>
>>     def __next__(self):
>>         """Return the next high-quality read"""
>>         while self.index < len(self.reads):
>>             read = self.reads[self.index]
>>             self.index += 1
>>             
>>             # Calculate average quality
>>             if read.quality:  # Ensure quality scores exist
>>                 avg_quality = sum(read.quality) / len(read.quality)
>>                 if avg_quality >= self.min_avg_quality:
>>                     return read
>>         
>>         raise StopIteration
>>
>> # Example usage with mock data
>> class Read:
>>     def __init__(self, sequence, quality):
>>         self.sequence = sequence
>>         self.quality = quality
>>
>> # Create test data
>> test_reads = [
>>     Read("ATCGATCG", [20, 25, 30, 35, 40, 35, 30, 25]),  # Avg: 30.0
>>     Read("GGGCCC", [10, 15, 20, 15, 10, 15]),            # Avg: 14.2
>>     Read("TTTTAAAA", [40, 45, 40, 45, 40, 45, 40, 45]),  # Avg: 42.5
>> ]
>>
>> print("High-quality reads (avg Phred ≥ 30):")
>> for read in HighQualityReadIterator(test_reads):
>>     avg_qual = sum(read.quality) / len(read.quality)
>>     print(f"  {read.sequence} (avg quality: {avg_qual:.1f})")
>> ```
>> {: .language-python}
>>
>> ```
>> High-quality reads (avg Phred ≥ 30):
>>   ATCGATCG (avg quality: 30.0)
>>   TTTTAAAA (avg quality: 42.5)
>> ```
>> {: .output}
> {: .solution}
{: .challenge}

## Composition Over Inheritance in Bioinformatics

While inheritance is useful for "is-a" relationships, **composition** (building objects from other objects) is often better for complex bioinformatics workflows. Composition promotes modularity and makes code easier to test and maintain.

```python
class SequenceFetcher:
    def fetch_sequences(self, query):
        """Fetch sequences from a database (simplified)"""
        # In real implementation, this might query GenBank, UniProt, etc.
        mock_sequences = {
            "BRCA1": "ATGCGTTAGCTAGCTAGCT",
            "TP53": "ATGCCGATCGATCGATCG", 
            "EGFR": "ATGGGCTAGCTAGCTAGC"
        }
        return mock_sequences.get(query, "")
    
    def fetch_multiple(self, queries):
        return {query: self.fetch_sequences(query) for query in queries}


class SequenceAligner:
    def align(self, sequences):
        """Perform multiple sequence alignment (simplified)"""
        # In real implementation, this might call MAFFT, ClustalOmega, etc.
        print(f"Aligning {len(sequences)} sequences...")
        return f"Alignment of {', '.join(sequences.keys())}"


class PhylogeneticTreeBuilder:
    def build_tree(self, alignment):
        """Build phylogenetic tree from alignment (simplified)"""
        print(f"Building tree from alignment...")
        return "Phylogenetic tree object"


class PhylogeneticPipeline:
    def __init__(self, fetcher, aligner, tree_builder):
        # Composition: we use these objects rather than inheriting from them
        self.fetcher = fetcher
        self.aligner = aligner
        self.tree_builder = tree_builder
    
    def run_analysis(self, gene_queries):
        """Run complete phylogenetic analysis"""
        print("=== Phylogenetic Analysis Pipeline ===")
        
        # 1. Fetch sequences
        sequences = self.fetcher.fetch_multiple(gene_queries)
        print(f"Fetched sequences for: {list(sequences.keys())}")
        
        # 2. Align sequences
        alignment = self.aligner.align(sequences)
        print(f"Alignment: {alignment}")
        
        # 3. Build tree
        tree = self.tree_builder.build_tree(alignment)
        print(f"Result: {tree}")
        
        return tree

# Create component objects
fetcher = SequenceFetcher()
aligner = SequenceAligner()
tree_builder = PhylogeneticTreeBuilder()

# Compose them into a pipeline
pipeline = PhylogeneticPipeline(fetcher, aligner, tree_builder)

# Run analysis
pipeline.run_analysis(["BRCA1", "TP53", "EGFR"])
```

> ## Challenge 2: Duck-typed biosequence interface
>
> Implement a class for biological sequences that supports `length()`, `complement()`, and `transcribe()` methods. Then write a generic function that can process any object with these methods using duck typing.
>
>> ## Solution
>>
>> ```python
>> class DNASequence:
>>     def __init__(self, sequence, name=""):
>>         self.sequence = sequence.upper()
>>         self.name = name
>>     
>>     def length(self):
>>         return len(self.sequence)
>>     
>>     def complement(self):
>>         """Return DNA complement"""
>>         comp_map = str.maketrans('ACGT', 'TGCA')
>>         return self.sequence.translate(comp_map)
>>     
>>     def transcribe(self):
>>         """Transcribe DNA to RNA"""
>>         return self.sequence.replace('T', 'U')
>>     
>>     def __repr__(self):
>>         return f"DNASequence('{self.sequence}', '{self.name}')"
>>
>>
>> class RNASequence:
>>     def __init__(self, sequence, name=""):
>>         self.sequence = sequence.upper()
>>         self.name = name
>>     
>>     def length(self):
>>         return len(self.sequence)
>>     
>>     def complement(self):
>>         """Return RNA complement"""
>>         comp_map = str.maketrans('ACGU', 'UGCA')
>>         return self.sequence.translate(comp_map)
>>     
>>     def transcribe(self):
>>         """RNA doesn't transcribe, return self"""
>>         return self.sequence
>>     
>>     def __repr__(self):
>>         return f"RNASequence('{self.sequence}', '{self.name}')"
>>
>>
>> def process_biological_sequence(seq_obj):
>>     """Generic function that works with any object having the right methods"""
>>     print(f"Processing: {seq_obj}")
>>     print(f"  Length: {seq_obj.length()} bases")
>>     print(f"  Complement: {seq_obj.complement()}")
>>     print(f"  Transcription: {seq_obj.transcribe()}")
>>     print()
>>
>> # Test with different sequence types
>> dna = DNASequence("ATGCGT", "TestDNA")
>> rna = RNASequence("AUGCGU", "TestRNA")
>>
>> process_biological_sequence(dna)
>> process_biological_sequence(rna)
>> ```
>> {: .language-python}
>>
>> ```
>> Processing: DNASequence('ATGCGT', 'TestDNA')
>>   Length: 6 bases
>>   Complement: TACGCA
>>   Transcription: AUGCGU
>>
>> Processing: RNASequence('AUGCGU', 'TestRNA')
>>   Length: 6 bases
>>   Complement: UACGCA
>>   Transcription: AUGCGU
>> ```
>> {: .output}
> {: .solution}
{: .challenge}

> ## Challenge 3: Composition-based phylogenetic pipeline
>
> Implement a `PhyloPipeline` using composition of `SequenceFetcher`, `Aligner`, and `TreeBuilder` components. Make the pipeline flexible enough to work with different implementations of each component.
>
>> ## Solution
>>
>> ```python
>> class SequenceFetcher:
>>     def fetch_sequences(self, queries):
>>         """Fetch sequences - base implementation"""
>>         return {query: f"SEQUENCE_FOR_{query}" for query in queries}
>>
>>
>> class DatabaseSequenceFetcher(SequenceFetcher):
>>     def fetch_sequences(self, queries):
>>         """Simulate fetching from a biological database"""
>>         mock_database = {
>>             "geneA": "ATGCGTTAGCTAGCT",
>>             "geneB": "ATGCCGATCGATCG", 
>>             "geneC": "ATGGGCTAGCTAGC",
>>             "geneD": "ATGTTTAGCTAGCT"
>>         }
>>         return {query: mock_database.get(query, "UNKNOWN") for query in queries}
>>
>>
>> class Aligner:
>>     def align(self, sequences):
>>         """Base aligner interface"""
>>         return f"Alignment of {len(sequences)} sequences"
>>
>>
>> class SimpleAligner(Aligner):
>>     def align(self, sequences):
>>         """Simple alignment implementation"""
>>         seq_names = list(sequences.keys())
>>         return f"Simple alignment: {' vs '.join(seq_names)}"
>>
>>
>> class TreeBuilder:
>>     def build_tree(self, alignment):
>>         """Base tree builder interface"""
>>         return "Phylogenetic tree"
>>
>>
>> class NeighborJoiningTreeBuilder(TreeBuilder):
>>     def build_tree(self, alignment):
>>         """Neighbor-joining tree building"""
>>         return "Neighbor-joining phylogenetic tree"
>>
>>
>> class PhyloPipeline:
>>     def __init__(self, fetcher, aligner, tree_builder):
>>         # Composition: we use objects rather than inheritance
>>         self.fetcher = fetcher
>>         self.aligner = aligner
>>         self.tree_builder = tree_builder
>>
>>     def run(self, gene_queries):
>>         """Run the complete phylogenetic pipeline"""
>>         print("=== Running Phylogenetic Pipeline ===")
>>         
>>         # Step 1: Fetch sequences
>>         sequences = self.fetcher.fetch_sequences(gene_queries)
>>         print(f"1. Fetched {len(sequences)} sequences")
>>         for gene, seq in sequences.items():
>>             print(f"   - {gene}: {seq[:10]}...")
>>         
>>         # Step 2: Align sequences
>>         alignment = self.aligner.align(sequences)
>>         print(f"2. {alignment}")
>>         
>>         # Step 3: Build tree
>>         tree = self.tree_builder.build_tree(alignment)
>>         print(f"3. Built: {tree}")
>>         
>>         return tree
>>
>> # Create different component implementations
>> db_fetcher = DatabaseSequenceFetcher()
>> simple_aligner = SimpleAligner()
>> nj_builder = NeighborJoiningTreeBuilder()
>>
>> # Compose the pipeline
>> pipeline = PhyloPipeline(db_fetcher, simple_aligner, nj_builder)
>>
>> # Run analysis
>> result = pipeline.run(["geneA", "geneB", "geneC"])
>> ```
>> {: .language-python}
>>
>> ```
>> === Running Phylogenetic Pipeline ===
>> 1. Fetched 3 sequences
>>    - geneA: ATGCGTTAGC...
>>    - geneB: ATGCCGATCG...
>>    - geneC: ATGGGCTAGC...
>> 2. Simple alignment: geneA vs geneB vs geneC
>> 3. Built: Neighbor-joining phylogenetic tree
>> ```
>> {: .output}
> {: .solution}
{: .challenge}

## Duck Typing with Variant Data

Duck typing is particularly useful when working with genomic variants from different sources (VCF files, databases, custom formats).

```python
class BasicVariant:
    def __init__(self, chrom, pos, ref, alt):
        self.chrom = chrom
        self.pos = pos
        self.ref = ref
        self.alt = alt
    
    def get_position(self):
        return self.pos
    
    def get_ref(self):
        return self.ref
    
    def get_alt(self):
        return self.alt
    
    def is_snp(self):
        return len(self.ref) == 1 and len(self.alt) == 1


class AnnotatedVariant(BasicVariant):
    def __init__(self, chrom, pos, ref, alt, annotations=None):
        super().__init__(chrom, pos, ref, alt)
        self.annotations = annotations or {}
    
    def get_annotation(self, key):
        return self.annotations.get(key)
    
    def is_pathogenic(self):
        return self.annotations.get('clinical_significance') == 'Pathogenic'


def summarize_variant(variant_obj):
    """Generic function that works with any variant-like object"""
    print(f"Variant at {variant_obj.chrom}:{variant_obj.get_position()}")
    print(f"  Change: {variant_obj.get_ref()} -> {variant_obj.get_alt()}")
    print(f"  Type: {'SNP' if variant_obj.is_snp() else 'INDEL/Other'}")
    
    # Duck typing: if the object has additional methods, use them
    if hasattr(variant_obj, 'is_pathogenic'):
        pathogenic = variant_obj.is_pathogenic()
        print(f"  Pathogenic: {pathogenic if pathogenic is not None else 'Unknown'}")
    
    if hasattr(variant_obj, 'get_annotation'):
        for key in ['gene', 'impact']:
            value = variant_obj.get_annotation(key)
            if value:
                print(f"  {key}: {value}")

# Test with different variant types
basic_snp = BasicVariant("chr1", 12345, "A", "G")
annotated_snp = AnnotatedVariant(
    "chr1", 12345, "A", "G", 
    {'gene': 'BRCA1', 'impact': 'missense', 'clinical_significance': 'Pathogenic'}
)

print("Basic variant:")
summarize_variant(basic_snp)
print("\nAnnotated variant:")
summarize_variant(annotated_snp)
```

> ## Challenge 4: Duck-typed variant object
>
> Implement SNP and Indel classes with `get_position()`, `get_ref()`, and `get_alt()` methods, plus class-specific methods. Write a generic function that can handle both types.
>
>> ## Solution
>>
>> ```python
>> class Variant:
>>     def __init__(self, chrom, pos, ref, alt):
>>         self.chrom = chrom
>>         self.pos = pos
>>         self.ref = ref
>>         self.alt = alt
>>     
>>     def get_position(self):
>>         return self.pos
>>     
>>     def get_ref(self):
>>         return self.ref
>>     
>>     def get_alt(self):
>>         return self.alt
>>     
>>     def __repr__(self):
>>         return f"Variant({self.chrom}:{self.pos} {self.ref}>{self.alt})"
>>
>>
>> class SNP(Variant):
>>     def __init__(self, chrom, pos, ref, alt, rs_id=None):
>>         super().__init__(chrom, pos, ref, alt)
>>         self.rs_id = rs_id
>>         if len(ref) != 1 or len(alt) != 1:
>>             raise ValueError("SNP must have single nucleotide ref and alt")
>>     
>>     def get_rs_id(self):
>>         return self.rs_id
>>     
>>     def is_transition(self):
>>         """Check if SNP is transition (purine<->purine or pyrimidine<->pyrimidine)"""
>>         transitions = [('A', 'G'), ('G', 'A'), ('C', 'T'), ('T', 'C')]
>>         return (self.ref, self.alt) in transitions
>>     
>>     def __repr__(self):
>>         rs_info = f" rs{self.rs_id}" if self.rs_id else ""
>>         return f"SNP({self.chrom}:{self.pos} {self.ref}>{self.alt}{rs_info})"
>>
>>
>> class Indel(Variant):
>>     def __init__(self, chrom, pos, ref, alt, length_change=None):
>>         super().__init__(chrom, pos, ref, alt)
>>         self.length_change = length_change or (len(alt) - len(ref))
>>     
>>     def get_length_change(self):
>>         return self.length_change
>>     
>>     def is_insertion(self):
>>         return self.length_change > 0
>>     
>>     def is_deletion(self):
>>         return self.length_change < 0
>>     
>>     def __repr__(self):
>>         change_type = "INS" if self.is_insertion() else "DEL"
>>         return f"Indel({self.chrom}:{self.pos} {change_type} len_change={self.length_change})"
>>
>>
>> def analyze_variant_collection(variants):
>>     """Generic function that works with any variant-like objects"""
>>     print("=== Variant Analysis ===")
>>     snp_count = 0
>>     indel_count = 0
>>     
>>     for variant in variants:
>>         print(f"\n{variant}")
>>         print(f"  Position: {variant.get_position()}")
>>         print(f"  Change: {variant.get_ref()} -> {variant.get_alt()}")
>>         
>>         # Duck typing: use methods if they exist
>>         if hasattr(variant, 'is_transition'):
>>             snp_count += 1
>>             trans_type = "transition" if variant.is_transition() else "transversion"
>>             print(f"  SNP type: {trans_type}")
>>             if variant.get_rs_id():
>>                 print(f"  RS ID: {variant.get_rs_id()}")
>>         
>>         if hasattr(variant, 'is_insertion'):
>>             indel_count += 1
>>             indel_type = "insertion" if variant.is_insertion() else "deletion"
>>             print(f"  Indel type: {indel_type}")
>>             print(f"  Length change: {variant.get_length_change()}")
>>     
>>     print(f"\nSummary: {snp_count} SNPs, {indel_count} indels")
>>
>> # Test with different variant types
>> variants = [
>>     SNP("chr1", 1000, "A", "G", rs_id="12345"),
>>     SNP("chr2", 2000, "C", "T"),  # No RS ID
>>     Indel("chr3", 3000, "AT", "A"),  # Deletion
>>     Indel("chr4", 4000, "C", "CTG"),  # Insertion
>> ]
>>
>> analyze_variant_collection(variants)
>> ```
>> {: .language-python}
>>
>> ```
>> === Variant Analysis ===
>>
>> SNP(chr1:1000 A>G rs12345)
>>   Position: 1000
>>   Change: A -> G
>>   SNP type: transition
>>   RS ID: rs12345
>>
>> SNP(chr2:2000 C>T)
>>   Position: 2000
>>   Change: C -> T
>>   SNP type: transition
>>
>> Indel(chr3:3000 DEL len_change=-1)
>>   Position: 3000
>>   Change: AT -> A
>>   Indel type: deletion
>>   Length change: -1
>>
>> Indel(chr4:4000 INS len_change=2)
>>   Position: 4000
>>   Change: C -> CTG
>>   Indel type: insertion
>>   Length change: 2
>>
>> Summary: 2 SNPs, 2 indels
>> ```
>> {: .output}
> {: .solution}
{: .challenge}

## Advanced Composition Patterns

For complex bioinformatics workflows, composition allows us to build sophisticated pipelines from simple, testable components.

> ## Challenge 5: Composition for CRISPR gRNA workflow
>
> Compose `Designer`, `OffTargetChecker`, and `EfficiencyPredictor` into a `CRISPRPipeline` that can design and evaluate guide RNAs.
>
>> ## Solution
>>
>> ```python
>> class GuideRNADesigner:
>>     def design_guides(self, target_sequence, num_guides=3):
>>         """Design guide RNAs for a target sequence"""
>>         # Simplified implementation - real one would use specific rules
>>         guides = []
>>         for i in range(num_guides):
>>             start = i * 5
>>             if start + 20 <= len(target_sequence):
>>                 guide_seq = target_sequence[start:start+20]
>>                 guides.append(f"guide_{i+1}_{guide_seq}")
>>         return guides
>>
>>
>> class OffTargetChecker:
>>     def check_off_targets(self, guide_rna, genome_sequence, max_mismatches=3):
>>         """Check for off-target binding sites"""
>>         # Simplified implementation
>>         import random
>>         # Simulate finding some off-target sites
>>         num_off_targets = random.randint(0, 2)
>>         return [f"off_target_{i+1}" for i in range(num_off_targets)]
>>
>>
>> class EfficiencyPredictor:
>>     def predict_efficiency(self, guide_rna):
>>         """Predict editing efficiency of guide RNA"""
>>         # Simplified implementation - real one might use machine learning
>>         import random
>>         return round(random.uniform(0.3, 0.9), 2)
>>
>>
>> class CRISPRPipeline:
>>     def __init__(self, designer, off_target_checker, efficiency_predictor):
>>         self.designer = designer
>>         self.off_target_checker = off_target_checker
>>         self.efficiency_predictor = efficiency_predictor
>>
>>     def run_design(self, target_sequence, genome_sequence):
>>         """Run complete gRNA design and evaluation pipeline"""
>>         print("=== CRISPR Guide RNA Design Pipeline ===")
>>         print(f"Target: {target_sequence}")
>>         print()
>>         
>>         # Step 1: Design guide RNAs
>>         guides = self.designer.design_guides(target_sequence)
>>         print(f"1. Designed {len(guides)} guide RNAs:")
>>         for guide in guides:
>>             print(f"   - {guide}")
>>         print()
>>         
>>         results = []
>>         
>>         # Step 2: Evaluate each guide
>>         for i, guide in enumerate(guides, 1):
>>             print(f"2.{i} Evaluating {guide}:")
>>             
>>             # Check off-targets
>>             off_targets = self.off_target_checker.check_off_targets(
>>                 guide, genome_sequence
>>             )
>>             print(f"   - Off-targets: {len(off_targets)}")
>>             for ot in off_targets:
>>                 print(f"     * {ot}")
>>             
>>             # Predict efficiency
>>             efficiency = self.efficiency_predictor.predict_efficiency(guide)
>>             print(f"   - Predicted efficiency: {efficiency}")
>>             
>>             # Store results
>>             results.append({
>>                 'guide': guide,
>>                 'off_targets': off_targets,
>>                 'efficiency': efficiency
>>             })
>>             print()
>>         
>>         # Step 3: Rank guides
>>         print("3. Ranking guides by efficiency:")
>>         ranked_results = sorted(results, key=lambda x: x['efficiency'], reverse=True)
>>         for i, result in enumerate(ranked_results, 1):
>>             print(f"   {i}. {result['guide']} (eff: {result['efficiency']}, "
>>                   f"off-targets: {len(result['off_targets'])})")
>>         
>>         return ranked_results
>>
>> # Create component objects
>> designer = GuideRNADesigner()
>> off_target_checker = OffTargetChecker()
>> efficiency_predictor = EfficiencyPredictor()
>>
>> # Compose the pipeline
>> crispr_pipeline = CRISPRPipeline(designer, off_target_checker, efficiency_predictor)
>>
>> # Run the pipeline
>> target_seq = "ATCGATCGATCGATCGATCGATCGATCGATCGATCGATCG"
>> genome_seq = "ATCGATCGATCGATCGATCGATCGATCGATCGATCGATCG" * 100  # Mock genome
>>
>> results = crispr_pipeline.run_design(target_seq, genome_seq)
>> ```
>> {: .language-python}
> {: .solution}
{: .challenge}

## When to Use Duck Typing vs. Inheritance

**Use inheritance when:**
- You have a clear "is-a" relationship (e.g., `ProteinCodingGene` is a `Gene`)
- You want to share implementation code between closely related classes
- You're extending functionality in a hierarchical manner

**Use duck typing/composition when:**
- Objects share behavior but not necessarily identity
- You want flexibility to swap implementations
- You're building complex systems from simpler components
- You're integrating with external libraries or tools

{% include links.md %}
