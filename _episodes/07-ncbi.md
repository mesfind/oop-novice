---
title: "Accessing NCBI's Entrez with Biopython"
teaching: 30
exercises: 25
questions:
- "How can I programmatically access biological data from NCBI databases?"
- "What are the main Entrez utilities available through Biopython?"
- "How do I search, fetch, and parse data from GenBank, PubMed, and other NCBI resources?"
- "What are the best practices for responsible data access from NCBI?"
objectives:
- "Use Biopython's Entrez module to search NCBI databases"
- "Fetch records from GenBank, PubMed, and other databases"
- "Parse and extract information from XML and other NCBI data formats"
- "Understand and comply with NCBI's data access policies"
keypoints:
- "Biopython's `Entrez` module provides a Python interface to NCBI's Entrez system"
- "Always set your email address when using Entrez to comply with NCBI's policies"
- "Use `esearch` to find records, `efetch` to retrieve them, and `esummary` to get summaries"
- "Parse complex data using Biopython's parsers like `Medline` and `SeqIO`"
- "Be respectful of NCBI's servers by implementing delays between requests"
- "`esearch` find records in any NCBI database"
- "`efetch` retrieve complete records"
- "`esummary`"" get summary information efficiently"
- "Search History work with large result sets"
- "Multiple Databases Access Gene, Protein, Taxonomy, and more"

---

## Introduction to NCBI Entrez

The National Center for Biotechnology Information (NCBI) provides a vast collection of biological databases including GenBank, PubMed, Protein, and many others. Biopython's `Entrez` module gives you programmatic access to these resources through NCBI's Entrez Programming Utilities (E-utilities).

## Setting Up Entrez Access

Before making any requests, you must configure your email address. This is required by NCBI and helps them contact you if there are problems with your requests.

```python
from Bio import Entrez

# Always set your email address
Entrez.email = "your.email@example.com"  # Use your real email

# Optional: Set your API key for higher rate limits
# Entrez.api_key = "your_api_key_here"  # Get from NCBI
```

> ## NCBI Data Access Policies
>
> NCBI provides free access to their databases, but they have usage guidelines:
> - **Always provide your email address**
> - **Make no more than 3 requests per second**
> - **Use the API key for higher rate limits (10 requests/second)**
> - **Be respectful of their servers during peak hours**
{: .callout}

## Searching NCBI Databases

The `esearch` function allows you to search any NCBI database. Let's start with searching PubMed for articles.

```python
def search_pubMed(query, max_results=10):
    """Search PubMed for articles matching a query"""
    handle = Entrez.esearch(db="pubmed", term=query, retmax=max_results)
    record = Entrez.read(handle)
    handle.close()
    return record

# Search for COVID-19 vaccine articles
covid_results = search_pubMed("COVID-19 vaccine", max_results=5)
print(f"Found {covid_results['Count']} articles")
print(f"First 5 IDs: {covid_results['IdList']}")
```

> ## Challenge 1: Search GenBank for a Gene
>
> Write a function to search GenBank for sequences related to the "insulin" gene in humans. Return the count and IDs of matching sequences.
>
>> ## Solution
>>
>> ```python
>> def search_genbank(gene_name, organism="human", max_results=10):
>>     """Search GenBank for a gene in a specific organism"""
>>     query = f"{gene_name}[Gene] AND {organism}[Organism]"
>>     handle = Entrez.esearch(db="nucleotide", term=query, retmax=max_results)
>>     record = Entrez.read(handle)
>>     handle.close()
>>     return record
>>
>> # Search for insulin gene in humans
>> insulin_results = search_genbank("insulin", "human", 5)
>> print(f"Found {insulin_results['Count']} insulin sequences")
>> print(f"IDs: {insulin_results['IdList']}")
>> ```
>> {: .language-python}
> {: .solution}
{: .challenge}

## Fetching Records

Once you have IDs from a search, you can fetch the actual records using `efetch`. The format of the returned data depends on the database and requested return type.

### Fetching PubMed Records

```python
def fetch_pubmed_details(pubmed_ids):
    """Fetch detailed information for PubMed IDs"""
    if not pubmed_ids:
        return []
    
    # Fetch in Medline format
    handle = Entrez.efetch(db="pubmed", id=pubmed_ids, rettype="medline", retmode="text")
    records = Entrez.read(handle, format="medline")
    handle.close()
    return records

# Fetch details for COVID articles
covid_ids = covid_results['IdList']
if covid_ids:
    articles = fetch_pubmed_details(covid_ids)
    for article in articles:
        print(f"Title: {article.get('TI', 'No title')}")
        print(f"Authors: {', '.join(article.get('AU', []))}")
        print(f"Journal: {article.get('SO', 'No journal info')}")
        print("-" * 50)
```

### Fetching GenBank Sequences

```python
from Bio import SeqIO
import io

def fetch_genbank_sequences(genbank_ids):
    """Fetch GenBank sequences as SeqRecord objects"""
    if not genbank_ids:
        return []
    
    # Fetch in GenBank format
    handle = Entrez.efetch(db="nucleotide", id=genbank_ids, rettype="gb", retmode="text")
    records = list(SeqIO.parse(handle, "gb"))
    handle.close()
    return records

# Fetch insulin sequences
insulin_ids = insulin_results['IdList'][:3]  # First 3 sequences
if insulin_ids:
    sequences = fetch_genbank_sequences(insulin_ids)
    for seq in sequences:
        print(f"Accession: {seq.id}")
        print(f"Description: {seq.description}")
        print(f"Length: {len(seq)} bp")
        print(f"Features: {len(seq.features)}")
        print("-" * 50)
```

> ## Challenge 2: Fetch Protein Sequences
>
> Write a function to search and fetch protein sequences from the Protein database for a specific gene. Extract the sequence and some annotations.
>
>> ## Solution
>>
>> ```python
>> def search_and_fetch_proteins(gene_name, organism="human", max_results=5):
>>     """Search and fetch protein sequences for a gene"""
>>     # Search for proteins
>>     query = f"{gene_name}[Gene] AND {organism}[Organism]"
>>     handle = Entrez.esearch(db="protein", term=query, retmax=max_results)
>>     search_results = Entrez.read(handle)
>>     handle.close()
>>     
>>     if not search_results['IdList']:
>>         print("No proteins found")
>>         return []
>>     
>>     # Fetch protein sequences
>>     handle = Entrez.efetch(db="protein", id=search_results['IdList'], 
>>                           rettype="fasta", retmode="text")
>>     fasta_data = handle.read()
>>     handle.close()
>>     
>>     # Parse FASTA sequences
>>     from io import StringIO
>>     records = list(SeqIO.parse(StringIO(fasta_data), "fasta"))
>>     
>>     print(f"Found {len(records)} protein sequences:")
>>     for record in records:
>>         print(f"  {record.id}: {record.description}")
>>         print(f"  Length: {len(record.seq)} amino acids")
>>         print(f"  First 30 aa: {str(record.seq[:30])}")
>>         print()
>>     
>>     return records
>>
>> # Fetch insulin protein sequences
>> insulin_proteins = search_and_fetch_proteins("insulin", "human", 3)
>> ```
>> {: .language-python}
> {: .solution}
{: .challenge}

## Using ESummary for Database Summaries

The `esummary` utility provides summary information without fetching complete records, which is faster for getting overview information.

```python
def get_pubmed_summaries(pubmed_ids):
    """Get summaries for PubMed articles"""
    if not pubmed_ids:
        return []
    
    handle = Entrez.esummary(db="pubmed", id=",".join(pubmed_ids))
    records = Entrez.read(handle)
    handle.close()
    return records

# Get summaries for COVID articles
if covid_ids:
    summaries = get_pubmed_summaries(covid_ids)
    for summary in summaries:
        print(f"Title: {summary.get('Title', 'No title')}")
        print(f"Authors: {summary.get('AuthorList', ['Unknown'])[0]} et al.")
        print(f"Publication Date: {summary.get('PubDate', 'Unknown')}")
        print(f"DOI: {summary.get('DOI', 'No DOI')}")
        print("-" * 50)
```

## Advanced Search Techniques

### Building Complex Queries

You can build sophisticated queries using NCBI's search field tags and Boolean operators.

```python
def advanced_genome_search(gene_name, organism, molecule_type="genomic", complete_only=True):
    """Perform an advanced search with multiple criteria"""
    query_parts = [
        f"{gene_name}[Gene]",
        f"{organism}[Organism]",
        f"{molecule_type}[Molecule Type]"
    ]
    
    if complete_only:
        query_parts.append("complete[Title]")
    
    query = " AND ".join(query_parts)
    
    print(f"Search query: {query}")
    
    handle = Entrez.esearch(db="nucleotide", term=query, retmax=10)
    record = Entrez.read(handle)
    handle.close()
    
    return record

# Search for complete genomic sequences of BRCA1 in humans
brca1_results = advanced_genome_search("BRCA1", "human", "genomic", True)
print(f"Found {brca1_results['Count']} complete BRCA1 genomic sequences")
```

### Using Search History

For large searches, you can use NCBI's search history feature to work with result sets.

```python
def search_with_history(query, db="pubmed", max_results=100):
    """Perform a search and store results in NCBI's history"""
    handle = Entrez.esearch(db=db, term=query, retmax=max_results, usehistory="y")
    record = Entrez.read(handle)
    handle.close()
    
    # These are needed for subsequent fetches
    webenv = record["WebEnv"]
    query_key = record["QueryKey"]
    
    return {
        'count': record['Count'],
        'webenv': webenv,
        'query_key': query_key,
        'ids': record['IdList']
    }

# Example: Search and store in history
cancer_search = search_with_history("cancer immunotherapy", "pubmed", 50)
print(f"Stored {cancer_search['count']} results in history")
print(f"WebEnv: {cancer_search['webenv'][:50]}...")
```

> ## Challenge 3: Fetch Large Result Sets
>
> Use search history to fetch large sets of records in batches to be respectful of NCBI's servers.
>
>> ## Solution
>>
>> ```python
>> def fetch_large_dataset(webenv, query_key, db="pubmed", batch_size=100):
>>     """Fetch large datasets in batches using search history"""
>>     all_records = []
>>     total_count = int(webenv.split('|')[-1])  # Extract count from webenv
>>     
>>     for start in range(0, total_count, batch_size):
>>         print(f"Fetching records {start+1} to {min(start+batch_size, total_count)}...")
>>         
>>         handle = Entrez.efetch(
>>             db=db,
>>             rettype="medline",
>>             retmode="text",
>>             retstart=start,
>>             retmax=batch_size,
>>             webenv=webenv,
>>             query_key=query_key
>>         )
>>         
>>         batch_records = list(Entrez.read(handle, format="medline"))
>>         all_records.extend(batch_records)
>>         handle.close()
>>         
>>         # Be nice to NCBI servers
>>         import time
>>         time.sleep(1)  # 1 second delay between requests
>>     
>>     return all_records
>>
>> # Usage example (commented out to avoid actual API calls)
>> # large_results = fetch_large_dataset(cancer_search['webenv'], cancer_search['query_key'])
>> # print(f"Fetched {len(large_results)} total records")
>> ```
>> {: .language-python}
> {: .solution}
{: .challenge}

## Working with Different Databases

NCBI provides many specialized databases. Here are examples of working with some commonly used ones:

### Gene Database

```python
def search_genes(gene_name, organism="human"):
    """Search the Gene database for gene information"""
    query = f"{gene_name}[Gene Name] AND {organism}[Organism]"
    handle = Entrez.esearch(db="gene", term=query, retmax=5)
    record = Entrez.read(handle)
    handle.close()
    
    if record['IdList']:
        # Fetch gene details
        handle = Entrez.efetch(db="gene", id=record['IdList'][0], retmode="xml")
        gene_record = Entrez.read(handle)
        handle.close()
        return gene_record
    return None

# Search for TP53 gene
tp53_info = search_genes("TP53", "human")
if tp53_info:
    print("TP53 Gene Information:")
    gene_data = tp53_info[0]['Entrezgene']
    print(f"Official Symbol: {gene_data['Gene-ref']['Gene-ref_locus']}")
    print(f"Official Name: {gene_data['Gene-ref']['Gene-ref_desc']}")
    if 'Gene-ref_syn' in gene_data['Gene-ref']:
        synonyms = gene_data['Gene-ref']['Gene-ref_syn']
        print(f"Synonyms: {', '.join(synonyms)}")
```

### Taxonomy Database

```python
def get_taxonomy_info(taxon_name):
    """Get taxonomy information for an organism"""
    handle = Entrez.esearch(db="taxonomy", term=taxon_name)
    record = Entrez.read(handle)
    handle.close()
    
    if record['IdList']:
        handle = Entrez.efetch(db="taxonomy", id=record['IdList'][0], retmode="xml")
        taxon_record = Entrez.read(handle)
        handle.close()
        return taxon_record
    return None

# Get taxonomy for Escherichia coli
e_coli_tax = get_taxonomy_info("Escherichia coli")
if e_coli_tax:
    taxon = e_coli_tax[0]
    print(f"Scientific Name: {taxon['ScientificName']}")
    print(f"Taxonomy ID: {taxon['TaxId']}")
    print(f"Lineage: {taxon['Lineage']}")
    print(f"Rank: {taxon['Rank']}")
```

> ## Challenge 4: Multi-Database Search
>
> Create a function that searches multiple NCBI databases simultaneously for comprehensive information about a gene.
>
>> ## Solution
>>
>> ```python
>> def comprehensive_gene_search(gene_name, organism="human"):
>>     """Search multiple databases for comprehensive gene information"""
>>     results = {}
>>     
>>     # Search nucleotide database
>>     handle = Entrez.esearch(db="nucleotide", 
>>                           term=f"{gene_name}[Gene] AND {organism}[Organism]", 
>>                           retmax=3)
>>     results['nucleotide'] = Entrez.read(handle)
>>     handle.close()
>>     
>>     # Search protein database
>>     handle = Entrez.esearch(db="protein", 
>>                           term=f"{gene_name}[Gene] AND {organism}[Organism]", 
>>                           retmax=3)
>>     results['protein'] = Entrez.read(handle)
>>     handle.close()
>>     
>>     # Search gene database
>>     handle = Entrez.esearch(db="gene", 
>>                           term=f"{gene_name}[Gene Name] AND {organism}[Organism]", 
>>                           retmax=3)
>>     results['gene'] = Entrez.read(handle)
>>     handle.close()
>>     
>>     # Search PubMed
>>     handle = Entrez.esearch(db="pubmed", 
>>                           term=f"{gene_name}[Title/Abstract] AND {organism}[Organism]", 
>>                           retmax=5)
>>     results['pubmed'] = Entrez.read(handle)
>>     handle.close()
>>     
>>     # Print summary
>>     print(f"Comprehensive search results for {gene_name} in {organism}:")
>>     for db, result in results.items():
>>         print(f"  {db.capitalize()}: {result['Count']} records found")
>>         if result['IdList']:
>>             print(f"    First ID: {result['IdList'][0]}")
>>     
>>     return results
>>
>> # Example usage
>> cfhr_search = comprehensive_gene_search("CFTR", "human")
>> ```
>> {: .language-python}
> {: .solution}
{: .challenge}

## Error Handling and Best Practices

Always implement proper error handling when working with web services.

```python
import time
from urllib.error import HTTPError

def safe_entrez_request(func, *args, max_retries=3, **kwargs):
    """Make Entrez requests with error handling and retries"""
    for attempt in range(max_retries):
        try:
            return func(*args, **kwargs)
        except HTTPError as e:
            if e.code == 429:  # Too Many Requests
                wait_time = 2 ** attempt  # Exponential backoff
                print(f"Rate limited. Waiting {wait_time} seconds...")
                time.sleep(wait_time)
            else:
                raise e
        except Exception as e:
            print(f"Attempt {attempt + 1} failed: {e}")
            if attempt == max_retries - 1:
                raise e
            time.sleep(1)
    
    return None

# Example usage with error handling
try:
    results = safe_entrez_request(
        Entrez.esearch, 
        db="pubmed", 
        term="cancer", 
        retmax=10
    )
    if results:
        record = Entrez.read(results)
        results.close()
        print(f"Found {record['Count']} articles")
except Exception as e:
    print(f"Search failed: {e}")
```

## Practical Example: Building a Literature Review Tool

Let's build a more comprehensive tool that combines multiple Entrez utilities.

```python
class LiteratureReviewer:
    """A tool for conducting systematic literature reviews using NCBI data"""
    
    def __init__(self, email):
        Entrez.email = email
    
    def search_literature(self, topic, years=None, max_results=100):
        """Search PubMed for literature on a specific topic"""
        query = topic
        if years:
            query += f" AND ({years}[PDAT])"
        
        print(f"Searching for: {query}")
        
        handle = Entrez.esearch(db="pubmed", term=query, retmax=max_results, usehistory="y")
        search_results = Entrez.read(handle)
        handle.close()
        
        return search_results
    
    def fetch_article_details(self, search_results, batch_size=50):
        """Fetch detailed information for search results"""
        webenv = search_results["WebEnv"]
        query_key = search_results["QueryKey"]
        total_count = int(search_results["Count"])
        
        all_articles = []
        
        for start in range(0, total_count, batch_size):
            print(f"Fetching articles {start+1} to {min(start+batch_size, total_count)}")
            
            handle = Entrez.efetch(
                db="pubmed",
                rettype="medline",
                retmode="text", 
                retstart=start,
                retmax=batch_size,
                webenv=webenv,
                query_key=query_key
            )
            
            batch_articles = list(Entrez.read(handle, format="medline"))
            all_articles.extend(batch_articles)
            handle.close()
            
            time.sleep(0.5)  # Be respectful
            
        return all_articles
    
    def analyze_results(self, articles):
        """Analyze and summarize literature search results"""
        if not articles:
            print("No articles to analyze")
            return
        
        print(f"\n=== Literature Analysis ===")
        print(f"Total articles: {len(articles)}")
        
        # Count articles by year
        years = {}
        for article in articles:
            year = article.get('DP', '')[:4]  # Extract year from date
            if year.isdigit():
                years[year] = years.get(year, 0) + 1
        
        print("\nArticles by year:")
        for year in sorted(years.keys()):
            print(f"  {year}: {years[year]} articles")
        
        # Most common journals
        journals = {}
        for article in articles:
            journal = article.get('TA', 'Unknown')
            journals[journal] = journals.get(journal, 0) + 1
        
        print("\nTop journals:")
        for journal, count in sorted(journals.items(), key=lambda x: x[1], reverse=True)[:5]:
            print(f"  {journal}: {count} articles")

# Example usage
if __name__ == "__main__":
    reviewer = LiteratureReviewer("your.email@example.com")
    
    # Search for articles on CRISPR gene editing in the last 5 years
    search_results = reviewer.search_literature(
        "CRISPR gene editing", 
        years="2018:2023", 
        max_results=50
    )
    
    articles = reviewer.fetch_article_details(search_results)
    reviewer.analyze_results(articles)
```




{% include links.md %}
